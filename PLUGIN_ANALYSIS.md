# Dictator Plugin Analysis

## Overview

The **Dictator** plugin is a VS Code/Cursor extension that adds voice recording and transcription capabilities to the Cursor AI chat interface. It also includes "Power Tools" - a set of developer-focused templates and shortcuts for common AI interactions.

## Architecture

### Core Components

1. **extension.js** - Main extension entry point (Node.js/VSCode API)
2. **dictator.js** - Voice recording and ASR (Automatic Speech Recognition) functionality
3. **power-tools.js** - Developer tools and template system
4. **statusbar.js** - Status bar indicator
5. **messages.js** - User-facing messages

### How It Works

#### 1. Installation & Activation

The extension works by **patching the VS Code/Cursor workbench HTML file**:

- Locates the workbench HTML file (`workbench.html`, `workbench-dev.html`, etc.)
- Creates a backup of the original file
- Injects JavaScript code directly into the HTML file
- Requires restart to take effect

**Key Code** (from `extension.js`):
```javascript
function locateWorkbench() {
  // Finds workbench HTML in:
  // - electron-browser/workbench/
  // - electron-browser/
  // - electron-sandbox/workbench/
  // - electron-sandbox/
}
```

#### 2. Voice Recording (dictator.js)

**Technology Stack:**
- Uses **OpenAI Whisper** models via Hugging Face Transformers.js
- Runs entirely in the browser using **WebGPU**
- Supports multiple Whisper models (base, small, medium, large)
- Processes audio locally - no external API calls

**Flow:**
1. User clicks microphone button in chat interface
2. Browser requests microphone access (`getUserMedia`)
3. Audio is recorded using `MediaRecorder` API
4. Audio is processed and resampled to 16kHz (Whisper's target rate)
5. Audio is sent to a Web Worker for transcription
6. Worker uses Hugging Face Transformers.js to load Whisper model
7. Transcription happens in real-time using streaming
8. Text is inserted into the chat input field

**Key Features:**
- Real-time transcription with streaming updates
- Support for 90+ languages
- Multiple model sizes (trade-off between quality and speed)
- Audio visualization waveform during recording
- Cancel/stop functionality

#### 3. Power Tools (power-tools.js)

Power Tools provides a context menu system with customizable templates for common AI interactions.

**Features:**
- Pre-built templates (test generation, bug analysis, documentation, etc.)
- Custom user templates
- Keyboard shortcuts (hotkeys)
- Category grouping
- Context menu integration

**Templates Structure:**
- Each template has:
  - ID, name, description
  - Prompt text (what gets sent to AI)
  - Category (development, debugging, documentation, learning, general)
  - Optional hotkey
  - Icon (Codicon)

**Usage:**
- Right-click the flash icon (⚡) in chat interface
- Select a template from the context menu
- Template prompt is inserted into chat input
- Optionally auto-sends the message

#### 4. Status Bar Integration (statusbar.js)

Adds a status indicator to the VS Code status bar showing the extension is active.

## File Structure

```
extension/
├── package.json          # Extension manifest
├── extension.js          # Main extension logic (Node.js)
├── dictator.js           # Voice recording (browser, minified)
├── power-tools.js        # Power Tools UI (browser)
├── statusbar.js          # Status bar indicator (browser)
└── messages.js           # User messages
```

## Key Technical Details

### Workbench Patching

The extension injects code by:
1. Finding the workbench HTML file in VS Code installation
2. Creating a backup with UUID
3. Injecting script tags before `</html>`
4. Using markers: `<!-- !! DICTATOR-START !! -->` and `<!-- !! DICTATOR-END !! -->`

### ASR System

- Uses Web Workers for audio processing (keeps UI responsive)
- Loads Whisper models from Hugging Face ONNX format
- Models are cached locally in browser
- Supports chunking for long audio (>30 seconds)
- Real-time streaming transcription

### Configuration

Settings are stored in VS Code workspace configuration:
- `dictator.statusbar` - Show status indicator
- `dictator.powerTools.showHotkeys` - Show keyboard shortcuts
- `dictator.powerTools.groupByCategory` - Group templates by category
- `dictator.powerTools.factoryTemplates` - Built-in templates
- `dictator.powerTools.customTemplates` - User templates

### Browser Compatibility

- Requires WebGPU support (for Whisper model inference)
- Requires MediaRecorder API (for audio recording)
- Requires Web Workers (for background processing)

## Limitations & Notes

1. **Requires Admin Rights**: The extension modifies VS Code installation files, which may require admin privileges
2. **Workbench Patching**: This is a hacky approach that modifies core VS Code files - could break with updates
3. **WebGPU Required**: Voice features won't work without WebGPU support
4. **Memory Intensive**: Larger Whisper models require significant RAM (16GB+ recommended for medium/large)
5. **Local Processing**: All audio processing happens locally - good for privacy but requires capable hardware

## Security Considerations

- No audio data leaves the user's machine
- All processing happens in browser using local models
- Extension modifies VS Code installation files (potential security concern)

## Dependencies

- `file-url` - File URL handling
- `node-fetch` - HTTP requests for model loading
- `uuid` - UUID generation for backups
- Hugging Face Transformers.js (loaded from CDN)

## Extracted Files

All source code has been extracted to `extracted-source/` directory:
- `extension.js` - Readable Node.js code
- `dictator.js` - Minified/bundled browser code
- `power-tools.js` - Readable browser code
- `statusbar.js` - Readable browser code
- `messages.js` - Simple message strings
- `package.json` - Extension manifest

