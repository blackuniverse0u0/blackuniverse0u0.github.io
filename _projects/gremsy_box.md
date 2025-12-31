---
layout: page
title: Gremsy Box
description: Production-Grade PTZ Camera Control System with AI Tracking and Multi-Modal Streaming
img: assets/img/gremsy_box.png
importance: 2
category: robotics
---

## Overview

**Gremsy Box** is a comprehensive camera gimbal control system designed for **defense and surveillance robotics**. This production-ready platform integrates hardware control, AI-powered object detection and tracking, video streaming infrastructure, and a modern web interface—all containerized with Docker for seamless deployment on embedded systems and edge devices.

The system supports **dual-sensor payloads** (EO/IR - Electro-Optical and Infrared) with real-time AI tracking, making it ideal for autonomous surveillance, target tracking, and reconnaissance missions.

## System Architecture

### Three-Tier Design

```
┌─────────────────────────────────────────────────┐
│           Web UI (Browser Interface)            │
│  - Live video streaming (RTSP/WebRTC)          │
│  - Gimbal control & tracking interface          │
│  - AI detection visualization                   │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────┐
│         MediaMTX Streaming Server               │
│  - RTSP → WebRTC protocol conversion           │
│  - Multi-stream support (EO/IR)                │
│  - Low-latency streaming pipeline               │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────┴────────────────────────────────┐
│            Gremsy API Server                    │
│  ┌──────────────┬──────────────┬──────────────┐│
│  │  Base API    │   AI API     │ Detection    ││
│  │  (Control)   │  (Tracking)  │  Server      ││
│  └──────────────┴──────────────┴──────────────┘│
│  - Gimbal control (pan/tilt/zoom)              │
│  - LRF integration (Laser Range Finder)         │
│  - AI detection & tracking                      │
│  - PD control for target following              │
└─────────────────────────────────────────────────┘
```

## Core Components

### 1. Gremsy API - Hardware Control Layer

The API server provides **RESTful endpoints** for controlling all gimbal and camera functions:

**Base API Features**
- **Gimbal Control**: Precision pan/tilt positioning with configurable speed and acceleration
- **Camera Control**: Zoom, focus, aperture, exposure settings
- **LRF Integration**: Laser rangefinder for target distance measurement
- **Status Monitoring**: Real-time telemetry and health checks

**AI-Enhanced API**
- **Auto-Tracking**: PD control loop for smooth target following
- **Scan & Surround**: Autonomous perimeter surveillance
- **Multi-Object Tracking**: Simultaneous tracking of multiple targets
- **EO/IR Sensor Fusion**: Dual-sensor tracking for all-weather operation

```bash
# Example: Start Gremsy API with AI tracking
docker run -it --restart unless-stopped --name gremsy_api \
  --network host --runtime nvidia \
  -e PORT=6783 \
  -e GREMSY_IP="192.168.12.240" \
  docker.argusvision.io/intel/krm-gremsy-ai-api:latest
```

Location: `@projects/vision_ptz_api_docker/Gremsy_Box`

### 2. AI Detection & Tracking Pipeline

**Object Detection Server**
- **Real-Time Inference**: YOLO-based object detection at 30+ FPS
- **Custom Models**: Trained on defense-specific datasets (vehicles, personnel, drones)
- **Multi-Camera Support**: Process streams from multiple robots simultaneously

**Tracking Algorithm**
- **PD Control**: Proportional-Derivative controller for smooth gimbal motion
- **Kalman Filtering**: Predictive tracking for occluded targets
- **Auto-Zoom**: Dynamic zoom adjustment based on target distance

```python
# Detection server locations
@projects/vision_ptz_api_docker/detection/Detection_Inference
@projects/vision_ptz_api_docker/detection/krm_object_detect
```

**Key Features**
- **Low Latency**: <100ms from detection to gimbal command
- **Robust Tracking**: Handles partial occlusions and lighting variations
- **Multi-Modal**: Seamlessly switch between EO and IR sensors

### 3. MediaMTX - Video Streaming Infrastructure

**Protocol Translation**
- RTSP camera streams → WebRTC for browser playback
- Multi-bitrate adaptive streaming
- Sub-second latency for real-time applications

**Stream Management**
- Automatic reconnection on network failures
- Bandwidth-aware quality adjustment
- Support for H.264/H.265 codecs

```bash
# MediaMTX configuration for dual-sensor streams
rtsp://<robot-ip>:8554/eo_camera/
rtsp://<robot-ip>:8554/ir_camera/
```

### 4. Web UI - Operator Interface

**Modern React-Based Frontend**
- **Live Video**: Dual-stream display (EO + IR)
- **Gimbal Joystick**: Virtual joystick for manual control
- **Tracking Controls**: Enable/disable AI tracking, select targets
- **Telemetry Display**: Range, bearing, gimbal angles, zoom level

