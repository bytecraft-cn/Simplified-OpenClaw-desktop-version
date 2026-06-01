# OpenClaw Desktop (Ollama Edition) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a desktop installer that sets up OpenClaw AI agent with Ollama local models, replacing EasyClaw's cloud API key configuration with three hardware presets (low/medium/high) for DeepSeek R1 models.

**Architecture:** Fork EasyClaw, strip cloud API and Telegram code, add Ollama manager service and OllamaSetupStep wizard page. The app follows Electron's 3-layer pattern: main process (Node.js services), preload (IPC bridge), renderer (React UI).

**Tech Stack:** Electron + electron-vite + React 19 + Tailwind CSS 4 + TypeScript

---

## File Structure

### New Files
- `src/main/services/ollama-manager.ts` — Ollama install, model pull, service management
- `src/renderer/src/steps/OllamaSetupStep.tsx` — Ollama install + 3-preset selection UI
- `src/renderer/src/components/PresetCard.tsx` — Reusable preset card component
- `src/renderer/src/components/ModelPullProgress.tsx` — Model download progress bar
- `src/shared/ollama-presets.ts` — Preset definitions and RAM detection logic

### Modified Files
- `src/main/ipc-handlers.ts` — Add Ollama IPC channels
- `src/main/index.ts` — Add Ollama lifecycle management
- `src/main/services/env-checker.ts` — Add Ollama detection
- `src/main/services/onboarder.ts` — Replace API key config with Ollama config
- `src/main/services/troubleshooter.ts` — Add Ollama diagnostics
- `src/main/services/tray-manager.ts` — Add Ollama status indicator
- `src/preload/index.ts` — Add Ollama IPC API
- `src/preload/index.d.ts` — Add Ollama type declarations
- `src/renderer/src/App.tsx` — Update wizard state
- `src/renderer/src/hooks/useWizard.ts` — Update step flow
- `src/renderer/src/steps/WelcomeStep.tsx` — Update branding
- `src/renderer/src/steps/EnvCheckStep.tsx` — Add Ollama check
- `src/renderer/src/steps/ConfigStep.tsx` — Show Ollama config
- `src/renderer/src/steps/DoneStep.tsx` — Start Ollama + Gateway
- `package.json` — Update name, description, version
- `electron-builder.yml` — Update app identity

### Deleted Files
- `src/main/services/oauth.ts` — OpenAI OAuth (not needed)
- `src/renderer/src/steps/ApiKeyGuideStep.tsx` — Cloud API key UI
- `src/renderer/src/steps/TelegramGuideStep.tsx` — Telegram setup UI

---

### Task 1: Clone EasyClaw and Initialize Project

**Files:**
- Create: `openclaw-desktop/` (entire project)

- [ ] **Step 1: Clone EasyClaw repository**

```bash
cd /workspace
git clone https://github.com/ybgwon96/easyclaw.git openclaw-desktop
cd openclaw-desktop
```

- [ ] **Step 2: Install dependencies**

```bash
cd /workspace/openclaw-desktop
npm install
```

- [ ] **Step 3: Update package.json identity**

Replace `name`, `description`, `author`, `homepage`, and `keywords` in `package.json`:

```json
{
  "name": "openclaw-desktop",
  "version": "0.1.0",
  "description": "One-click installer for OpenClaw AI agent with Ollama local models",
  "author": "OpenClaw",
  "homepage": "https://github.com/openclaw/openclaw",
  "keywords": [
    "openclaw",
    "ollama",
    "ai-agent",
    "local-ai",
    "deepseek",
    "electron",
    "desktop-app"
  ]
}
```

- [ ] **Step 4: Update electron-builder.yml app identity**

Change `appId` and `productName`:

```yaml
appId: com.openclaw.desktop
productName: OpenClaw Desktop
```

- [ ] **Step 5: Verify dev mode works**

```bash
cd /workspace/openclaw-desktop
npm run dev
```

Expected: Electron window opens with EasyClaw wizard UI.

- [ ] **Step 6: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "chore: initialize openclaw-desktop from easyclaw fork"
```

---

### Task 2: Remove Cloud API Key and Telegram Code

**Files:**
- Delete: `src/main/services/oauth.ts`
- Delete: `src/renderer/src/steps/ApiKeyGuideStep.tsx`
- Delete: `src/renderer/src/steps/TelegramGuideStep.tsx`
- Modify: `src/main/ipc-handlers.ts`
- Modify: `src/main/services/onboarder.ts`
- Modify: `src/renderer/src/hooks/useWizard.ts`
- Modify: `src/renderer/src/App.tsx`
- Modify: `src/preload/index.ts`
- Modify: `src/preload/index.d.ts`

- [ ] **Step 1: Delete oauth.ts**

```bash
rm src/main/services/oauth.ts
```

- [ ] **Step 2: Delete ApiKeyGuideStep.tsx**

```bash
rm src/renderer/src/steps/ApiKeyGuideStep.tsx
```

- [ ] **Step 3: Delete TelegramGuideStep.tsx**

```bash
rm src/renderer/src/steps/TelegramGuideStep.tsx
```

- [ ] **Step 4: Remove oauth imports and handlers from ipc-handlers.ts**

Open `src/main/ipc-handlers.ts` and remove all lines referencing `oauth`, `oauth:`, or the OAuth service. Remove the import statement for `./services/oauth`.

- [ ] **Step 5: Remove API key and Telegram logic from onboarder.ts**

Open `src/main/services/onboarder.ts` and remove:
- All functions related to API key configuration (e.g., `setApiKey`, `configureProvider`)
- All functions related to Telegram channel setup (e.g., `addTelegramChannel`)
- Keep the function signatures that will be reused for Ollama configuration but clear their bodies (add `// TODO: implement for Ollama` comment)

