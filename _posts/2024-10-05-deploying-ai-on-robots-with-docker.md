---
layout: post
title: Deploying AI Vision Systems on Robots with Docker - From Development to Production
date: 2024-10-05 09:00:00
description: How we built a production-grade PTZ camera tracking system using Docker, NVIDIA runtime, and WebRTC streaming
tags: robotics docker deployment ai computer-vision
categories: engineering
---

## The Challenge: From Laptop to Robot

You've trained a great object detector. It works perfectly on your laptop with a GTX 3090. But when you try to deploy it on a robot with limited resources, you face:

- **Dependency hell**: Different CUDA versions, missing libraries, conflicting Python packages
- **Performance gaps**: 60 FPS on desktop → 5 FPS on Jetson Xavier
- **Integration complexity**: Need to combine detection, tracking, gimbal control, and video streaming

**Solution**: Containerize everything with Docker.

## System Overview: Gremsy Box

Our production system has three Docker services:

```
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│   Gremsy API  │────>│   MediaMTX    │────>│    Web UI     │
│  (Detection & │     │  (Streaming)  │     │  (Interface)  │
│   Tracking)   │     │               │     │               │
└───────────────┘     └───────────────┘     └───────────────┘
```

Each service runs independently, communicating via network APIs.

## Part 1: Containerizing AI Inference

### The Base Image

Start with NVIDIA's official images for GPU support:

```dockerfile
# Dockerfile.detection
FROM nvcr.io/nvidia/pytorch:23.10-py3

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libopencv-dev \
    python3-opencv \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r /app/requirements.txt

# Copy application code
COPY detection/ /app/detection/
WORKDIR /app
```

### Key Dependencies

```txt
# requirements.txt
torch==2.0.1
torchvision==0.15.2
opencv-python==4.8.0
ultralytics==8.0.120  # YOLOv8
flask==2.3.2
numpy==1.24.3
```

### Optimizing for Embedded GPUs

**Problem**: YOLOv8 is too slow on Jetson Xavier (15 FPS).

**Solution**: Convert to TensorRT.

```python
from ultralytics import YOLO

# Load PyTorch model
model = YOLO('yolov8n.pt')

# Export to TensorRT (INT8 quantization for Jetson)
model.export(format='engine', device=0, half=True, int8=True)

# Load TensorRT model
trt_model = YOLO('yolov8n.engine')

# Inference (3x faster!)
results = trt_model(frame)
```

**Performance gains**:
- PyTorch: 15 FPS
- TensorRT FP16: 35 FPS
- TensorRT INT8: **45 FPS** ✅

## Part 2: Docker Compose for Multi-Service Deployment

### docker-compose.yml

```yaml
version: '3.8'

services:
  # AI detection and tracking service
  gremsy-api:
    build:
      context: .
      dockerfile: Dockerfile.api
    runtime: nvidia  # Enable GPU
    environment:
      - NVIDIA_VISIBLE_DEVICES=0
      - GREMSY_IP=192.168.12.240
      - PORT=6783
    network_mode: host  # Direct hardware access
    restart: unless-stopped
    volumes:
      - ./models:/app/models:ro
      - ./logs:/app/logs

  # Video streaming server
  mediamtx:
    image: bluenviron/mediamtx:latest
    network_mode: host
    restart: unless-stopped
    volumes:
      - ./mediamtx.yml:/mediamtx.yml:ro
    environment:
      - MTX_PROTOCOLS=tcp,udp
      - MTX_RTSPADDRESS=:8554
      - MTX_WEBRTCADDRESS=:8889

  # Web interface
  web-ui:
    build:
      context: ./ui
      dockerfile: Dockerfile.ui
    ports:
      - "7777:80"
    restart: unless-stopped
    depends_on:
      - gremsy-api
      - mediamtx
```

### Deployment

```bash
# Build all services
docker-compose build

# Start in detached mode
docker-compose up -d

# View logs
docker-compose logs -f gremsy-api

# Restart specific service
docker-compose restart mediamtx
```

## Part 3: Real-Time Video Streaming with MediaMTX

### The Streaming Pipeline

```
Camera (RTSP) → MediaMTX → WebRTC → Browser
   H.264           Protocol      Low-latency
   30 FPS          conversion    <200ms
```

### MediaMTX Configuration

```yaml
# mediamtx.yml
paths:
  eo_camera:
    source: rtsp://192.168.12.240:554/stream1
    runOnReady: echo "EO camera connected"
    runOnDemand: echo "Client requested stream"

  ir_camera:
    source: rtsp://192.168.12.241:554/stream1

# WebRTC for browser playback
webrtc: yes
webrtcAddress: :8889

# RTSP server
rtsp: yes
rtspAddress: :8554
```

### Client-Side: WebRTC in Browser

