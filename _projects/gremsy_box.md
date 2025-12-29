---
layout: page
title: Gremsy Box
description: PTZ camera and gimbal control system with web interface and AI tracking
img:
importance: 2
category: robotics
---

## Overview

**Gremsy Box** is a comprehensive camera and gimbal control system built with Docker, providing API-based control, video streaming, and AI-powered tracking capabilities. The system integrates hardware control with modern web technologies for remote operation.

## Core Components

The system consists of three main services working together:

- **Gremsy API**: Handles communication and control for gimbal, camera, and other hardware
- **MediaMTX Proxy**: Manages and relays video streams from cameras
- **Web UI**: Provides a user-friendly web interface for viewing streams and controlling the API

## Key Features

- RESTful API for gimbal and camera control
- Real-time video streaming (RTSP/WebRTC)
- AI-enhanced features (auto-tracking, scan-surround)
- Dockerized deployment for easy setup
- Web-based control interface
- LRF (Laser Range Finder) integration

## Technology Stack

- **Containerization**: Docker
- **Streaming**: MediaMTX, RTSP, WebRTC
- **Backend**: Python, Flask
- **Frontend**: HTML5, JavaScript, CSS
- **AI Runtime**: NVIDIA Docker Runtime
- **Hardware Control**: Custom API layer

## Architecture

### 1. Gremsy API Server

The API server is the core backend controlling hardware. Available in two versions:

- **Base API**: Standard control for LRF, camera, gimbal
- **AI API**: Enhanced with auto-tracking and environmental scanning

### 2. MediaMTX Stream Proxy

MediaMTX proxies RTSP streams from cameras, making them accessible via WebRTC for the UI.

### 3. Web UI

Browser-based interface for viewing camera feeds and sending control commands.

## Quick Start

### Start Base API

```bash
docker run -it --restart unless-stopped --name gremsy_api \
  --network host \
  -e PORT=6783 \
  -e GREMSY_IP="192.168.12.240" \
  docker.argusvision.io/intel/krm-gremsy-api:0.3.0-start
```

### Access Web UI

Open your browser and navigate to:

```
http://<robot-ip>:7777
```

## Stream Access

- **RTSP**: `rtsp://<robot-ip>:8554/gremsy/`
- **WebRTC**: `http://<robot-ip>:8889/gremsy/`

## Project Link

[View on GitHub](https://github.com/blackuniverse0u0/vision_ptz_api_docker)
