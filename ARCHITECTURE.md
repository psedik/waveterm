# Wave Terminal Architecture

## Overview
Wave Terminal is an open-source terminal that combines traditional terminal features with graphical capabilities. It's built as an Electron application with a React frontend and a Go backend, following a client-server architecture where the Electron app serves as the client interface while Go handles the core terminal functionality.

## Frontend Architecture

### Framework & Technologies
- **Framework**: React with TypeScript
- **State Management**: Uses Jotai for atomic state management (see `frontend/app/store/`)
- **Styling**: SCSS for styling (see `frontend/app/app.scss`)
- **Build System**: Vite (configured in `electron.vite.config.ts`) with Hot Module Reloading support
- **Network Layer**: Custom fetch abstraction that uses Electron's net module or falls back to browser's fetch API
- **Development Tools**: Chrome DevTools integration for debugging (Cmd+Option+I on macOS, Ctrl+Option+I on Linux/Windows)

### Key Frontend Components
- `frontend/app/`: Contains the main React application components
- `frontend/app/store/`: State management and models
- `frontend/app/block/`: Terminal block components for command isolation and monitoring
- `frontend/layout/`: Flexible drag & drop interface system
- `frontend/util/`: Utility functions and helpers

## Backend Architecture

### Core Technologies
- **Language**: Go
- **Main Components**: 
  - `cmd/server/`: Main server implementation
  - `cmd/wsh/`: Wave Shell implementation for CLI workspace management
  - `db/`: Database management and migrations
- **CGO Integration**: Uses Zig compiler for static linking
- **Platform Support**: 
  - macOS 11+ (arm64, x64)
  - Windows 10 1809+ (x64)
  - Linux with glibc-2.28+ (arm64, x64)

### Communication Layer
- The frontend communicates with the Go backend through a combination of:
  - WebSocket connections for real-time updates (using native WebSocket API)
  - HTTP/REST endpoints for standard requests (using Electron's net module in main process and fetch API in renderer)
  - IPC (Inter-Process Communication) through Electron for system-level operations
- Frontend HTTP requests are abstracted through `frontend/util/fetchutil.ts` which:
  - Uses Electron's net module when available
  - Falls back to browser's fetch API when needed
- The application uses a custom endpoints system (`frontend/util/endpoints.ts`) to manage API endpoints

## Key Components

### Electron Main Process
- Located in `emain/`
- Handles window management
- Manages communication between frontend and backend
- Handles system-level operations
- Logs stored at `~/.waveterm-dev/waveapp.log` during development

### Terminal Implementation
- Custom terminal implementation in Go
- Handles shell sessions and terminal emulation
- Manages terminal state and input/output
- Supports file previews (markdown, images, video, PDFs, CSVs, directories)
- Integrated editor with syntax highlighting
- AI chat integration with multiple model support

### Database
- Uses SQLite for data storage
- Handles:
  - File store
  - Workspace management
  - Terminal session history
  - Block parent relationships
  - Remote connections data

### Configuration
- User settings stored in `~/.config/waveterm/settings.json`
- Terminal themes in `~/.config/waveterm/termthemes.json`
- Application data in platform-specific locations
- Supports rich customization including tab themes, terminal styles, and background images

## Development Workflow

### Prerequisites
- NodeJS 22 LTS with Corepack enabled
- Go
- Task (taskfile.dev)
- Platform-specific dependencies:
  - Linux: zip, zig compiler
  - Windows: zig compiler
  - macOS: no specific dependencies

### Building
- Uses Task (Taskfile.yml) for build automation
- Development modes:
  - `task dev`: Development server with Hot Module Reloading
  - `task start`: Standalone build without dev server
  - `task package`: Production build with installers
- Initial setup: `task init` for dependencies

### Architecture Decisions
1. **Electron**: Chosen for cross-platform desktop capabilities while maintaining web technologies
2. **Go Backend**: Selected for:
   - Efficient terminal handling
   - Cross-platform compatibility
   - Strong standard library
   - CGO capabilities with Zig static linking
3. **React Frontend**: Provides:
   - Component-based architecture
   - Efficient rendering
   - Rich ecosystem of tools and libraries
   - Hot Module Reloading support

### File Structure
```
waveterm/
├── cmd/                    # Go commands and entry points
├── db/                    # Database management
├── emain/                 # Electron main process
├── frontend/             # React frontend application
│   ├── app/              # Main React components
│   ├── layout/           # Layout system
│   └── util/             # Frontend utilities
├── pkg/                  # Go packages
└── public/               # Static assets
```

## Development Guidelines

### Adding New Features
1. Frontend changes:
   - Add components in appropriate directories under `frontend/app/`
   - Update state management in `frontend/app/store/` if needed
   - Style with SCSS in component-specific files
   - Test using Chrome DevTools

2. Backend changes:
   - Add new endpoints in appropriate Go packages
   - Update database schema if needed (add migrations)
   - Follow Go best practices and error handling patterns
   - Check logs in `~/.waveterm-dev/waveapp.log`

### Testing
- Frontend: Jest for unit tests
- Backend: Go testing framework
- E2E testing: Custom test driver in `testdriver/`
- Manual testing via development server with HMR

## Building and Packaging
The application uses electron-builder for packaging, configured in `electron-builder.config.cjs`. The build process:
1. Compiles TypeScript/React frontend
2. Builds Go backend binaries with Zig for CGO
3. Packages everything into platform-specific installers

Artifacts are generated in the `make/` directory, including:
- DMG installers for macOS
- ZIP archives for portable distribution
- Platform-specific builds (arm64, x64, universal)
- Additional formats for Linux (deb, rpm, snap)