```javascript
// Web UI: Display camera stream
const pc = new RTCPeerConnection();

// Fetch WebRTC offer from MediaMTX
const offer = await fetch('http://robot-ip:8889/eo_camera/whep')
  .then(res => res.text());

await pc.setRemoteDescription(new RTCSessionDescription({
  type: 'offer',
  sdp: offer
}));

// Create answer
const answer = await pc.createAnswer();
await pc.setLocalDescription(answer);

// Display video
pc.ontrack = (event) => {
  document.getElementById('video').srcObject = event.streams[0];
};
```

**Latency**: ~150ms (vs 1-2 seconds for RTSP over TCP)

## Part 4: Production Considerations

### 1. Health Checks & Restarts

Add health checks to detect and recover from failures:

```yaml
services:
  gremsy-api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:6783/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
```

### 2. Logging & Monitoring

Structured logging for debugging:

```python
import logging
import json

logger = logging.getLogger(__name__)

# JSON formatter for log aggregation
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_obj = {
            'timestamp': record.created,
            'level': record.levelname,
            'message': record.getMessage(),
            'module': record.module,
        }
        return json.dumps(log_obj)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
```

View logs:
```bash
# Real-time logs
docker-compose logs -f --tail=100 gremsy-api

# Export logs to file
docker-compose logs gremsy-api > api_logs.txt
```

### 3. Resource Limits

Prevent services from hogging resources:

```yaml
services:
  gremsy-api:
    deploy:
      resources:
        limits:
          cpus: '4.0'      # Max 4 CPU cores
          memory: 8G       # Max 8GB RAM
        reservations:
          cpus: '2.0'
          memory: 4G
```

### 4. Secrets Management

**Never hardcode API keys or passwords!**

```yaml
services:
  gremsy-api:
    environment:
      - GREMSY_IP=${GREMSY_IP}  # From .env file
    env_file:
      - .env.production
```

`.env.production`:
```bash
GREMSY_IP=192.168.12.240
API_KEY=your-secret-key-here
```

Add to `.gitignore`:
```
.env*
!.env.example
```

## Part 5: CI/CD for Robot Deployments

### GitHub Actions for Auto-Build

```yaml
# .github/workflows/docker-build.yml
name: Build and Push Docker Images

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build Docker image
        run: |
          docker build -t krm-gremsy-api:${{ github.sha }} .

      - name: Push to registry
        run: |
          docker tag krm-gremsy-api:${{ github.sha }} \
            registry.company.com/krm-gremsy-api:latest
          docker push registry.company.com/krm-gremsy-api:latest
```

### Deploying to Robot

```bash
# On robot (via SSH)
ssh robot@192.168.1.100

# Pull latest image
docker pull registry.company.com/krm-gremsy-api:latest

# Restart services
docker-compose pull
docker-compose up -d

# Zero-downtime update (blue-green)
docker-compose --project-name blue up -d
docker-compose --project-name green down
```

## Performance Metrics

After containerization:

| Metric | Before | After Docker |
|--------|--------|--------------|
| Setup time | 4 hours | **5 minutes** |
| Inference latency | 70ms | 22ms (TensorRT) |
| Video latency | 1.5s (RTSP) | **150ms** (WebRTC) |
| Deployment errors | Frequent | Rare |
| Rollback time | 1 hour | **30 seconds** |

## Lessons Learned

### 1. Network Mode Matters
- `network_mode: host` for hardware peripherals (cameras, serial)
- `bridge` for isolated services (web UI, databases)

### 2. GPU Memory Management
```python
# Clear cache between inferences
torch.cuda.empty_cache()

# Use model.half() for FP16 (saves 50% memory)
model = model.half()
```

### 3. Multi-Stage Builds Reduce Image Size

```dockerfile
# Build stage
FROM nvidia/cuda:11.8-devel AS builder
COPY . /app
RUN pip install -r requirements.txt

# Runtime stage (smaller)
FROM nvidia/cuda:11.8-runtime
COPY --from=builder /usr/local/lib/python3.10 /usr/local/lib/python3.10
COPY --from=builder /app /app
```

**Result**: 8GB → **2.5GB** image

### 4. Test Locally with Docker Compose

Before deploying to robot:
```bash
# Simulate robot environment
docker-compose -f docker-compose.test.yml up

# Run integration tests
pytest tests/integration/
```

## Conclusion

Docker transforms robot deployment from:
- ❌ Manual setup, dependency nightmares, "it works on my machine"

To:
- ✅ One-command deployment, reproducible environments, easy rollbacks

**Key benefits**:
- **Portability**: Same container runs on laptop, Jetson, cloud
- **Isolation**: Services don't interfere with each other
- **Scalability**: Easy to add more robots (just pull image)
- **Reproducibility**: Exact same environment every time

## Resources

- Code: [vision_ptz_api_docker/Gremsy_Box](https://github.com/blackuniverse0u0/vision_ptz_api_docker)
- MediaMTX: [bluenviron/mediamtx](https://github.com/bluenviron/mediamtx)
- NVIDIA Container Toolkit: [nvidia/nvidia-docker](https://github.com/NVIDIA/nvidia-docker)

---

*Containerization is not just for web apps - it's the future of robotics deployment.*
