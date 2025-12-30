---
layout: page
title: Physics to Robot Locomotion
description: Building quadruped robot controllers with Model-Based Control and JAX-accelerated RL
img: assets/img/locomotion_rl.png
importance: 3
category: robotics
---

![Python](https://img.shields.io/badge/Python-3.11%2B-blue?logo=python)
![MuJoCo](https://img.shields.io/badge/Sim-MuJoCo-orange?logo=mujoco)
![JAX](https://img.shields.io/badge/Library-JAX%2FMJX-green?logo=google-jax)
![Control](https://img.shields.io/badge/Method-MPC_%26_RL-purple)

## Overview

Building a quadruped robot controller from scratch: Bridging Rigid Body Dynamics, Model-Based Control (MPC), and JAX-accelerated Reinforcement Learning.

Instead of treating RL as a "black box," this project emphasizes understanding the underlying physics, kinematics, and classical control theories to build robust and explainable locomotion policies.

**Our Goal: Scalable Perceptive Locomotion with Reliability & Safety for Real World.**

## Key Features

- **Model-Based Optimization**: Convex MPC for dynamic locomotion
- **Model-Free RL**: JAX-accelerated reinforcement learning with PPO
- **Physics-First Approach**: Understanding rigid body dynamics and classical control
- **Sim-to-Real**: Bridging simulation (MuJoCo) to real robot deployment
- **Scalable**: JAX/MJX acceleration for parallel training

## Tech Stack

- **Simulation**: [MuJoCo](https://mujoco.org/)
- **Language**: Python 3.11+
- **Math & Physics**: NumPy, SciPy
- **Optimization**: OSQP, CVXPY (for MPC)
- **Reinforcement Learning**: PyTorch, JAX, Brax, MuJoCo MJX, [rsl_rl](https://github.com/leggedrobotics/rsl_rl)

## Approach

### 1. Model-Based Control (MPC)

Following MIT Cheetah 3's convex model-predictive control approach for dynamic locomotion.

### 2. Model-Free RL

Using PPO with rsl_rl framework for learning complex behaviors.

### 3. State Estimation

Robust state estimation techniques for real-world deployment.

## Project Components

The repository includes multiple sub-projects:

- **go1_joystick**: Joystick control interface for Unitree Go1
- **krm_loco**: Core locomotion framework with MuJoCo MPC
- **physics_to_robot**: Sim-to-real transfer pipeline

## Getting Started

```bash
# Clone the repository
git clone https://github.com/Korea-Robot/krm_loco.git
cd krm_loco

source install_setup.sh

python Furo/cosine_moving.py
```

## References

1. **Theory**: _Modern Robotics: Mechanics, Planning, and Control_ (Lynch & Park), _Reinforcement Learning: An Introduction_ (Sutton & Barto)
2. **MPC**: Dynamic Locomotion in the MIT Cheetah 3 Through Convex Model-Predictive Control
3. **Reference Code**: [google-deepmind/mujoco_mpc](https://github.com/google-deepmind/mujoco_mpc) & [leggedrobotics/rsl_rl](https://github.com/leggedrobotics/rsl_rl)

## License

This project is released under the **MIT License**, with acknowledgments to:

- Google DeepMind (MuJoCo MPC & Playground - Apache 2.0)
- ETH Zurich RSL (rsl_rl)

## Project Link

[View on GitHub](https://github.com/blackuniverse0u0/locomotion_RL_dynamics)
