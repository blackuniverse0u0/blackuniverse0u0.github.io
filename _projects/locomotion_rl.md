---
layout: page
title: Quadruped Locomotion
description: From Rigid Body Dynamics to Deep RL - A Complete Pipeline for Legged Robot Control
img: assets/img/locomotion_rl.png
importance: 3
category: robotics
---

## Overview

This project represents a **ground-up approach to quadruped robot control**, starting from fundamental rigid body dynamics and progressing to state-of-the-art deep reinforcement learning. Rather than treating RL as a black box, this work emphasizes understanding the underlying physics, kinematics, and classical control theory to build robust and explainable locomotion policies.

**Philosophy**: Understanding the "why" behind every equation and algorithm, from Lie algebra to policy gradients.

**Goal**: Scalable, perceptive locomotion with reliability and safety for real-world deployment.

## Mathematical Foundations

### 1. Rigid Body Dynamics & Lie Theory

**Special Orthogonal Group SO(3) & Special Euclidean Group SE(3)**
- Rotation matrices and transformation matrices
- Exponential coordinates and logarithmic maps
- Adjoint representations for velocity transformations

**Screw Theory & Product of Exponentials (PoE)**
- Twist coordinates for rigid body motion
- Exponential formula for forward kinematics
- Geometric interpretation of robot motion

**Body & Spatial Jacobians**
- Velocity kinematics and manipulability
- Force-torque relationships
- Singularity analysis

Location: `physics_to_robot/mujoco_physics/rigid_body_dynamics/`

### 2. Forward Kinematics

**Analytical Forward Kinematics**
- DH parameters and transformation chains
- Product of Exponentials formulation
- Efficient computation for control loops

**Implementation**
```python
# Clean, modular FK implementation
@physics_to_robot/mujoco_physics/forward_kinematics/
@physics_to_robot/mujoco_physics/robot/fk.py
```

### 3. Inverse Kinematics

**Numerical IK Solver**
- Jacobian-based iterative methods
- Levenberg-Marquardt algorithm
- Joint limit constraints and singularity handling

**Applications**
- Foot position control for trot gait
- Body pose stabilization
- Trajectory tracking

```python
# IK solver with configuration space constraints
@physics_to_robot/mujoco_physics/robot/ik.py
```

## Control Methods

### 1. Model-Based Control: Quadruped Trot Gait

**Gait Generation**
- Phase-based trot pattern generation
- Swing and stance leg coordination
- Smooth foot trajectories with polynomial interpolation

**Stability Control**
- Body orientation stabilization
- Center of Mass (CoM) tracking
- Ground reaction force distribution

**MuJoCo Simulation**
```python
# Unitree A1 trot controller
@physics_to_robot/mujoco_physics/trot/a1_trot/mj_a1_trot.py

# Furo quadruped dynamics
@physics_to_robot/mujoco_physics/trot/Furo/
```

### 2. Model Predictive Control (MPC)

**Convex MPC Framework**
- Linearized dynamics for real-time optimization
- QP-based trajectory optimization
- Receding horizon control

**Inspired by MIT Cheetah 3**
- Single rigid body dynamics approximation
- Ground reaction force optimization
- Foothold planning

### 3. Reinforcement Learning

**Deep RL for Locomotion**
- Proximal Policy Optimization (PPO)
- Teacher-student distillation for sim-to-real transfer
- Domain randomization for robustness

**Training Frameworks**
```python
# Baseline RL training with Unitree Go1
@locomotion_RL_dynamics/go1_joystick/baseline/

# MuJoCo MJX-accelerated training (GPU parallelization)
@krm_loco/mujoco_playground/learning/train_rsl_rl.py
```

**Key Features**
- JAX/MJX acceleration for 10-100x faster training
- Curriculum learning for complex behaviors
- Privileged teacher network for realistic simulation

## Technical Stack

### Simulation & Physics
- **MuJoCo**: High-fidelity physics simulation
- **MuJoCo MJX**: GPU-accelerated parallel simulation
- **Brax**: Differentiable physics for JAX

### Optimization & Control
- **OSQP**: Quadratic programming solver for MPC
- **CVXPY**: Convex optimization modeling
- **NumPy/SciPy**: Numerical computation

### Deep Learning & RL
- **PyTorch**: Deep learning framework
- **JAX**: High-performance numerical computing
- **rsl_rl**: Reinforcement learning library (ETH Zurich RSL)
- **Brax**: JAX-based RL environments

## Project Structure

```
locomotion_RL_dynamics/
├── physics_to_robot/              # Fundamental robotics
│   ├── mujoco_physics/
│   │   ├── forward_kinematics/    # FK implementation
│   │   ├── robot/
│   │   │   ├── fk.py             # Forward kinematics solver
│   │   │   └── ik.py             # Inverse kinematics solver
│   │   └── trot/
│   │       ├── a1_trot/          # Unitree A1 trot controller
│   │       ├── Furo/             # Furo quadruped
│   │       └── unitree_robotics_a1/
│   ├── mujoco_playground/         # MuJoCo experiments
│   └── rsl_rl/                    # RL training framework
├── go1_joystick/                  # Unitree Go1 teleoperation
│   └── baseline/                  # Baseline RL training
└── krm_loco/                      # Advanced locomotion framework
    ├── mujoco_playground/
    │   └── learning/
    │       └── train_rsl_rl.py    # JAX-accelerated RL training
    ├── brax/                      # Brax RL environments
    ├── loco-mujoco/               # Locomotion benchmarks
    └── rsl_rl/                    # RL algorithms
```

## Key Achievements

### Mathematical Rigor
- Derived all kinematic equations from first principles
- Implemented Lie group operations for SE(3)
- Understanding of geometric mechanics for legged locomotion

### Simulation Performance
- **10-100x speedup** with MuJoCo MJX GPU parallelization
- Thousands of parallel environments for sample-efficient learning
- Real-time MPC at 1kHz control frequency

### Learning Efficiency
- PPO-based policies trained in <1 hour on consumer GPU
- Curriculum learning for progressive skill acquisition
- Sim-to-real transfer with domain randomization

## Research Inspirations

### Theory
- **Modern Robotics** (Lynch & Park): Kinematics, dynamics, and control
- **Reinforcement Learning: An Introduction** (Sutton & Barto): Foundations of RL

### Control
- **MIT Cheetah 3**: Convex Model-Predictive Control for dynamic locomotion
- **ANYmal (ETH Zurich)**: Perceptive locomotion in challenging terrain

### Code References
- [google-deepmind/mujoco_mpc](https://github.com/google-deepmind/mujoco_mpc) - MuJoCo MPC examples
- [leggedrobotics/rsl_rl](https://github.com/leggedrobotics/rsl_rl) - RL for legged robots
- [google/brax](https://github.com/google/brax) - JAX-based physics and RL

## Future Directions

- **Perceptive Locomotion**: Vision-based terrain estimation and adaptive gait
- **Whole-Body Control**: Manipulation during locomotion
- **Sim-to-Real Transfer**: Deploy learned policies on hardware
- **Multi-Contact Planning**: Beyond periodic gaits for extreme terrains

## Learning Philosophy

> "Don't just use the tools—understand how they work. Every abstraction hides important details."

This project is a journey from:
- Rotation matrices → Optimal control
- Inverse kinematics → Deep reinforcement learning
- Mathematical derivations → Working robot controllers

The goal is not just to make a robot walk, but to **understand every step** from physics to learning algorithms.

## Project Link

[View on GitHub](https://github.com/blackuniverse0u0/locomotion_RL_dynamics)

---

*Building intelligent robots requires understanding both the mathematics of motion and the algorithms of learning.*