- [ ] **Step 6: Remove apiKeyGuide and telegramGuide from wizard steps**

Open `src/renderer/src/hooks/useWizard.ts` and remove `apiKeyGuide` and `telegramGuide` from the STEPS array. The flow should now be:

```typescript
const STEPS = ['welcome', 'envCheck', 'install', 'config', 'done'] as const
```

- [ ] **Step 7: Remove API key and Telegram state from App.tsx**

Open `src/renderer/src/App.tsx` and remove:
- State variables for `apiKey`, `provider`, `telegramBotToken`, `telegramChatId`
- Any props passed to removed step components
- Import statements for deleted step components

- [ ] **Step 8: Remove OAuth and API key IPC from preload**

Open `src/preload/index.ts` and remove:
- `oauth:*` IPC handlers from the `electronAPI` object
- `setApiKey`, `configureProvider`, `addTelegramChannel` methods

Open `src/preload/index.d.ts` and remove corresponding type declarations.

- [ ] **Step 9: Verify the app still compiles**

```bash
cd /workspace/openclaw-desktop
npm run typecheck
```

Expected: Type errors only in files that reference removed code. Fix any remaining references.

- [ ] **Step 10: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "refactor: remove cloud API key and Telegram configuration code"
```

---

### Task 3: Create Ollama Preset Definitions

**Files:**
- Create: `src/shared/ollama-presets.ts`

- [ ] **Step 1: Create the preset definitions module**

Create `src/shared/ollama-presets.ts`:

```typescript
export interface OllamaPreset {
  id: 'low' | 'medium' | 'high' | 'custom'
  label: string
  labelZh: string
  description: string
  descriptionZh: string
  model: string
  diskSpace: string
  minRamGB: number
  color: string
  icon: string
}

export const OLLAMA_PRESETS: OllamaPreset[] = [
  {
    id: 'low',
    label: 'Low',
    labelZh: '低配',
    description: 'For laptops with 8GB RAM',
    descriptionZh: '适合 8GB 内存笔记本电脑',
    model: 'deepseek-r1:8b',
    diskSpace: '~5GB',
    minRamGB: 8,
    color: '#22c55e',
    icon: '🟢'
  },
  {
    id: 'medium',
    label: 'Medium',
    labelZh: '中配',
    description: 'For desktops with 16GB RAM',
    descriptionZh: '适合 16GB 内存台式机',
    model: 'deepseek-r1:14b',
    diskSpace: '~10GB',
    minRamGB: 16,
    color: '#eab308',
    icon: '🟡'
  },
  {
    id: 'high',
    label: 'High',
    labelZh: '高配',
    description: 'For workstations with 32GB+ RAM',
    descriptionZh: '适合 32GB+ 内存工作站',
    model: 'deepseek-r1:32b',
    diskSpace: '~20GB',
    minRamGB: 32,
    color: '#ef4444',
    icon: '🔴'
  }
]

export function getRecommendedPreset(totalRamBytes: number): OllamaPreset {
  const ramGB = totalRamBytes / (1024 * 1024 * 1024)
  if (ramGB < 12) return OLLAMA_PRESETS[0]
  if (ramGB < 24) return OLLAMA_PRESETS[1]
  return OLLAMA_PRESETS[2]
}
```

- [ ] **Step 2: Verify types compile**

```bash
cd /workspace/openclaw-desktop
npx tsc --noEmit src/shared/ollama-presets.ts
```

Expected: No errors.

- [ ] **Step 3: Commit**

```bash
cd /workspace/openclaw-desktop
git add src/shared/ollama-presets.ts
git commit -m "feat: add Ollama preset definitions with RAM-based recommendation"
```

---

### Task 4: Create Ollama Manager Service

**Files:**
- Create: `src/main/services/ollama-manager.ts`

- [ ] **Step 1: Create ollama-manager.ts with core methods**

Create `src/main/services/ollama-manager.ts`:

```typescript
import { execFile } from 'node:child_process'
import { promisify } from 'node:util'
import { app } from 'electron'
import { createReadStream } from 'node:fs'
import { createWriteStream, mkdirSync, existsSync } from 'node:fs'
import { join } from 'node:path'
import { platform } from 'node:os'
import { pipeline } from 'node:stream/promises'
import { get } from 'node:https'
import http from 'node:http'
import { OLLAMA_PRESETS, getRecommendedPreset } from '../../shared/ollama-presets'

const execFileAsync = promisify(execFile)

const OLLAMA_API_BASE = 'http://localhost:11434'

export interface OllamaStatus {
  installed: boolean
  running: boolean
  models: string[]
  version: string | null
}

export interface PullProgress {
  status: string
  digest: string
  total: number
  completed: number
  percent: number
}

async function runCommand(
  cmd: string,
  args: string[],
  options?: { cwd?: string; env?: Record<string, string> }
): Promise<{ stdout: string; stderr: string }> {
  return execFileAsync(cmd, args, {
    timeout: 300000,
    ...options
  })
}

