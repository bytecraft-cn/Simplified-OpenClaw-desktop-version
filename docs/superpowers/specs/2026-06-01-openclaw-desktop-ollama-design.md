# OpenClaw Desktop (Ollama Edition) - Design Spec

## Overview

A desktop installer application that sets up OpenClaw AI agent with Ollama local models, eliminating the need for cloud API keys. Built by forking EasyClaw and replacing cloud provider configuration with Ollama local model management. Users choose from three hardware presets (low/medium/high) to automatically pull and configure appropriate DeepSeek R1 models.

## Product Positioning

- **Target users**: People who want to run AI agents locally without cloud API costs or privacy concerns
- **Core value**: "Download → Choose spec → Run" — three steps to a fully local AI agent
- **Key differentiator from EasyClaw**: No API keys needed, everything runs locally via Ollama

## Tech Stack

Same as EasyClaw:

| Area | Technology |
|------|-----------|
| Framework | Electron + electron-vite |
| Frontend | React 19 + Tailwind CSS 4 |
| Language | TypeScript |
| Build | electron-builder |
| Auto-update | electron-updater |

## Wizard Flow

```
welcome → envCheck → (wslSetup) → (install) → ollamaSetup → config → done
```

Changes from EasyClaw:
- **Removed**: `apiKeyGuide` step (cloud API key configuration)
- **Removed**: `telegramGuide` step (Telegram channel setup)
- **Added**: `ollamaSetup` step (Ollama installation + model preset selection)

Conditional steps:
- `wslSetup`: Windows only, when WSL is not ready
- `install`: When Node.js or OpenClaw is not installed

## Three Hardware Presets

| Preset | Target Machine | Models | Disk Space | Intelligence Level |
|--------|---------------|--------|-----------|-------------------|
| Low (green) | 8GB RAM laptop | `deepseek-r1:8b` | ~5GB | Basic reasoning, usable for daily tasks |
| Medium (yellow) | 16GB RAM desktop | `deepseek-r1:14b` | ~10GB | Strong reasoning capability |
| High (red) | 32GB+ RAM workstation | `deepseek-r1:32b` | ~20GB | Near DeepSeek V4 level reasoning |

Advanced users on 64GB+ machines can manually select `deepseek-r1:70b` or `deepseek-v3` from a custom option.

## Architecture

### Project Structure (based on EasyClaw fork)

```
src/
├── main/                        # Main process (Node.js)
│   ├── services/
│   │   ├── ollama-manager.ts    # NEW: Ollama install, model pull, status
│   │   ├── env-checker.ts       # MODIFIED: Add Ollama detection
│   │   ├── installer.ts         # KEPT: Node.js/OpenClaw installation
│   │   ├── onboarder.ts         # MODIFIED: Ollama config replaces API key
│   │   ├── gateway.ts           # KEPT: Gateway start/stop/status
│   │   ├── wsl-utils.ts         # KEPT: WSL helpers (Windows)
│   │   ├── path-utils.ts        # KEPT: PATH expansion helpers
│   │   ├── tray-manager.ts      # KEPT: System tray + Gateway monitoring
│   │   ├── updater.ts           # KEPT: Auto-update via electron-updater
│   │   ├── troubleshooter.ts    # MODIFIED: Add Ollama diagnostics
│   │   ├── backup.ts            # KEPT: Config backup/restore
│   │   └── uninstaller.ts       # KEPT: OpenClaw uninstall
│   ├── ipc-handlers.ts          # MODIFIED: Add Ollama IPC channels
│   └── index.ts                 # MODIFIED: App lifecycle adjustments
├── preload/
│   ├── index.ts                 # MODIFIED: Add Ollama IPC API
│   └── index.d.ts               # MODIFIED: Add Ollama type declarations
├── renderer/
│   └── src/
│       ├── steps/
│       │   ├── WelcomeStep.tsx       # MODIFIED: Branding/copy
│       │   ├── EnvCheckStep.tsx      # MODIFIED: Add Ollama check item
│       │   ├── WslSetupStep.tsx      # KEPT
│       │   ├── InstallStep.tsx       # KEPT
│       │   ├── OllamaSetupStep.tsx   # NEW: Ollama install + 3-preset selection
│       │   ├── ConfigStep.tsx        # MODIFIED: Show Ollama config summary
│       │   └── DoneStep.tsx          # MODIFIED: Start Ollama + Gateway
│       ├── hooks/
│       │   └── useWizard.ts          # MODIFIED: Updated step flow
│       └── App.tsx                   # MODIFIED: Updated wizard state
└── shared/
    └── i18n/                         # MODIFIED: Updated translations
```

### Removed EasyClaw Modules

- `oauth.ts` — OpenAI OAuth flow (not needed for local models)
- API Key provider selection UI (anthropic/google/openai/minimax/glm/deepseek/ollama cloud)
- Telegram channel configuration wizard

### New Service: ollama-manager.ts

