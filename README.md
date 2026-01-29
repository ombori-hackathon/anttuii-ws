# Anttuii Workspace

Monorepo workspace for the Anttuii terminal application.

## Structure

```
├── apps/macos-client/   # SwiftUI macOS app
├── services/api/        # FastAPI backend
└── specs/               # Feature specifications
```

## Quick Start

```bash
# Clone with submodules
git clone --recursive <repo-url>

# Start database
docker compose up -d

# Run API
cd services/api && uv run fastapi dev

# Run client (new terminal)
cd apps/macos-client && swift run AnttuiiClient
```

## About

Anttuii is a user-friendly terminal app designed to make the command line accessible with smart completions, visual file browsing, and git integration.
