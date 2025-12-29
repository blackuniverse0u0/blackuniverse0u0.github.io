---
layout: page
title: SideWalker
description: Autonomous robot navigation for urban sidewalks with Imitation and Reinforcement Learning
img:
importance: 1
category: robotics
---

## Overview

**SideWalker** is an autonomous robot navigation project for urban sidewalks, powered by state-of-the-art Imitation Learning and Reinforcement Learning techniques. We leverage realistic simulation environments using **MetaUrban** to train and validate robust navigation policies for wheeled or legged robots equipped with cameras or depth sensors.

## Key Features

- Vision-based navigation using only cameras or depth sensors
- Imitation Learning and Reinforcement Learning approaches
- MetaUrban simulation environment for realistic urban scenarios
- Modular architecture for observation, action, and reward components

## Project Structure

```
sidewalker/
│
├── action/               # Action spaces and control
├── assets/              # Project logos, images, diagrams
├── env/                 # MetaUrban-based environment wrappers
├── model/               # RL/IL vision-based agent
├── obs/                 # Observation: Depth, RGB, Segment
├── Reward/              # Reward function definitions
├── scripts/             # Training and evaluation scripts
└── utils/               # Utility functions, logging, eval
```

## Quick Start

#### Train with single processor
```python
python3 -m scripts.core
```

#### Validation with Visualization
```python
python3 -m scripts.core_viz
```

## Technology Stack

- **Simulation**: MetaUrban
- **Deep Learning**: PyTorch
- **Reinforcement Learning**: Custom RL/IL implementations
- **Perception**: RGB, Depth, Segmentation

## About MetaUrban

This project uses [MetaUrban](https://metadriverse.github.io/metaurban/) for simulating urban navigation environments with high fidelity and flexibility.

## Project Link

[View on GitHub](https://github.com/blackuniverse0u0/perception_planning_control)