| Method | Description |
|--------|-------------|
| `checkOllamaInstalled()` | Detect Ollama binary on system PATH |
| `installOllama()` | Platform-specific: macOS (brew cask or .zip), Windows (.exe installer), Linux (curl script) |
| `startOllamaService()` | Start Ollama as background service (`ollama serve`) |
| `stopOllamaService()` | Stop Ollama service |
| `pullModel(name, onProgress?)` | Pull model with progress callback |
| `listModels()` | List installed Ollama models via `ollama list` |
| `getOllamaStatus()` | Check if Ollama service is running (GET http://localhost:11434/api/tags) |
| `getRecommendedPreset()` | Auto-detect system RAM and recommend a preset |
| `deleteModel(name)` | Remove an installed model |

### Ollama Installation Strategy

**macOS**:
1. Check if `ollama` is on PATH
2. If not, download from `https://ollama.com/download/Ollama-darwin.zip`
3. Extract to `/Applications/Ollama.app`
4. Add symlink to `/usr/local/bin/ollama`

**Windows (WSL)**:
1. Check if `ollama` is available in WSL
2. If not, run `curl -fsSL https://ollama.com/install.sh | sh` inside WSL
3. Verify installation

**Linux**:
1. Run `curl -fsSL https://ollama.com/install.sh | sh`
2. Verify installation

### OpenClaw Ollama Configuration

OpenClaw is compatible with OpenAI API. Ollama exposes a compatible endpoint at `http://localhost:11434/v1`. Configuration is written to `~/.openclaw/openclaw.json`:

```json
{
  "provider": "openai",
  "baseUrl": "http://localhost:11434/v1",
  "apiKey": "ollama",
  "model": "deepseek-r1:14b"
}
```

On Windows with WSL, the Ollama endpoint may be at `http://host.docker.internal:11434/v1` or `http://$(hostname).local:11434/v1` depending on WSL networking configuration.

### IPC Channels (New/Modified)

| Channel | Direction | Description |
|---------|-----------|-------------|
| `ollama:check` | renderer → main | Check if Ollama is installed |
| `ollama:install` | renderer → main | Install Ollama |
| `ollama:start` | renderer → main | Start Ollama service |
| `ollama:status` | renderer → main | Get Ollama service status |
| `ollama:pull-model` | renderer → main | Pull a specific model |
| `ollama:list-models` | renderer → main | List installed models |
| `ollama:delete-model` | renderer → main | Delete a model |
| `ollama:recommend-preset` | renderer → main | Auto-recommend preset based on system RAM |
| `ollama:progress` | main → renderer | Model pull progress |
| `ollama:error` | main → renderer | Ollama operation error |

### OllamaSetupStep UI Design

The step has two sub-phases:

**Phase 1: Ollama Installation**
- Auto-detect if Ollama is installed
- If not, show "Install Ollama" button with progress
- Once installed, verify service is running

**Phase 2: Model Preset Selection**
- Display 3 preset cards side by side:
  - 🟢 Low (8GB) — `deepseek-r1:8b` — ~5GB
  - 🟡 Medium (16GB) — `deepseek-r1:14b` — ~10GB
  - 🔴 High (32GB+) — `deepseek-r1:32b` — ~20GB
- Auto-highlight recommended preset based on detected system RAM
- "Custom" option for advanced users to specify any model name
- After selection, show pull progress with download speed and ETA
- Once complete, auto-configure openclaw

### System RAM Detection

Use `systeminformation` or Node.js `os.totalmem()` to detect total RAM:
- `< 12GB` → recommend Low
- `12-24GB` → recommend Medium
- `> 24GB` → recommend High

### App Lifecycle Changes

1. On app start: Check if Ollama service is running, start if not
2. Tray menu: Add "Ollama Status" indicator (running/stopped)
3. Done step: Start both Ollama service and OpenClaw Gateway
4. On app quit: Optionally stop Ollama service (user preference)

### Error Handling

| Scenario | Handling |
|----------|----------|
| Ollama install fails | Show error, offer retry or manual install link |
| Model pull fails (network) | Show error with retry button, resume partial download |
| Ollama service won't start | Check port 11434, offer troubleshooter |
| Insufficient RAM for selected preset | Show warning, suggest lower preset |
| Insufficient disk space | Show warning with required vs available space |
| WSL Ollama connection issues | Auto-detect WSL networking mode, adjust baseUrl |

### i18n

Support 4 languages same as EasyClaw: Korean (ko), English (en), Japanese (ja), Chinese (zh). Add Chinese as primary since target audience is Chinese-speaking users.

## Implementation Plan

### Phase 1: Fork and Setup
1. Fork EasyClaw repository
2. Remove cloud API key related code
3. Remove Telegram configuration code
4. Update branding and copy

### Phase 2: Ollama Manager Service
1. Implement `ollama-manager.ts` with all methods
2. Add Ollama IPC channels
3. Update preload bridge

### Phase 3: OllamaSetupStep UI
1. Build Ollama installation sub-phase
2. Build 3-preset selection UI
3. Build model pull progress UI
4. Implement auto RAM detection and preset recommendation

### Phase 4: Integration
1. Update wizard flow (useWizard hook)
2. Update onboarder to write Ollama config
3. Update env-checker to detect Ollama
4. Update DoneStep to start Ollama + Gateway
5. Update tray manager with Ollama status

### Phase 5: Polish
1. Update i18n translations
2. Update troubleshooter for Ollama diagnostics
3. Test on macOS and Windows (WSL)
4. Update electron-builder config for new app identity