function getOllamaBinary(): string {
  const p = platform()
  if (p === 'win32') return 'ollama.exe'
  return 'ollama'
}

export async function checkOllamaInstalled(): Promise<boolean> {
  try {
    const { stdout } = await runCommand(getOllamaBinary(), ['--version'])
    return stdout.includes('ollama')
  } catch {
    return false
  }
}

export async function getOllamaVersion(): Promise<string | null> {
  try {
    const { stdout } = await runCommand(getOllamaBinary(), ['--version'])
    const match = stdout.match(/(\d+\.\d+\.\d+)/)
    return match ? match[1] : null
  } catch {
    return null
  }
}

export async function installOllama(): Promise<void> {
  const p = platform()
  if (p === 'darwin') {
    const zipUrl = 'https://ollama.com/download/Ollama-darwin.zip'
    const tmpDir = app.getPath('temp')
    const zipPath = join(tmpDir, 'Ollama-darwin.zip')
    await downloadFile(zipUrl, zipPath)
    await runCommand('unzip', ['-o', zipPath, '-d', '/Applications/'])
  } else if (p === 'linux') {
    const scriptUrl = 'https://ollama.com/install.sh'
    const tmpDir = app.getPath('temp')
    const scriptPath = join(tmpDir, 'ollama-install.sh')
    await downloadFile(scriptUrl, scriptPath)
    await runCommand('sh', [scriptPath])
  } else {
    const exeUrl = 'https://ollama.com/download/OllamaSetup.exe'
    const tmpDir = app.getPath('temp')
    const exePath = join(tmpDir, 'OllamaSetup.exe')
    await downloadFile(exeUrl, exePath)
    await runCommand(exePath, ['/S'])
  }
}

export async function startOllamaService(): Promise<void> {
  try {
    const status = await getOllamaStatus()
    if (status.running) return
  } catch {
    // not running, proceed to start
  }

  const binary = getOllamaBinary()
  const child = require('child_process').spawn(binary, ['serve'], {
    detached: true,
    stdio: 'ignore'
  })
  child.unref()

  await waitForOllamaReady(30000)
}

export async function stopOllamaService(): Promise<void> {
  try {
    await runCommand(getOllamaBinary(), ['stop'])
  } catch {
    // ignore errors on stop
  }
}

export async function getOllamaStatus(): Promise<OllamaStatus> {
  const installed = await checkOllamaInstalled()
  let running = false
  let models: string[] = []
  const version = await getOllamaVersion()

  if (installed) {
    try {
      const response = await fetchWithTimeout(`${OLLAMA_API_BASE}/api/tags`, 3000)
      if (response.ok) {
        running = true
        const data = await response.json()
        models = (data.models || []).map((m: { name: string }) => m.name)
      }
    } catch {
      running = false
    }
  }

  return { installed, running, models, version }
}

export async function pullModel(
  modelName: string,
  onProgress?: (progress: PullProgress) => void
): Promise<void> {
  const response = await fetch(`${OLLAMA_API_BASE}/api/pull`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name: modelName, stream: true })
  })

  if (!response.ok) {
    throw new Error(`Failed to pull model: ${response.statusText}`)
  }

  const reader = response.body?.getReader()
  if (!reader) throw new Error('No response body')

  const decoder = new TextDecoder()
  let buffer = ''

  while (true) {
    const { done, value } = await reader.read()
    if (done) break

    buffer += decoder.decode(value, { stream: true })
    const lines = buffer.split('\n')
    buffer = lines.pop() || ''

    for (const line of lines) {
      if (!line.trim()) continue
      try {
        const data = JSON.parse(line)
        const progress: PullProgress = {
          status: data.status || '',
          digest: data.digest || '',
          total: data.total || 0,
          completed: data.completed || 0,
          percent: data.total > 0 ? Math.round((data.completed / data.total) * 100) : 0
        }
        onProgress?.(progress)
      } catch {
        // skip malformed lines
      }
    }
  }
}

export async function listModels(): Promise<string[]> {
  try {
    const response = await fetchWithTimeout(`${OLLAMA_API_BASE}/api/tags`, 5000)
    if (!response.ok) return []
    const data = await response.json()
    return (data.models || []).map((m: { name: string }) => m.name)
  } catch {
    return []
  }
}

export async function deleteModel(modelName: string): Promise<void> {
  await runCommand(getOllamaBinary(), ['rm', modelName])
}

export async function getRecommendedPresetForSystem(): Promise<{
  preset: (typeof OLLAMA_PRESETS)[number]
  totalRamGB: number
}> {
  const os = await import('node:os')
  const totalRam = os.totalmem()
  const totalRamGB = Math.round(totalRam / (1024 * 1024 * 1024))
  return {
    preset: getRecommendedPreset(totalRam),
    totalRamGB
  }
}

export async function configureOpenClawForOllama(modelName: string): Promise<void> {
  const os = await import('node:os')
  const p = platform()
  let configDir: string
  let baseUrl: string

  if (p === 'win32') {
    configDir = join(os.homedir(), '.openclaw')
    baseUrl = 'http://localhost:11434/v1'
  } else {
    configDir = join(os.homedir(), '.openclaw')
    baseUrl = 'http://localhost:11434/v1'
  }

  const configPath = join(configDir, 'openclaw.json')
  mkdirSync(configDir, { recursive: true })

  let existingConfig: Record<string, unknown> = {}
  if (existsSync(configPath)) {
    const content = await import('node:fs/promises').then((fs) => fs.readFile(configPath, 'utf-8'))
    try {
      existingConfig = JSON.parse(content)
    } catch {
      existingConfig = {}
    }
  }

  const newConfig = {
    ...existingConfig,
    provider: 'openai',
    baseUrl,
    apiKey: 'ollama',
    model: modelName
  }

  const { writeFile } = await import('node:fs/promises')
  await writeFile(configPath, JSON.stringify(newConfig, null, 2), 'utf-8')
}

