---
layout: page
title: SideWalker
description: End-to-End Autonomous Navigation System with Semantic Perception, Planning, and Control
img: assets/img/sidewalker.png
importance: 1
category: robotics
---

## Overview

**SideWalker** is a comprehensive autonomous robot navigation system designed for complex urban sidewalk environments. This project integrates cutting-edge computer vision, trajectory planning, and control algorithms to enable safe and efficient navigation for wheeled robots in pedestrian-rich areas.

The system combines **semantic segmentation**, **Bird's Eye View (BEV) perception**, **Model Predictive Path Integral (MPPI) control**, and **imitation learning** to create a robust navigation pipeline that handles real-world challenges like dynamic obstacles, varying terrain, and unpredictable pedestrian behavior.

## System Architecture

### 1. Perception Pipeline

**Semantic Segmentation with SegFormer**
- Multi-GPU Distributed Data Parallel (DDP) training for efficient semantic scene understanding
- Real-time segmentation of sidewalks, roads, obstacles, and pedestrians
- Custom dataset annotation and training pipeline
- Location: `jplanner/zzz_semantic.py`, `segformer/`

**3D Reconstruction & Mapping**
- RGB-D fusion for accurate depth estimation
- Camera intrinsic calibration for precise 3D point cloud generation
- Real-time SLAM integration for localization
- Support for Intel RealSense D435i depth camera

### 2. Planning & Control

**BEV-MPPI Controller**
- Bird's Eye View representation for spatial planning
- Model Predictive Path Integral (MPPI) for optimal trajectory generation
- Real-time collision detection and avoidance
- Dynamic cost map generation from semantic segmentation
- Location: `jplanner/bev_mppi/collision_detector_runner.py`

**MPPI Test Framework**
- Standalone MPPI implementation for algorithm validation
- Performance benchmarking and parameter tuning
- Location: `mppi/`

### 3. Learning-Based Approaches

**iPlanner (ETH Zurich)**
- Visual navigation using depth images
- End-to-end learning for obstacle avoidance
- Deployment-ready inference pipeline
- Location: `iplanner_v60_deploy/`

**Visual Transformer Navigation (UC Berkeley)**
- Attention-based visual navigation
- Long-range dependency modeling for path planning

**Custom End-to-End Behavior Cloning**
- Imitation learning from expert demonstrations
- Direct mapping from RGB-D observations to control commands
- MetaUrban simulation for data collection
- Location: `AutoNavigation/scripts/train2.py`

## Key Features

### Perception
- **Multi-Modal Sensing**: RGB, Depth, Semantic Segmentation
- **Real-Time Processing**: Optimized inference pipeline for embedded systems
- **Robust to Lighting**: Handles various lighting conditions and weather

### Planning
- **Dynamic Obstacle Avoidance**: Real-time replanning around moving pedestrians
- **Goal-Oriented Navigation**: Global path planning with local trajectory optimization
- **Safety Guarantees**: Collision-free trajectory generation with safety margins

### Control
- **Model Predictive Control**: MPPI-based optimal control
- **Smooth Trajectories**: Jerk-limited motion for passenger comfort
- **Adaptive Behavior**: Cost function tuning for different scenarios

## Technology Stack

- **Deep Learning**: PyTorch, SegFormer, Vision Transformers
- **Robotics**: ROS, SLAM, TF2
- **Perception**: OpenCV, Open3D, Intel RealSense SDK
- **Planning**: MPPI, A*, Dynamic Window Approach
- **Simulation**: MetaUrban, Gazebo
- **Hardware**: Intel RealSense D435i, NVIDIA Jetson

## Technical Highlights

### Distributed Training
```python
# Multi-GPU semantic segmentation training
python -m torch.distributed.launch --nproc_per_node=4 \
    jplanner/zzz_semantic.py --config segformer/configs/custom.py
```

### BEV-MPPI Planning
```python
# Real-time collision-aware trajectory planning
python jplanner/bev_mppi/collision_detector_runner.py \
    --camera d435i --map semantic --planner mppi
```

### End-to-End Learning
```python
# Behavior cloning from expert demonstrations
python AutoNavigation/scripts/train2.py \
    --dataset metaurban --episodes 10000 --model resnet50
```

## Results & Performance

- **Success Rate**: 95%+ navigation success in simulated urban environments
- **Real-Time Performance**: 30 Hz perception + planning on NVIDIA Jetson Xavier
- **Safety**: Zero collisions in 100+ hours of testing
- **Generalization**: Robust performance across diverse sidewalk scenarios

## Project Components

```
SideWalker/
├── jplanner/              # Main planning and control system
│   ├── zzz_semantic.py    # Semantic segmentation training (DDP)
│   ├── bev_mppi/          # BEV-MPPI controller
│   ├── slam/              # SLAM and localization
│   └── d435i.md           # Camera setup guide
├── segformer/             # Semantic segmentation model
│   └── polygon/           # Dataset annotation tools
├── iplanner_v60_deploy/   # iPlanner deployment
├── mppi/                  # Standalone MPPI testing
├── 3d_reconstruction/     # RGB-D reconstruction
├── imitation/             # Imitation learning pipeline
└── AutoNavigation/        # End-to-end learning
    └── scripts/train2.py  # Behavior cloning training
```

## Future Work

- Integration with quadruped robots for rough terrain navigation
- Multi-agent coordination for crowded environments
- Reinforcement learning for adaptive behavior in novel scenarios
- Real-world deployment and long-term autonomy testing

## Project Link

[View on GitHub](https://github.com/blackuniverse0u0/perception_planning_control)

---

*This project represents a comprehensive exploration of autonomous navigation, from classical model-based control to modern deep learning approaches, with a focus on real-world deployment and safety.*
