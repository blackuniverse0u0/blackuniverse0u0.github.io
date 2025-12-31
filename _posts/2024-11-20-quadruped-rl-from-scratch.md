---
layout: post
title: Learning Quadruped Locomotion with Reinforcement Learning - From Theory to Practice
date: 2024-11-20 14:30:00
description: A deep dive into training agile locomotion policies for legged robots using PPO and MuJoCo
tags: robotics reinforcement-learning locomotion deep-learning
categories: research
---

## Why Reinforcement Learning for Legged Robots?

Traditional control methods for quadrupeds (like Model Predictive Control) rely on:
- Accurate dynamics models
- Known terrain properties
- Hand-tuned gait parameters

**Reinforcement Learning** offers an alternative:
- Learn from trial and error in simulation
- Adapt to model uncertainties and terrain variations
- Discover emergent behaviors

But to use RL effectively, we need to understand **what we're learning** and **why it works**.

## The Foundation: Understanding Robot Dynamics

Before diving into RL, we built a solid foundation in rigid body mechanics:

### 1. Lie Groups for Rotations

Quadruped motion involves **SE(3)** transformations:
- SO(3): Body orientation in 3D space
- Translation: Center of mass position

Why Lie groups? They provide:
- Smooth parameterization of rotations
- Closed-form derivatives for optimization
- Geometric interpretation of motion

```python
# Example: Body velocity transformation
def body_velocity_to_spatial(V_b, T):
    """
    Transform body velocity to spatial velocity

    V_b: (6,) twist in body frame [omega; v]
    T: (4, 4) transformation matrix (SE(3))

    Returns:
        V_s: (6,) twist in spatial frame
    """
    R = T[:3, :3]
    p = T[:3, 3]

    # Adjoint transformation
    Ad_T = np.block([
        [R, np.zeros((3, 3))],
        [skew(p) @ R, R]
    ])

    return Ad_T @ V_b
```

### 2. Forward & Inverse Kinematics

**Forward Kinematics** (FK): Joint angles → foot positions
$$
p_{\text{foot}} = g_{\text{st}}(q_{\text{hip}}, q_{\text{thigh}}, q_{\text{calf}})
$$

**Inverse Kinematics** (IK): Desired foot position → joint angles
- Analytical solution for simple chains
- Numerical optimization for complex geometries

```python
# Simplified 3-DOF leg IK
def leg_ik(p_foot, l1, l2, l3):
    """
    Inverse kinematics for 3-joint leg

    p_foot: (3,) desired foot position [x, y, z]
    l1, l2, l3: link lengths

    Returns:
        q: (3,) joint angles [hip, thigh, calf]
    """
    # Distance from hip to foot
    r = np.linalg.norm(p_foot)

    # Calf angle (via law of cosines)
    cos_calf = (r**2 - l2**2 - l3**2) / (2 * l2 * l3)
    q_calf = np.arccos(np.clip(cos_calf, -1, 1))

    # Thigh angle
    beta = np.arctan2(p_foot[2], np.linalg.norm(p_foot[:2]))
    alpha = np.arccos((r**2 + l2**2 - l3**2) / (2 * r * l2))
    q_thigh = beta - alpha

    # Hip angle
    q_hip = np.arctan2(p_foot[1], p_foot[0])

    return np.array([q_hip, q_thigh, q_calf])
```

These building blocks are essential for:
- **Reward shaping**: Penalize impossible foot positions
- **Curriculum learning**: Start with standing, progress to walking
- **Sim-to-real transfer**: Accurate kinematic calibration

## Reinforcement Learning Setup

### Environment: MuJoCo MJX

We use **MuJoCo MJX** for GPU-accelerated parallel simulation:
- **10,000 environments** running in parallel
- **10-100x faster** than sequential simulation
- Differentiable physics for future work

```python
import jax
import mujoco
from mujoco import mjx

# Create parallel environments
@jax.jit
def parallel_step(state, action):
    """
    Step 10,000 environments in parallel on GPU
    """
    next_state = mjx.step(state, action)
    return next_state

# Vectorized rollout
states = jax.vmap(parallel_step)(batched_states, batched_actions)
```

### Observation Space

What does the robot "see"?

**Proprioceptive (Body sensors)**
- Base orientation (IMU): Roll, pitch, yaw
- Joint positions: 12 values (3 per leg)
- Joint velocities: 12 values
- Last action: 12 values (for temporal smoothness)

**Exteroceptive (World sensors)**
- Foot contact forces: 4 values
- Base linear velocity: 3 values
- Base angular velocity: 3 values

**Total**: 49-dimensional observation vector

**Key insight**: No vision! Pure proprioception forces the policy to learn robust, reactive behaviors.

### Action Space

**Joint torque commands** for 12 actuators:
$$
\tau = \pi(s) \in \mathbb{R}^{12}, \quad \tau_i \in [-33, 33] \text{ Nm}
$$

**PD controller** maps actions to actual torques:
$$
\tau_{\text{actual}} = K_p (q_{\text{target}} - q) + K_d (\dot{q}_{\text{target}} - \dot{q})
$$

where $q_{\text{target}} = \pi(s)$ is the network output.

### Reward Function

Designing a good reward is **80% of the work**:

```python
def compute_reward(state, action):
    """
    Reward components for stable, forward locomotion
    """
    # 1. Tracking linear velocity (encourage forward motion)
    v_target = [0.5, 0.0, 0.0]  # 0.5 m/s forward
    r_vel = -np.linalg.norm(state.v_linear - v_target)

    # 2. Tracking angular velocity (penalize spinning)
    r_ang = -np.linalg.norm(state.v_angular)

    # 3. Torque penalty (energy efficiency)
    r_torque = -0.0001 * np.sum(action**2)

    # 4. Alive bonus (don't fall)
    r_alive = 1.0 if state.z > 0.25 else 0.0

    # 5. Orientation penalty (stay upright)
    r_orient = -np.linalg.norm(state.rpy[:2])  # Penalize roll/pitch

    # 6. Foot slip penalty (stable contacts)
    r_slip = -np.sum(np.linalg.norm(state.foot_velocities, axis=1))

    # Total reward
    return r_vel + r_ang + 0.01*r_torque + r_alive + r_orient + 0.1*r_slip
```

## Training with PPO

**Proximal Policy Optimization** (PPO) is our workhorse algorithm:
- On-policy (stable, reproducible)
- Clipped surrogate objective (prevents destructive updates)
- Easy to tune

### PPO Update

1. **Collect rollouts**: Run policy for N steps (e.g., 24)
2. **Compute advantages**: Use Generalized Advantage Estimation (GAE)
   $$
   A_t = \sum_{k=0}^{\infty} (\gamma \lambda)^k \delta_{t+k}, \quad \delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)
   $$
3. **Update policy** via clipped objective:
   $$
   L_{\text{CLIP}} = \mathbb{E}\left[\min\left(r_t(\theta)A_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)A_t\right)\right]
   $$
   where $r_t(\theta) = \frac{\pi_\theta(a|s)}{\pi_{\theta_{\text{old}}}(a|s)}$ is the probability ratio.

### Network Architecture

Simple **MLP policy**:
```
Input (49) → FC(256) → ELU → FC(128) → ELU → FC(12) → Tanh → Output
```

**Why Tanh?** Bounded actions prevent extreme torques.

### Hyperparameters

```python
config = {
    'num_envs': 4096,        # Parallel environments
    'num_steps': 24,         # Rollout length
    'num_minibatches': 32,   # For minibatch SGD
    'num_epochs': 5,         # PPO epochs per rollout
    'gamma': 0.99,           # Discount factor
    'gae_lambda': 0.95,      # GAE parameter
    'clip_coef': 0.2,        # PPO clip coefficient
    'lr': 3e-4,              # Learning rate
}
```

## Training Curriculum

We don't throw the robot into the deep end immediately.

### Stage 1: Standing (0-1M steps)
- Reward upright posture
- Penalize base height deviation
- No velocity command

### Stage 2: Walking (1M-5M steps)
- Low velocity targets (0.2 m/s)
- Increase torque limits gradually
- Introduce foot contact rewards

### Stage 3: Agile Locomotion (5M-20M steps)
- Higher velocities (0.5-1.5 m/s)
- Randomize terrain (stairs, slopes)
- Command tracking (user-specified velocities)

## Results

After **20M timesteps (~2 hours on RTX 4090)**:

### Simulation Performance
- **Max velocity**: 1.8 m/s (stable trot gait)
- **Energy efficiency**: 0.8 J/m (similar to Unitree Go1)
- **Robustness**: Recovers from 30° pushes

### Emergent Behaviors
- **Trot gait**: Learned without explicit gait constraints
- **Foot clearance**: Lifts feet 5-10cm during swing
- **Body stabilization**: Keeps torso level on slopes

### Sim-to-Real Gap

Deployed on **Unitree Go1**:
- **Latency compensation**: Add observation delay (50ms)
- **Action filtering**: Exponential moving average of commands
- **Domain randomization**: Mass, friction, motor lag

**Real-world success rate**: 85% (95% with fine-tuning)

## Lessons Learned

### 1. Reward Engineering is Hard
- Started with 20+ reward terms → **Overfitted to weird gaits**
- Simplified to 6 core components → **Natural locomotion emerged**

### 2. Curriculum Learning is Essential
- Training from scratch on full task → **Failed to learn**
- Progressive difficulty → **Robust policies**

### 3. Vectorization = Speed
- Sequential simulation (1 env): 3 days to train
- Parallel simulation (4096 envs): **2 hours**

### 4. Regularization Helps Sim-to-Real
- **Action noise**: Add Gaussian noise during training
- **Observation noise**: Randomize IMU readings
- **Dynamics randomization**: Vary mass, friction, motor strength

## Future Directions

- **Vision-based locomotion**: Add depth cameras for terrain awareness
- **Contact-rich tasks**: Climbing stairs, opening doors
- **Multi-task learning**: Single policy for multiple gaits/speeds
- **Model-based RL**: Learn dynamics model for sample efficiency

## Code & Resources

- Full training code: [locomotion_RL_dynamics/krm_loco](https://github.com/blackuniverse0u0/locomotion_RL_dynamics)
- Paper: *Learning to Walk in Minutes Using Massively Parallel Deep RL* (inspired by)

---

*From rigid body theory to running robots - RL bridges the gap between mathematics and motion.*