async function downloadFile(url: string, destPath: string): Promise<void> {
  const dir = join(destPath, '..')
  mkdirSync(dir, { recursive: true })

  return new Promise((resolve, reject) => {
    const protocol = url.startsWith('https') ? get : http.get
    protocol(url, (response) => {
      if (response.statusCode === 301 || response.statusCode === 302) {
        const redirectUrl = response.headers.location
        if (redirectUrl) {
          downloadFile(redirectUrl, destPath).then(resolve).catch(reject)
          return
        }
      }
      if (response.statusCode !== 200) {
        reject(new Error(`Download failed: ${response.statusCode}`))
        return
      }
      const file = createWriteStream(destPath)
      response.pipe(file)
      file.on('finish', () => {
        file.close()
        resolve()
      })
      file.on('error', reject)
    }).on('error', reject)
  })
}

async function fetchWithTimeout(url: string, timeoutMs: number): Promise<Response> {
  const controller = new AbortController()
  const timer = setTimeout(() => controller.abort(), timeoutMs)
  try {
    const response = await fetch(url, { signal: controller.signal })
    return response
  } finally {
    clearTimeout(timer)
  }
}

async function waitForOllamaReady(timeoutMs: number): Promise<void> {
  const start = Date.now()
  while (Date.now() - start < timeoutMs) {
    try {
      const response = await fetchWithTimeout(`${OLLAMA_API_BASE}/api/tags`, 2000)
      if (response.ok) return
    } catch {
      // not ready yet
    }
    await new Promise((r) => setTimeout(r, 1000))
  }
  throw new Error('Ollama service did not become ready in time')
}
```

- [ ] **Step 2: Verify the service compiles**

```bash
cd /workspace/openclaw-desktop
npx tsc --noEmit --skipLibCheck src/main/services/ollama-manager.ts
```

Expected: No errors.

- [ ] **Step 3: Commit**

```bash
cd /workspace/openclaw-desktop
git add src/main/services/ollama-manager.ts
git commit -m "feat: add Ollama manager service with install, pull, and config methods"
```

---

### Task 5: Add Ollama IPC Channels

**Files:**
- Modify: `src/main/ipc-handlers.ts`
- Modify: `src/preload/index.ts`
- Modify: `src/preload/index.d.ts`

- [ ] **Step 1: Add Ollama IPC handlers to ipc-handlers.ts**

Add the following imports and handlers to `src/main/ipc-handlers.ts`:

```typescript
import {
  checkOllamaInstalled,
  installOllama,
  startOllamaService,
  stopOllamaService,
  getOllamaStatus,
  pullModel,
  listModels,
  deleteModel,
  getRecommendedPresetForSystem,
  configureOpenClawForOllama
} from './services/ollama-manager'
```

Add these handler registrations:

```typescript
ipcMain.handle('ollama:check', async () => {
  return checkOllamaInstalled()
})

ipcMain.handle('ollama:install', async () => {
  await installOllama()
  return true
})

ipcMain.handle('ollama:start', async () => {
  await startOllamaService()
  return true
})

ipcMain.handle('ollama:stop', async () => {
  await stopOllamaService()
  return true
})

ipcMain.handle('ollama:status', async () => {
  return getOllamaStatus()
})

ipcMain.handle('ollama:pull-model', async (_event, modelName: string) => {
  await pullModel(modelName, (progress) => {
    _event.sender.send('ollama:progress', progress)
  })
  return true
})

ipcMain.handle('ollama:list-models', async () => {
  return listModels()
})

ipcMain.handle('ollama:delete-model', async (_event, modelName: string) => {
  await deleteModel(modelName)
  return true
})

ipcMain.handle('ollama:recommend-preset', async () => {
  return getRecommendedPresetForSystem()
})

