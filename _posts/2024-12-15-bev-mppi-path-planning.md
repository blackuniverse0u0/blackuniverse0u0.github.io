---
layout: post
title: BEV-MPPI for Robot Path Planning - Bridging Perception and Control
date: 2024-12-15 10:00:00
description: How we use Bird's Eye View representation and Model Predictive Path Integral control for real-time collision-free navigation
tags: robotics planning control perception
categories: research
---

## The Challenge of Real-Time Navigation

Autonomous navigation in complex environments like urban sidewalks requires solving two fundamental problems simultaneously:
1. **Perception**: Understanding the 3D world from sensor data
2. **Planning**: Finding collision-free paths that respect robot dynamics

Traditional approaches often treat these as separate modules, leading to suboptimal performance. Our **BEV-MPPI** framework tightly couples semantic perception with sampling-based optimal control.

## Bird's Eye View (BEV) Representation

### Why BEV?

Unlike perspective images where scale varies with distance, BEV provides a **metrically consistent** representation of the environment:
- Uniform spatial resolution across the planning horizon
- Direct correspondence between pixels and real-world coordinates
- Easier integration with cost maps and collision checking

### From RGB-D to BEV

Our pipeline:

1. **Semantic Segmentation**: SegFormer processes RGB images to classify pixels (sidewalk, road, obstacle, pedestrian)
2. **3D Projection**: Depth maps + camera intrinsics → 3D point cloud
3. **BEV Transform**: Project 3D points onto a ground plane grid

```python
def rgb_depth_to_bev(rgb, depth, K, grid_size=128, resolution=0.1):
    """
    Transform RGB-D to BEV semantic grid

    Args:
        rgb: (H, W, 3) RGB image
        depth: (H, W) depth map in meters
        K: (3, 3) camera intrinsic matrix
        grid_size: BEV grid dimension
        resolution: meters per pixel

    Returns:
        bev_grid: (grid_size, grid_size, C) semantic BEV
    """
    # Segment RGB image
    semantic_map = segformer(rgb)  # (H, W, num_classes)

    # Back-project to 3D
    points_3d = backproject(depth, K)

    # Transform to robot frame and project to BEV
    bev_grid = project_to_bev(points_3d, semantic_map,
                               grid_size, resolution)

    return bev_grid
```

## Model Predictive Path Integral (MPPI)

### Why MPPI?

Unlike gradient-based optimizers (e.g., iLQR), MPPI is a **sampling-based** method that:
- Handles non-differentiable cost functions (e.g., collision checks)
- Naturally incorporates uncertainty
- Parallelizes well on GPUs

### The MPPI Algorithm

At each time step:

1. **Sample Trajectories**: Draw K noisy control sequences
   $$
   u_k = \bar{u} + \epsilon_k, \quad \epsilon_k \sim \mathcal{N}(0, \Sigma)
   $$

2. **Rollout Dynamics**: Simulate each trajectory forward
   $$
   x_{t+1} = f(x_t, u_k^t)
   $$

3. **Evaluate Cost**: Compute cost for each trajectory
   $$
   S_k = \sum_{t=0}^{T} c(x_t^k, u_k^t)
   $$

4. **Weighted Average**: Combine trajectories using exponential weights
   $$
   u^* = \sum_{k=1}^{K} w_k u_k, \quad w_k = \frac{\exp(-\lambda S_k)}{\sum_j \exp(-\lambda S_j)}
   $$

### Cost Function Design

Our cost function balances multiple objectives:

```python
def mppi_cost(trajectory, bev_grid):
    """
    Compute trajectory cost from BEV semantic grid

    Cost components:
    - Collision: High cost for obstacle cells
    - Goal: Distance to target
    - Comfort: Penalize high accelerations
    """
    cost = 0.0

    for x, y, theta, v in trajectory:
        # Collision cost from BEV
        if bev_grid[x, y] == OBSTACLE:
            cost += 1000.0

        # Goal reaching
        cost += np.linalg.norm([x - goal_x, y - goal_y])

        # Comfort (smooth motion)
        cost += 0.1 * (v - v_prev)**2

    return cost
```

## Integration: BEV + MPPI

### The Control Loop

```python
class BevMppiPlanner:
    def __init__(self, K=1000, T=20, dt=0.1):
        self.K = K  # Number of samples
        self.T = T  # Planning horizon
        self.dt = dt  # Time step

    def plan(self, rgb, depth, state, goal):
        # 1. Perception: Generate BEV
        bev = rgb_depth_to_bev(rgb, depth, self.K_camera)

        # 2. Planning: MPPI optimization
        controls = self.mppi_step(bev, state, goal)

        return controls[0]  # Return first control

    def mppi_step(self, bev, state, goal):
        # Sample K control sequences
        u_samples = self.sample_controls()

        # Rollout and evaluate
        costs = []
        for u_k in u_samples:
            traj = self.rollout(state, u_k)
            cost = mppi_cost(traj, bev, goal)
            costs.append(cost)

        # Weighted average
        weights = self.compute_weights(costs)
        u_opt = np.average(u_samples, weights=weights, axis=0)

        return u_opt
```

### Real-Time Performance

On NVIDIA Jetson Xavier:
- BEV generation: 15ms (SegFormer + projection)
- MPPI optimization: 18ms (K=1000 samples, T=20 steps)
- **Total**: 33ms → **30 Hz** control loop

## Results

### Simulation (MetaUrban)

- **Success Rate**: 96% over 500 episodes
- **Collision-Free**: 100% (conservative cost tuning)
- **Path Efficiency**: 1.15x optimal path length

### Qualitative Behavior

BEV-MPPI naturally exhibits:
- **Proactive avoidance**: Slows down near crowded areas
- **Smooth trajectories**: Low jerk due to comfort cost
- **Goal-directed**: Balances safety and progress

## Comparison with Baselines

| Method | Success | Comfort | Latency |
|--------|---------|---------|---------|
| DWA (Dynamic Window) | 87% | Medium | 5ms |
| iPlanner (IL) | 92% | High | 12ms |
| **BEV-MPPI (Ours)** | **96%** | **High** | 33ms |

## Lessons Learned

### 1. BEV Resolution Matters
- Too coarse (>20cm): Misses narrow gaps
- Too fine (<5cm): Computational overhead
- Sweet spot: 10cm for urban navigation

### 2. Cost Function Tuning
Collision weight dominates:
```python
# Good balance
w_collision = 1000.0
w_goal = 1.0
w_comfort = 0.1
```

### 3. Multi-Modal Prediction
For dynamic obstacles (pedestrians), we extend MPPI with:
- Multiple intention hypotheses
- Risk-aware cost (worst-case over predictions)

## Future Work

- **Learning Cost Functions**: Use inverse RL to learn from demonstrations
- **Receding Horizon Replanning**: Adaptive horizon based on scene complexity
- **GPU Acceleration**: Port MPPI to CUDA for 10x speedup

## Code

Full implementation: [SideWalker/jplanner/bev_mppi](https://github.com/blackuniverse0u0/perception_planning_control)

---

*Combining semantic perception with optimal control unlocks robust, real-time navigation in complex environments.*
