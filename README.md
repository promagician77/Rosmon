# rosmon — ROS2 System Monitor

A real-time ROS2 system monitoring dashboard for robotic platforms running ROS2 + micro-ROS on embedded hardware.

Built for robotics teams managing multi-node ROS2 systems with embedded MCU integration (STM32, ESP32) via micro-ROS Agent.

## Features

**Graph** — live node graph with animated message flow, group-coloured node plates,
click or arrow-key inspection, and a per-node publish-rate trace in the inspector.

**Topics** — filterable by group and time window:

- Publish rate by group (streaming multi-series timeline)
- Bandwidth by topic (top 10, KiB/s)
- Message size distribution
- Sortable topic table: type, publisher, subscriber count, rate, bandwidth, QoS

**Diagnostics** — SBC and MCU health:

- Resource utilisation (CPU, memory, package temperature)
- MCU→SBC round-trip latency histogram with p50/p95/p99 markers
- Executor budget per node
- Callback jitter heatmap
- Lifecycle and diagnostic event log

**Throughout**

- Per-node inspection: published/subscribed topics, message types, QoS profiles
- DDS middleware status (CycloneDDS / FastDDS)
- micro-ROS transport monitoring (Serial, UDP, Ethernet, CAN)
- Grouped node list: Perception, Estimation, Planning, Control, MCU, System
- Dark and light themes, following the OS by default and remembered per browser
- Every chart has a table view, so no value is reachable only through a tooltip

## Design system

The chart palette is not hand-picked. Both themes were validated with the
data-viz six-checks validator against the surface each one actually renders on
(OKLCH lightness band, chroma floor, CVD separation under simulated
protan/deutan, normal-vision separation, and WCAG contrast):

| Group | Dark | Light |
|---|---|---|
| Perception | `#23a081` | `#008f72` |
| State Estimation | `#5183d4` | `#2f68c4` |
| Planning | `#d4566a` | `#c33049` |
| Control | `#9a7bdd` | `#6f52c9` |
| MCU (micro-ROS) | `#bd8620` | `#9a6a12` |
| System | `#1f9cbe` | `#0080a8` |

Groups are ordered so that adjacent hues alternate warm and cool, which is what
keeps them separable for red-green colourblind readers:

- dark — worst adjacent CVD ΔE **15.3**, normal-vision ΔE **17.6**, all ≥3:1
- light — worst adjacent CVD ΔE **18.6**, normal-vision ΔE **19.5**, all ≥3:1

(ΔE is OKLab ×100; the target is ≥8 for CVD and ≥15 for normal vision.)

Colour follows the entity, never its rank — filtering or re-sorting never
repaints a group. Status colours (`good` / `warning` / `serious` / `critical`)
are reserved, never reused as a series colour, and always ship with an icon and
a text label so state never rests on hue alone. The jitter heatmap uses a
single-hue sequential ramp with monotonic lightness, never a rainbow.

No chart uses a second y-axis. Where measures of different units share a plot
(CPU %, memory, temperature) each is expressed as a share of its own capacity on
one 0–100% axis, with absolute values in the tooltip.

## Quick Start

### Option 1: Open directly

Open `index.html` in any modern browser. No build step.

Keep `vendor/` next to `index.html` — the charting library is vendored there so
the dashboard works on an isolated robot network with no internet access.

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
mkdir -p /var/www/html/rosmon
cp -r index.html vendor /var/www/html/rosmon/
```

### Option 5: Python dev server

```bash
python3 -m http.server 3000
```

## Tech Stack

- Vanilla HTML/CSS/JS — zero build step, no package manager, no bundler
- [Apache ECharts](https://echarts.apache.org/) 5.5.1 for charting, vendored in
  `vendor/` so the dashboard runs fully offline (Apache-2.0)
- Inter (UI) and JetBrains Mono (identifiers, values, axis ticks), loaded from
  Google Fonts and falling back to `system-ui` / `ui-monospace` when offline
- Inline SVG icon sprite — no icon font, no icon package
- SVG node graph with animated message particles
- CSS custom properties for dark/light theming, honouring
  `prefers-color-scheme` and `prefers-reduced-motion`

Why ECharts over the alternatives: it is the only option that covers every form
this dashboard needs — streaming time series, histograms, horizontal bars and a
heatmap — from one canvas-rendered library that stays smooth while several
charts update once a second. Chart.js has a thinner chart vocabulary, uPlot is
faster but has no heatmap or bar forms, and ApexCharts renders to SVG, which
gets heavy at this update rate and node count.

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
├── index.html          # Complete application (markup, styles, logic, icons)
├── vendor/
│   └── echarts.min.js  # Charting library, vendored for offline use
├── Dockerfile          # Production container build
├── docker-compose.yml  # One-command deployment
├── nginx.conf          # Production Nginx configuration
├── README.md           # This file
└── .gitignore
```

## License

MIT