ipcMain.handle('ollama:configure', async (_event, modelName: string) => {
  await configureOpenClawForOllama(modelName)
  return true
})
```

- [ ] **Step 2: Add Ollama methods to preload/index.ts**

Add to the `electronAPI` object in `src/preload/index.ts`:

```typescript
ollama: {
  check: () => ipcRenderer.invoke('ollama:check'),
  install: () => ipcRenderer.invoke('ollama:install'),
  start: () => ipcRenderer.invoke('ollama:start'),
  stop: () => ipcRenderer.invoke('ollama:stop'),
  status: () => ipcRenderer.invoke('ollama:status'),
  pullModel: (modelName: string) => ipcRenderer.invoke('ollama:pull-model', modelName),
  listModels: () => ipcRenderer.invoke('ollama:list-models'),
  deleteModel: (modelName: string) => ipcRenderer.invoke('ollama:delete-model', modelName),
  recommendPreset: () => ipcRenderer.invoke('ollama:recommend-preset'),
  configure: (modelName: string) => ipcRenderer.invoke('ollama:configure', modelName),
  onProgress: (callback: (progress: PullProgress) => void) => {
    ipcRenderer.on('ollama:progress', (_event, progress) => callback(progress))
    return () => ipcRenderer.removeListener('ollama:progress', callback)
  },
  onError: (callback: (error: string) => void) => {
    ipcRenderer.on('ollama:error', (_event, error) => callback(error))
    return () => ipcRenderer.removeListener('ollama:error', callback)
  }
}
```

Import `PullProgress` type from shared:

```typescript
import type { PullProgress } from '../shared/ollama-presets'
```

- [ ] **Step 3: Add Ollama type declarations to preload/index.d.ts**

Add to the `ElectronAPI` interface in `src/preload/index.d.ts`:

```typescript
ollama: {
  check: () => Promise<boolean>
  install: () => Promise<boolean>
  start: () => Promise<boolean>
  stop: () => Promise<boolean>
  status: () => Promise<{
    installed: boolean
    running: boolean
    models: string[]
    version: string | null
  }>
  pullModel: (modelName: string) => Promise<boolean>
  listModels: () => Promise<string[]>
  deleteModel: (modelName: string) => Promise<void>
  recommendPreset: () => Promise<{
    preset: {
      id: string
      label: string
      labelZh: string
      description: string
      descriptionZh: string
      model: string
      diskSpace: string
      minRamGB: number
      color: string
      icon: string
    }
    totalRamGB: number
  }>
  configure: (modelName: string) => Promise<boolean>
  onProgress: (callback: (progress: {
    status: string
    digest: string
    total: number
    completed: number
    percent: number
  }) => void) => () => void
  onError: (callback: (error: string) => void) => () => void
}
```

- [ ] **Step 4: Verify typecheck passes**

```bash
cd /workspace/openclaw-desktop
npm run typecheck
```

Expected: No errors related to Ollama IPC.

- [ ] **Step 5: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "feat: add Ollama IPC channels and preload bridge"
```

---

### Task 6: Update Env Checker for Ollama Detection

**Files:**
- Modify: `src/main/services/env-checker.ts`
- Modify: `src/renderer/src/steps/EnvCheckStep.tsx`

- [ ] **Step 1: Add Ollama check to env-checker.ts**

Open `src/main/services/env-checker.ts` and add an `ollama` field to the environment check result. Add a function:

```typescript
export async function checkOllama(): Promise<{
  installed: boolean
  version: string | null
  running: boolean
}> {
  const installed = await checkOllamaInstalled()
  const version = installed ? await getOllamaVersion() : null
  let running = false
  if (installed) {
    try {
      const status = await getOllamaStatus()
      running = status.running
    } catch {
      running = false
    }
  }
  return { installed, version, running }
}
```

Import `checkOllamaInstalled`, `getOllamaVersion`, `getOllamaStatus` from `./ollama-manager`.

Add `ollama` to the `EnvCheckResult` type:

```typescript
ollama: {
  installed: boolean
  version: string | null
  running: boolean
}
```

Include the Ollama check in the main `checkEnvironment()` function.

- [ ] **Step 2: Update EnvCheckStep.tsx to show Ollama status**

Open `src/renderer/src/steps/EnvCheckStep.tsx` and add a row for Ollama in the environment check display:

```tsx
<div className="flex items-center gap-2">
  <span>{envCheck.ollama?.installed ? '✅' : '❌'}</span>
  <span>Ollama</span>
  {envCheck.ollama?.installed && envCheck.ollama.version && (
    <span className="text-text-muted">v{envCheck.ollama.version}</span>
  )}
  {envCheck.ollama?.installed && (
    <span className="text-text-muted">
      {envCheck.ollama.running ? '(running)' : '(stopped)'}
    </span>
  )}
</div>
```

- [ ] **Step 3: Verify typecheck passes**

```bash
cd /workspace/openclaw-desktop
npm run typecheck
```

- [ ] **Step 4: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "feat: add Ollama detection to environment checker"
```

---

### Task 7: Create OllamaSetupStep UI

**Files:**
- Create: `src/renderer/src/components/PresetCard.tsx`
- Create: `src/renderer/src/components/ModelPullProgress.tsx`
- Create: `src/renderer/src/steps/OllamaSetupStep.tsx`

- [ ] **Step 1: Create PresetCard component**

Create `src/renderer/src/components/PresetCard.tsx`:

```tsx
import type { OllamaPreset } from '../../../shared/ollama-presets'

interface PresetCardProps {
  preset: OllamaPreset
  recommended: boolean
  selected: boolean
  onSelect: () => void
}

export function PresetCard({ preset, recommended, selected, onSelect }: PresetCardProps) {
  return (
    <button
      type="button"
      onClick={onSelect}
      className={`relative flex flex-col items-center gap-3 rounded-2xl border-2 p-6 transition-all hover:scale-[1.02] ${
        selected
          ? 'border-primary bg-primary/10 shadow-lg shadow-primary/20'
          : 'border-border-default bg-bg-card hover:border-primary/50'
      }`}
    >
      {recommended && (
        <span className="absolute -top-3 left-1/2 -translate-x-1/2 rounded-full bg-primary px-3 py-0.5 text-xs font-semibold text-white">
          Recommended
        </span>
      )}
      <span className="text-3xl">{preset.icon}</span>
      <h3 className="text-lg font-bold text-text-primary">{preset.labelZh}</h3>
      <p className="text-sm text-text-muted">{preset.descriptionZh}</p>
      <div className="mt-2 space-y-1 text-center">
        <p className="text-sm font-medium text-text-primary">{preset.model}</p>
        <p className="text-xs text-text-muted">Disk: {preset.diskSpace}</p>
        <p className="text-xs text-text-muted">Min RAM: {preset.minRamGB}GB</p>
      </div>
    </button>
  )
}
```

- [ ] **Step 2: Create ModelPullProgress component**

Create `src/renderer/src/components/ModelPullProgress.tsx`:

```tsx
interface ModelPullProgressProps {
  modelName: string
  percent: number
  status: string
  completed: number
  total: number
}

