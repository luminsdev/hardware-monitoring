# Implementation Plan - Hardware Monitor App

## User Review Required

> [!IMPORTANT]
> **GPU & FPS Monitoring**:
>
> - Getting granular GPU data (Temp, Fan, Clock) often requires vendor-specific SDKs (NVML for Nvidia, ADL for AMD). We will start with standard polling via system APIs, which might be limited.
> - **FPS Monitoring** is complex and typically requires "hooking" into game processes. For Phase 1, we will focus on System Resources (CPU/RAM/GPU Utilization).

## Tech Stack Decisions

### Frontend Framework

- **Core**: React 18+ (Vite)
- **Styling**: Tailwind CSS v3 + **shadcn/ui**
  - _Rationale_: Tailwind v4 is currently in Beta/RC. v3 is the stable choice for shadcn/ui. `shadcn/ui` is critical for achieving the "Premium/Clean" look efficiently as it provides pre-built, accessible components (Cards, Switches, Dialogs) that we can theme easily.
- **State Management**: React Query or simple React Context (for local app state) + Tauri Event system for headers.
- **Charts**: Recharts (High performance for realtime time-series data).

### Backend (Tauri/Rust)

- **Framework**: Tauri v2
- **System Info Crate**: `sysinfo` (Primary), `nvml-wrapper` (Nvidia specific features optional later).
- **Window Management**: Multi-window approach (Main Dashboard + Overlay Widget).

---

## Project Folder Structure (Real-World, Scalable)

```
hardware-monitoring/
├── src-tauri/                      # Rust Backend (Tauri Core)
│   ├── src/
│   │   ├── lib.rs                  # Main library entry, Tauri setup
│   │   ├── main.rs                 # Bootstrap (calls lib)
│   │   ├── commands/               # Tauri Commands (IPC handlers)
│   │   │   ├── mod.rs
│   │   │   ├── system_stats.rs     # get_cpu, get_ram, get_gpu
│   │   │   └── window.rs           # toggle_mini_mode, etc.
│   │   ├── services/               # Business Logic (decoupled from Tauri)
│   │   │   ├── mod.rs
│   │   │   ├── monitor.rs          # SystemMonitor struct, polling loop
│   │   │   └── gpu.rs              # GPU-specific logic (NVML, etc.)
│   │   ├── models/                 # Data structures
│   │   │   ├── mod.rs
│   │   │   └── stats.rs            # CpuStats, RamStats, GpuStats
│   │   └── utils/                  # Helpers
│   │       ├── mod.rs
│   │       └── error.rs            # Custom error types
│   ├── Cargo.toml
│   ├── tauri.conf.json             # Tauri config (windows, permissions)
│   └── capabilities/               # Tauri v2 permissions
│
├── src/                            # React Frontend
│   ├── main.tsx                    # React entry point
│   ├── App.tsx                     # Root component, routing
│   ├── index.css                   # Global styles, Tailwind imports
│   │
│   ├── components/                 # Reusable UI Components
│   │   ├── ui/                     # shadcn/ui primitives (auto-generated)
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   ├── common/                 # App-wide shared components
│   │   │   ├── ThemeToggle.tsx
│   │   │   ├── Logo.tsx
│   │   │   └── WindowControls.tsx  # Custom title bar buttons
│   │   └── charts/                 # Chart components
│   │       ├── AreaChart.tsx
│   │       └── GaugeChart.tsx
│   │
│   ├── features/                   # Feature-based modules
│   │   ├── dashboard/              # Main Dashboard feature
│   │   │   ├── components/
│   │   │   │   ├── CpuCard.tsx
│   │   │   │   ├── RamCard.tsx
│   │   │   │   └── GpuCard.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useSystemStats.ts
│   │   │   └── index.tsx           # Dashboard page
│   │   │
│   │   └── mini-mode/              # Mini Overlay feature
│   │       ├── components/
│   │       │   └── CompactWidget.tsx
│   │       └── index.tsx
│   │
│   ├── hooks/                      # Global custom hooks
│   │   ├── useTauriEvent.ts        # Listen to Tauri events
│   │   └── useTheme.ts
│   │
│   ├── lib/                        # Utilities & Tauri bindings
│   │   ├── tauri.ts                # invoke() wrappers, type-safe commands
│   │   ├── utils.ts                # cn(), formatBytes(), etc.
│   │   └── constants.ts
│   │
│   ├── providers/                  # React Context Providers
│   │   └── ThemeProvider.tsx
│   │
│   └── types/                      # TypeScript types
│       └── stats.ts                # Mirrors Rust models
│
├── public/                         # Static assets
│   └── icons/
│
├── .vscode/                        # Editor settings
├── package.json
├── vite.config.ts
├── tailwind.config.js
├── tsconfig.json
└── README.md
```

### Giải thích cấu trúc

| Layer                 | Mục đích                                                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `src-tauri/commands/` | Xử lý IPC (gọi từ Frontend). Mỗi file = 1 domain.                                                                                     |
| `src-tauri/services/` | Business logic thuần Rust. Dễ unit test, không phụ thuộc Tauri.                                                                       |
| `src/features/`       | **Feature-first architecture**. Mỗi feature chứa components, hooks riêng. Dễ scale khi thêm feature mới (Settings, Notifications...). |
| `src/components/ui/`  | shadcn/ui components (được generate bằng CLI). Không chỉnh sửa trực tiếp.                                                             |
| `src/lib/tauri.ts`    | Type-safe wrapper cho `invoke()`. Tránh hardcode command names khắp nơi.                                                              |

## Proposed Features

### 1. UI/UX Architecture

- **Theme**: System-aware Dark/Light mode with user override.
- **Main Dashboard**:
  - Glassmorphism effect (Blur background).
  - Modern "Cards" layout for each metrics.
  - Animated gauge bars / wave charts.

### 2. "Mini Mode" (Popup Widget)

- Uses a separate Tauri Window labeled `mini_monitor`.
- **Properties**:
  - `decorations`: false (No title bar)
  - `transparent`: true
  - `alwaysOnTop`: true
  - `skipTaskbar`: true (optional)
- **Functionality**: A toggle button on the Main Dashboard will hide the Main Window and show the Mini Window, and vice-versa.

## Implementation Steps

### Backend (Rust)

#### [NEW] `src-tauri/src/lib.rs`

- Logic to initialize `sysinfo::System`.
- A loop running in a separate thread `spawn` that updates stats every 500ms-1s.
- `emit("system-stats", payload)` to send JSON data to frontend.

### Frontend

#### [NEW] `src/features/dashboard/components/`

- `CpuCard.tsx`, `RamCard.tsx`, `GpuCard.tsx`: Stats cards with gauges.
- `SystemChart.tsx`: Realtime area chart.

#### [NEW] `src/features/mini-mode/components/`

- `CompactWidget.tsx`: Minimalist overlay view of stats.

## Verification Plan

### Automated Tests

- Run `cargo test` for backend logic (ensure sysinfo initializes).
- `npm run build` to verify frontend compilation.

### Manual Verification

- **Realtime Check**: Compare app values with Windows Task Manager.
- **Mini Mode**: Verify clicking "Mini Mode" opens the widget and it stays on top of other windows (like Chrome/Game).
- **Theme**: Toggle Light/Dark mode to ensure consistency.
