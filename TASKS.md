# Task Checklist: Hardware Monitoring Desktop App

- [x] **Phase 1: Project Initialization & Setup**
  - [x] Initialize Tauri v2 project (React, TypeScript, Vite)
  - [x] Configure Tailwind CSS v3 & PostCSS
  - [x] Install & Configure `shadcn/ui` (utils, components.json)
  - [x] Setup folder structure as per implementation plan
  - [x] Setup Theme Provider (Light/Dark mode)
  - [x] Install dependencies: Recharts, Framer Motion, Lucide React

- [x] **Phase 2: Rust Backend (System Monitor)**
  - [x] Add crates: `sysinfo`, `serde`, `serde_json`
  - [x] Add crate: `nvml-wrapper` (Nvidia GPU support)
  - [x] Create `models/stats.rs` - Data structures
  - [x] Create `services/monitor.rs` - SystemMonitor polling logic
  - [x] Create `commands/system_stats.rs` - Tauri command handler
  - [x] Implement background thread to emit `system-stats` events (1s interval)

- [x] **Phase 3: Frontend Implementation (Main Window)**
  - [x] Create `features/dashboard/` - Dashboard layout
  - [x] Create `CpuCard`, `RamCard`, `GpuCard` components
  - [x] Create `SystemChart` - Realtime area chart
  - [x] Create `hooks/useTauriEvent.ts` - Listen to Tauri events
  - [x] Integrate charts with realtime data

- [x] **Phase 4: Mini Mode (Popup/Overlay)**
  - [x] Configure secondary window in `tauri.conf.json`
  - [x] Create `features/mini-mode/` components
  - [x] Implement toggle between Main ↔ Mini mode
  - [x] Style transparent, always-on-top widget
  - [x] **BONUS**: LibreHardwareMonitor sidecar for CPU/GPU temperature

- [x] **Phase 5: Polish & Build**
  - [x] Add animations (Framer Motion) - Card hover effects, entrance animations
  - [x] Optimize Rust release build (LTO, strip, codegen-units=1)
  - [x] Setup sccache for faster builds
  - [x] UAC manifest for admin elevation
  - [x] Fix capabilities mismatch (hw-monitor → lhm-sidecar)
  - [ ] Test accuracy vs Windows Task Manager
  - [ ] Build production installer (NSIS)

- [ ] **Phase 6: FPS Tracking (Future)**
  - [ ] Research RTSS/PresentMon integration
  - [ ] Implement FPS overlay for games
  - [ ] Windows Service for zero-UAC experience (optional)