function formatBytes(bytes: number): string {
  if (bytes === 0) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return `${parseFloat((bytes / Math.pow(k, i)).toFixed(1))} ${sizes[i]}`
}

export function ModelPullProgress({
  modelName,
  percent,
  status,
  completed,
  total
}: ModelPullProgressProps) {
  return (
    <div className="space-y-3">
      <div className="flex items-center justify-between">
        <span className="text-sm font-medium text-text-primary">Pulling {modelName}</span>
        <span className="text-sm text-text-muted">{percent}%</span>
      </div>
      <div className="h-3 overflow-hidden rounded-full bg-bg-card">
        <div
          className="h-full rounded-full bg-primary transition-all duration-300"
          style={{ width: `${Math.max(percent, 2)}%` }}
        />
      </div>
      <div className="flex items-center justify-between text-xs text-text-muted">
        <span>{status}</span>
        <span>
          {formatBytes(completed)} / {formatBytes(total)}
        </span>
      </div>
    </div>
  )
}
```

- [ ] **Step 3: Create OllamaSetupStep component**

Create `src/renderer/src/steps/OllamaSetupStep.tsx`:

```tsx
import { useState, useEffect } from 'react'
import { PresetCard } from '../components/PresetCard'
import { ModelPullProgress } from '../components/ModelPullProgress'
import { OLLAMA_PRESETS } from '../../../shared/ollama-presets'
import type { OllamaPreset } from '../../../shared/ollama-presets'

interface OllamaSetupStepProps {
  onNext: () => void
}

type Phase = 'install' | 'select' | 'pulling' | 'done'

