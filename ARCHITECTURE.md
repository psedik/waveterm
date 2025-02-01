# Wave Terminal Architecture

## Overview
Wave Terminal is built as an Electron application with a React frontend and a Go backend. The application follows a client-server architecture where the Electron app serves as the client interface while Go handles the core terminal functionality.

## Frontend Architecture

### Framework & Technologies
- **Framework**: React with TypeScript
- **State Management**: Uses Jotai for atomic state management (see `frontend/app/store/`)
- **Styling**: SCSS for styling (see `frontend/app/app.scss`)
- **Build System**: Vite (configured in `electron.vite.config.ts`)
- **Network Layer**: Custom fetch abstraction that uses Electron's net module or falls back to browser's fetch API

### Key Frontend Components
- `frontend/app/`: Contains the main React application components
- `frontend/app/store/`: State management and models
- `frontend/app/block/`: Terminal block components
- `frontend/layout/`: Layout management system
- `frontend/util/`: Utility functions and helpers

## Backend Architecture

### Core Technologies
- **Language**: Go
- **Main Components**: 
  - `cmd/server/`: Main server implementation
  - `cmd/wsh/`: Wave Shell implementation
  - `db/`: Database management and migrations

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

### Terminal Implementation
- Custom terminal implementation in Go
- Handles shell sessions and terminal emulation
- Manages terminal state and input/output

### Database
- Uses SQLite for data storage
- Handles:
  - File store
  - Workspace management
  - Terminal session history
  - Block parent relationships

### Configuration
- User settings stored in `~/.config/waveterm/settings.json`
- Terminal themes in `~/.config/waveterm/termthemes.json`
- Application data in platform-specific locations

## Development Workflow

### Building
- Uses Task (Taskfile.yml) for build automation
- Development server: `task dev`
- Production build: `task package`

### Architecture Decisions
1. **Electron**: Chosen for cross-platform desktop capabilities while maintaining web technologies
2. **Go Backend**: Selected for:
   - Efficient terminal handling
   - Cross-platform compatibility
   - Strong standard library
3. **React Frontend**: Provides:
   - Component-based architecture
   - Efficient rendering
   - Rich ecosystem of tools and libraries

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

2. Backend changes:
   - Add new endpoints in appropriate Go packages
   - Update database schema if needed (add migrations)
   - Follow Go best practices and error handling patterns

### Testing
- Frontend: Jest for unit tests
- Backend: Go testing framework
- E2E testing: Custom test driver in `testdriver/`

## Building and Packaging
The application uses electron-builder for packaging, configured in `electron-builder.config.cjs`. The build process:
1. Compiles TypeScript/React frontend
2. Builds Go backend binaries
3. Packages everything into platform-specific installers

Artifacts are generated in the `make/` directory, including:
- DMG installers for macOS
- ZIP archives for portable distribution
- Platform-specific builds (arm64, x64, universal)
