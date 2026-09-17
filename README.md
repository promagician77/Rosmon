# rosmon — ROS2 System Monitor

A real-time ROS2 system monitoring dashboard for robotic platforms running ROS2 + micro-ROS on embedded hardware.

Built for robotics teams managing multi-node ROS2 systems with embedded MCU integration (STM32, ESP32) via micro-ROS Agent.

## Features

- Live node graph with animated message flow visualization
- Per-node inspection: published/subscribed topics, message types, QoS profiles
- DDS middleware status (CycloneDDS / FastDDS)
- micro-ROS transport monitoring (Serial, UDP, Ethernet, CAN)
- MCU-to-SBC latency measurement
- Real-time CPU, memory, and frequency metrics
- Grouped node list: Perception, Estimation, Planning, Control, MCU, System

## Quick Start

### Option 1: Open directly

Open `index.html` in any modern browser. No build step, no dependencies.

### Option 2: Docker

```bash
docker compose up -d
```

Dashboard available at `http://localhost:3000`

### Option 3: Docker (manual)

```bash
docker build -t rosmon .
docker run -d -p 3000:80 rosmon
```

### Option 4: Any static hosting

Upload `index.html` to Nginx, Apache, Netlify, Vercel, Cloudflare Pages, or S3 + CloudFront.

Nginx example:

```bash
cp index.html /var/www/html/rosmon/index.html
```

### Option 5: Python dev server

```bash
python3 -m http.server 3000
```

## Tech Stack

- Vanilla HTML/CSS/JS — zero dependencies, zero build step
- JetBrains Mono typeface (loaded from Google Fonts CDN)
- SVG-based node graph with animated message particles
- CSS custom properties for dark/light theme support

## ROS2 Integration Notes

This is a standalone monitoring UI. To connect it to a live ROS2 system:

1. Implement a rosbridge_server WebSocket backend
2. Replace the static node data in `index.html` with live data from `/rosapi` topics
3. Subscribe to `/diagnostics`, `/tf`, and node lifecycle events

The current version uses representative demo data showing a typical mobile robot configuration:

| Layer | Nodes | Platform |
|---|---|---|
| Perception | lidar_driver, camera_node, imu_filter | SBC (Jetson/RPi) |
| Estimation | robot_localization (EKF), slam_toolbox | SBC |
| Planning | planner_server, controller_server, bt_navigator | SBC |
| Control | twist_mux, base_driver | SBC |
| MCU | pid_front_left, pid_front_right | STM32H743 via micro-ROS |
| System | diagnostic_agg, lifecycle_manager, micro_ros_agent | SBC |

## Project Structure

```
rosmon/
├── index.html          # Complete application (single file)
├── Dockerfile          # Production container build
├── docker-compose.yml  # One-command deployment
├── nginx.conf          # Production Nginx configuration
├── README.md           # This file
└── .gitignore
```

## License

MIT