export function OllamaSetupStep({ onNext }: OllamaSetupStepProps) {
  const [phase, setPhase] = useState<Phase>('install')
  const [ollamaInstalled, setOllamaInstalled] = useState(false)
  const [installing, setInstalling] = useState(false)
  const [installError, setInstallError] = useState<string | null>(null)
  const [selectedPreset, setSelectedPreset] = useState<OllamaPreset | null>(null)
  const [recommendedId, setRecommendedId] = useState<string>('medium')
  const [totalRamGB, setTotalRamGB] = useState(16)
  const [pullProgress, setPullProgress] = useState({
    percent: 0,
    status: '',
    completed: 0,
    total: 0
  })
  const [customModel, setCustomModel] = useState('')
  const [showCustom, setShowCustom] = useState(false)

  useEffect(() => {
    checkAndRecommend()
  }, [])

  async function checkAndRecommend() {
    try {
      const installed = await window.electronAPI.ollama.check()
      if (installed) {
        setOllamaInstalled(true)
        setPhase('select')
      }
      const rec = await window.electronAPI.ollama.recommendPreset()
      setRecommendedId(rec.preset.id)
      setTotalRamGB(rec.totalRamGB)
    } catch {
      // defaults are fine
    }
  }

  async function handleInstallOllama() {
    setInstalling(true)
    setInstallError(null)
    try {
      await window.electronAPI.ollama.install()
      await window.electronAPI.ollama.start()
      setOllamaInstalled(true)
      setPhase('select')
    } catch (err) {
      setInstallError(err instanceof Error ? err.message : 'Failed to install Ollama')
    } finally {
      setInstalling(false)
    }
  }

  async function handleSelectPreset(preset: OllamaPreset) {
    setSelectedPreset(preset)
    setShowCustom(false)
  }

  async function handlePullModel() {
    const modelName = showCustom ? customModel : selectedPreset?.model
    if (!modelName) return

    setPhase('pulling')
    setPullProgress({ percent: 0, status: 'Starting...', completed: 0, total: 0 })

    const unsubscribe = window.electronAPI.ollama.onProgress((progress) => {
      setPullProgress({
        percent: progress.percent,
        status: progress.status,
        completed: progress.completed,
        total: progress.total
      })
    })

    try {
      await window.electronAPI.ollama.pullModel(modelName)
      await window.electronAPI.ollama.configure(modelName)
      setPhase('done')
    } catch (err) {
      setPhase('select')
      setInstallError(err instanceof Error ? err.message : 'Failed to pull model')
    } finally {
      unsubscribe()
    }
  }

  if (phase === 'install') {
    return (
      <div className="flex flex-col items-center gap-6">
        <h2 className="text-2xl font-bold text-text-primary">Install Ollama</h2>
        <p className="text-text-muted">
          Ollama is required to run AI models locally on your machine.
        </p>
        {installError && (
          <div className="rounded-lg bg-red-500/10 p-4 text-sm text-red-400">{installError}</div>
        )}
        <button
          type="button"
          onClick={handleInstallOllama}
          disabled={installing}
          className="rounded-xl bg-primary px-8 py-3 font-semibold text-white transition hover:bg-primary/90 disabled:opacity-50"
        >
          {installing ? 'Installing...' : 'Install Ollama'}
        </button>
        <p className="text-xs text-text-muted">
          Already installed?{' '}
          <button
            type="button"
            onClick={async () => {
              await window.electronAPI.ollama.start()
              setOllamaInstalled(true)
              setPhase('select')
            }}
            className="text-primary underline"
          >
            Start Ollama
          </button>
        </p>
      </div>
    )
  }

  if (phase === 'select') {
    return (
      <div className="flex flex-col items-center gap-6">
        <h2 className="text-2xl font-bold text-text-primary">Choose Model Preset</h2>
        <p className="text-text-muted">
          Your system has {totalRamGB}GB RAM. Select a model preset:
        </p>
        <div className="grid grid-cols-3 gap-4">
          {OLLAMA_PRESETS.map((preset) => (
            <PresetCard
              key={preset.id}
              preset={preset}
              recommended={preset.id === recommendedId}
              selected={selectedPreset?.id === preset.id}
              onSelect={() => handleSelectPreset(preset)}
            />
          ))}
        </div>
        <button
          type="button"
          onClick={() => setShowCustom(!showCustom)}
          className="text-sm text-primary underline"
        >
          {showCustom ? 'Hide custom model' : 'Use custom model name'}
        </button>
        {showCustom && (
          <input
            type="text"
            value={customModel}
            onChange={(e) => setCustomModel(e.target.value)}
            placeholder="e.g. deepseek-r1:70b"
            className="w-full max-w-md rounded-lg border border-border-default bg-bg-card px-4 py-2 text-text-primary placeholder:text-text-muted"
          />
        )}
        {installError && (
          <div className="rounded-lg bg-red-500/10 p-4 text-sm text-red-400">{installError}</div>
        )}
        <button
          type="button"
          onClick={handlePullModel}
          disabled={!selectedPreset && !customModel}
          className="rounded-xl bg-primary px-8 py-3 font-semibold text-white transition hover:bg-primary/90 disabled:opacity-50"
        >
          Pull Model
        </button>
      </div>
    )
  }

  if (phase === 'pulling') {
    const modelName = showCustom ? customModel : selectedPreset?.model || ''
    return (
      <div className="flex flex-col items-center gap-6">
        <h2 className="text-2xl font-bold text-text-primary">Downloading Model</h2>
        <div className="w-full max-w-lg">
          <ModelPullProgress
            modelName={modelName}
            percent={pullProgress.percent}
            status={pullProgress.status}
            completed={pullProgress.completed}
            total={pullProgress.total}
          />
        </div>
      </div>
    )
  }

  return (
    <div className="flex flex-col items-center gap-6">
      <h2 className="text-2xl font-bold text-text-primary">Model Ready!</h2>
      <p className="text-text-muted">
        {showCustom ? customModel : selectedPreset?.model} has been downloaded and configured.
      </p>
      <button
        type="button"
        onClick={onNext}
        className="rounded-xl bg-primary px-8 py-3 font-semibold text-white transition hover:bg-primary/90"
      >
        Continue
      </button>
    </div>
  )
}
```

- [ ] **Step 4: Verify typecheck passes**

```bash
cd /workspace/openclaw-desktop
npm run typecheck
```

- [ ] **Step 5: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "feat: add OllamaSetupStep with preset selection and model pull UI"
```

---

### Task 8: Update Wizard Flow and Integration

**Files:**
- Modify: `src/renderer/src/hooks/useWizard.ts`
- Modify: `src/renderer/src/App.tsx`
- Modify: `src/renderer/src/steps/WelcomeStep.tsx`
- Modify: `src/renderer/src/steps/DoneStep.tsx`
- Modify: `src/renderer/src/steps/ConfigStep.tsx`

- [ ] **Step 1: Update useWizard.ts step flow**

Open `src/renderer/src/hooks/useWizard.ts` and update the STEPS array to include `ollamaSetup`:

```typescript
const STEPS = ['welcome', 'envCheck', 'install', 'ollamaSetup', 'config', 'done'] as const
```

- [ ] **Step 2: Update App.tsx to include OllamaSetupStep**

Open `src/renderer/src/App.tsx` and:
- Import `OllamaSetupStep` from `./steps/OllamaSetupStep`
- Add `ollamaSetup` case to the step rendering switch:

```tsx
case 'ollamaSetup':
  return <OllamaSetupStep onNext={goNext} />
```

- Remove any references to `ApiKeyGuideStep` and `TelegramGuideStep`

- [ ] **Step 3: Update WelcomeStep.tsx branding**

Open `src/renderer/src/steps/WelcomeStep.tsx` and update:
- Title: "OpenClaw Desktop"
- Subtitle: "Run AI agents locally with Ollama — no cloud API keys needed"
- Description: Emphasize local AI, privacy, and zero cost

- [ ] **Step 4: Update DoneStep.tsx to start Ollama + Gateway**

Open `src/renderer/src/steps/DoneStep.tsx` and add Ollama service start alongside Gateway start:

```tsx
async function handleStart() {
  try {
    await window.electronAPI.ollama.start()
    await window.electronAPI.gateway.start()
  } catch (err) {
    // handle error
  }
}
```

Add Ollama status display alongside Gateway status in the done screen.

- [ ] **Step 5: Update ConfigStep.tsx to show Ollama config**

