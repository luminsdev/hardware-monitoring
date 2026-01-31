# Task Checklist: Hardware Monitoring Desktop App

- [ ] **Phase 1: Project Initialization & Setup**
  - [ ] Initialize Tauri v2 project (React, TypeScript, Vite)
  - [ ] Configure Tailwind CSS v3 & PostCSS
  - [ ] Install & Configure `shadcn/ui` (utils, components.json)
  - [ ] Setup folder structure as per implementation plan
  - [ ] Setup Theme Provider (Light/Dark mode)
  - [ ] Install dependencies: Recharts, Framer Motion, Lucide React

- [ ] **Phase 2: Rust Backend (System Monitor)**
  - [ ] Add crates: `sysinfo`, `serde`, `serde_json`
  - [ ] Add crate: `nvml-wrapper` (Nvidia GPU support)
  - [ ] Create `models/stats.rs` - Data structures
  - [ ] Create `services/monitor.rs` - SystemMonitor polling logic
  - [ ] Create `commands/system_stats.rs` - Tauri command handler
  - [ ] Implement background thread to emit `system-stats` events (1s interval)

- [ ] **Phase 3: Frontend Implementation (Main Window)**
  - [ ] Create `features/dashboard/` - Dashboard layout
  - [ ] Create `CpuCard`, `RamCard`, `GpuCard` components
  - [ ] Create `SystemChart` - Realtime area chart
  - [ ] Create `hooks/useTauriEvent.ts` - Listen to Tauri events
  - [ ] Integrate charts with realtime data

- [ ] **Phase 4: Mini Mode (Popup/Overlay)**
  - [ ] Configure secondary window in `tauri.conf.json`
  - [ ] Create `features/mini-mode/` components
  - [ ] Implement toggle between Main ↔ Mini mode
  - [ ] Style transparent, always-on-top widget

- [ ] **Phase 5: Polish & Build**
  - [ ] Add animations (Framer Motion)
  - [ ] Optimize Rust release build
  - [ ] Test accuracy vs Windows Task Manager
  - [ ] Build production installer