**Features**
- Responsive design for desktop and tablet
- Low-bandwidth mode for field operations
- Recording and screenshot capture
- Multi-robot dashboard

Access: `http://<robot-ip>:7777`

## Technical Highlights

### Docker-Native Deployment

**Multi-Service Orchestration**
```yaml
# docker-compose.yml
services:
  gremsy-api:
    image: krm-gremsy-ai-api:latest
    runtime: nvidia  # GPU acceleration for AI inference

  mediamtx:
    image: mediamtx:latest
    ports: [8554, 8889]  # RTSP + WebRTC

  web-ui:
    image: gremsy-ui:latest
    ports: [7777]
```

**Hardware Acceleration**
- NVIDIA Docker runtime for GPU-accelerated AI inference
- CUDA-optimized YOLO models
- TensorRT for production deployment

### EO/IR Dual-Sensor Support

**Electro-Optical (EO) Camera**
- High-resolution RGB imaging
- Zoom: 30x optical + 12x digital
- Auto-focus and exposure control

**Infrared (IR) Camera**
- Thermal imaging for night operations
- Temperature-based detection
- Contrast enhancement for low-visibility scenarios

**Sensor Fusion**
- Automatic sensor switching based on lighting conditions
- Multi-modal tracking (visual + thermal signatures)
- Coordinated pan/tilt for both sensors

### Real-Time PD Control Tracking

**Control Loop Architecture**
```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   YOLO       │────>│ PD Controller│────>│   Gimbal     │
│   Detector   │     │  (P=0.5,D=0.1│     │   Commands   │
└──────────────┘     └──────────────┘     └──────────────┘
        │                     │                     │
        └─────────────────────┴─────────────────────┘
              Feedback: Target position error
```

**Performance**
- Update Rate: 30 Hz (synchronized with camera framerate)
- Tracking Accuracy: <2° error at 100m distance
- Response Time: <50ms from detection to motion

## Technology Stack

- **Backend**: Python, Flask, OpenCV
- **AI**: PyTorch, YOLO, TensorRT
- **Streaming**: MediaMTX, RTSP, WebRTC
- **Frontend**: HTML5, JavaScript, WebRTC APIs
- **Deployment**: Docker, Docker Compose
- **Hardware**: NVIDIA Jetson Xavier, Gremsy T3V3 Gimbal

## Deployment Scenarios

### 1. Autonomous Surveillance Robot
```bash
# Full stack deployment on robot
docker-compose -f gremsy-full-stack.yml up -d

# Access web interface
firefox http://192.168.1.100:7777
```

### 2. Multi-Robot Fleet Management
- Central MediaMTX server for stream aggregation
- Distributed Gremsy API instances on each robot
- Unified web dashboard for fleet monitoring

### 3. Development & Testing
```bash
# Base API only (no AI)
docker run -it --name gremsy_api_base \
  -p 6783:6783 krm-gremsy-base-api:latest

# AI API with GPU
docker run -it --runtime nvidia --name gremsy_api_ai \
  -p 6783:6783 krm-gremsy-ai-api:latest
```

## API Endpoints

**Gimbal Control**
- `POST /gimbal/angle` - Absolute positioning
- `POST /gimbal/velocity` - Velocity control
- `GET /gimbal/status` - Current state

**Tracking**
- `POST /tracking/enable` - Start AI tracking
- `POST /tracking/target` - Select target by ID
- `GET /tracking/status` - Tracking state

**Camera**
- `POST /camera/zoom` - Set zoom level
- `POST /camera/focus` - Manual focus control

**LRF**
- `GET /lrf/range` - Get distance measurement

## Project Components

```
vision_ptz_api_docker/
├── Gremsy_Box/                    # Main project
│   ├── base_api/                  # Base gimbal control
│   ├── development/               # AI-enhanced API
│   │   └── app/
│   │       └── dev_ai/
│   │           └── tracker/       # PD control tracking
│   └── assets/                    # Web UI resources
├── detection/                     # AI detection servers
│   ├── Detection_Inference/       # Inference engine
│   └── krm_object_detect/         # Object detection models
└── docker-compose.yml             # Full deployment config
```

## Production Readiness

- **Reliability**: Auto-restart on failures, health monitoring
- **Security**: API authentication, encrypted streams (SRTP)
- **Performance**: <100ms end-to-end latency
- **Scalability**: Supports 10+ concurrent client connections
- **Monitoring**: Prometheus metrics, logging

## Future Enhancements

- **Multi-Agent Coordination**: Collaborative tracking across multiple robots
- **3D Tracking**: Integrate stereo vision for depth estimation
- **Edge AI**: Quantized models for lower-power devices
- **Advanced Autonomy**: Predictive tracking and behavior recognition

## Project Link

[View on GitHub](https://github.com/blackuniverse0u0/vision_ptz_api_docker)

---

*From hardware control to AI tracking - a complete vision system for autonomous robotics.*