Open `src/renderer/src/steps/ConfigStep.tsx` and replace API key display with Ollama model configuration summary:

```tsx
<div className="space-y-2">
  <h3 className="font-semibold text-text-primary">Ollama Configuration</h3>
  <p className="text-sm text-text-muted">Provider: Ollama (local)</p>
  <p className="text-sm text-text-muted">Endpoint: http://localhost:11434/v1</p>
  <p className="text-sm text-text-muted">Model: {selectedModel}</p>
</div>
```

- [ ] **Step 6: Verify typecheck passes**

```bash
cd /workspace/openclaw-desktop
npm run typecheck
```

- [ ] **Step 7: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "feat: integrate OllamaSetupStep into wizard flow"
```

---

### Task 9: Update Main Process Lifecycle

**Files:**
- Modify: `src/main/index.ts`
- Modify: `src/main/services/tray-manager.ts`
- Modify: `src/main/services/onboarder.ts`

- [ ] **Step 1: Add Ollama lifecycle to index.ts**

Open `src/main/index.ts` and add Ollama service management:

```typescript
import { startOllamaService, stopOllamaService, getOllamaStatus } from './services/ollama-manager'
```

In the `app.whenReady()` block, after window creation, add:

```typescript
startOllamaService().catch(() => {
  // Ollama start failure is non-fatal, user can retry from UI
})
```

In the `app.on('before-quit')` handler, add:

```typescript
stopOllamaService().catch(() => {})
```

- [ ] **Step 2: Add Ollama status to tray-manager.ts**

Open `src/main/services/tray-manager.ts` and add Ollama status to the tray menu:

```typescript
const ollamaStatus = await getOllamaStatus()
const ollamaLabel = ollamaStatus.running ? '🟢 Ollama: Running' : '🔴 Ollama: Stopped'
```

Add a menu item to start/stop Ollama:

```typescript
{
  label: ollamaLabel,
  type: 'normal',
  click: async () => {
    if (ollamaStatus.running) {
      await stopOllamaService()
    } else {
      await startOllamaService()
    }
    updateTray()
  }
}
```

- [ ] **Step 3: Update onboarder.ts for Ollama configuration**

Open `src/main/services/onboarder.ts` and replace the API key onboarding logic with Ollama configuration:

```typescript
import { configureOpenClawForOllama } from './ollama-manager'

export async function onboardOllama(modelName: string): Promise<void> {
  await configureOpenClawForOllama(modelName)
}
```

Remove any remaining API key or Telegram onboarding functions.

- [ ] **Step 4: Verify typecheck passes**

```bash
cd /workspace/openclaw-desktop
npm run typecheck
```

- [ ] **Step 5: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "feat: integrate Ollama into app lifecycle, tray, and onboarding"
```

---

### Task 10: Update Troubleshooter for Ollama

**Files:**
- Modify: `src/main/services/troubleshooter.ts`

- [ ] **Step 1: Add Ollama diagnostic checks**

Open `src/main/services/troubleshooter.ts` and add:

```typescript
import { getOllamaStatus, startOllamaService } from './ollama-manager'

export async function troubleshootOllama(): Promise<{
  issues: string[]
  fixes: string[]
}> {
  const issues: string[] = []
  const fixes: string[] = []
  const status = await getOllamaStatus()

  if (!status.installed) {
    issues.push('Ollama is not installed')
    fixes.push('Install Ollama from the setup wizard')
  }

  if (status.installed && !status.running) {
    issues.push('Ollama service is not running')
    fixes.push('Start Ollama service')
    try {
      await startOllamaService()
      fixes.push('Auto-fixed: Ollama service started')
    } catch {
      fixes.push('Failed to auto-start Ollama. Try running "ollama serve" manually.')
    }
  }

  return { issues, fixes }
}
```

- [ ] **Step 2: Add Ollama troubleshooter IPC handler**

In `src/main/ipc-handlers.ts`, add:

```typescript
ipcMain.handle('troubleshoot:ollama', async () => {
  return troubleshootOllama()
})
```

- [ ] **Step 3: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "feat: add Ollama diagnostics to troubleshooter"
```

---

### Task 11: Final Polish and Verification

**Files:**
- Modify: `src/renderer/src/assets/main.css` (if branding changes needed)
- Modify: Various i18n files

- [ ] **Step 1: Update app title and branding in main.css**

If the theme uses EasyClaw-specific colors or branding, update to OpenClaw Desktop branding. The primary color (#f97316 orange) can remain as it matches OpenClaw's identity.

- [ ] **Step 2: Run full typecheck**

```bash
cd /workspace/openclaw-desktop
npm run typecheck
```

Expected: Zero errors.

- [ ] **Step 3: Run linter**

```bash
cd /workspace/openclaw-desktop
npm run lint
```

Expected: Zero errors (or fix any that appear).

- [ ] **Step 4: Run dev mode and manually test the wizard flow**

```bash
cd /workspace/openclaw-desktop
npm run dev
```

Verify:
1. Welcome screen shows "OpenClaw Desktop" branding
2. Environment check shows Ollama status
3. OllamaSetupStep shows 3 preset cards
4. Preset recommendation matches system RAM
5. Model pull shows progress
6. Done step shows Ollama + Gateway status

- [ ] **Step 5: Commit**

```bash
cd /workspace/openclaw-desktop
git add -A
git commit -m "chore: final polish and verification"
```
