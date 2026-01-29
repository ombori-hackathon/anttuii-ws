# Feature: Anttuii - User-Friendly Terminal App

## Summary
A mouse-friendly terminal app for macOS with smart command completions, tabbed interface, file browser sidebar, git integration, and file previewing - designed to make the terminal accessible to users unfamiliar with bash commands.

## Trigger
- User launches the Anttuii app
- User types commands (triggers completions)
- User interacts with sidebar (triggers file operations, previews)
- File system changes in git repos (triggers status indicators)

## Expected Result
A functional macOS terminal app with:
1. **Core terminal** with multiple tabs (browser-like "+" button)
2. **Smart completions** via large inline dropdown (arrow/enter navigation)
3. **File browser sidebar** with directory tree and installed apps list
4. **Git integration** with visual status indicators
5. **Preview panel** for file contents and git diffs

## Edge Cases
- No backend connection: Terminal works, completions disabled with error toast
- Empty completions: Dropdown hidden gracefully
- Tab close with running process: Confirmation dialog
- Large man pages: Truncation/pagination
- Binary files in preview: Show "Cannot preview" message

## Technical Design

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Anttuii App                          │
├─────────────┬───────────────────────────────┬───────────────┤
│  Sidebar    │      Terminal Tabs            │   Preview     │
│  (toggle)   │  [Tab1] [Tab2] [+]            │   (toggle)    │
│             ├───────────────────────────────┤               │
│  Files      │                               │   File        │
│  ├── src/   │  $ git comm█                  │   Contents    │
│  ├── docs/  │  ┌─────────────────────┐      │   or          │
│  └── ...    │  │ commit      Record  │      │   Git Diff    │
│             │  │ commit -m   With... │      │               │
│  Apps       │  │ commit -am  All...  │      │               │
│  ├── git    │  └─────────────────────┘      │               │
│  └── claude │       (completion dropdown)   │               │
└─────────────┴───────────────────────────────┴───────────────┘
```

### API Changes

**POST /api/v1/completions**
- Request: `{ input: string, cursor_position: int, cwd: string, shell: string }`
- Response: `{ completions: [{text, type, description, insert_text, score}], prefix_length: int }`

**GET /api/v1/commands**
- Response: `{ commands: [{name, path, description}], last_indexed: datetime }`

**GET /api/v1/commands/{name}/help**
- Response: `{ synopsis: string, description: string, options: [{flag, description}] }`

### Database Changes

New tables:
- `commands` (id, name, path, description, indexed_at)
- `subcommands` (id, command_id, name, description)
- `command_options` (id, command_id, subcommand_id, short_flag, long_flag, description, takes_value)

### Swift Client Changes

New dependency: SwiftTerm (terminal emulation)

New components:
- `AnttuiiTerminalView`: Custom terminal with input interception
- `TerminalWrapper`: NSViewRepresentable for SwiftUI
- `TabBar`, `TabContainerView`: Tab management
- `CompletionOverlay`, `CompletionRow`: Dropdown UI
- `AppState`, `CompletionState`: Observable state
- `APIClient`, `CompletionManager`: Backend communication

## Implementation Plan

### Phase 1: Core Terminal + Tabs (MVP)
1. [x] Plan and spec creation
2. [ ] Backend: Command models and migrations
3. [ ] Backend: CommandIndexer service (scan PATH)
4. [ ] Backend: ManParser service
5. [ ] Backend: CompletionEngine service
6. [ ] Backend: Completions API endpoints
7. [ ] Backend: Tests
8. [ ] Swift: Add SwiftTerm, create terminal wrapper
9. [ ] Swift: Tab system (TabBar, TabContainerView)
10. [ ] Integration test

### Phase 2: Smart Completions UI
1. [ ] Swift: APIClient and CompletionManager
2. [ ] Swift: CompletionOverlay dropdown UI
3. [ ] Swift: Keyboard navigation (arrows, enter, escape)
4. [ ] Swift: Completion insertion into terminal
5. [ ] End-to-end testing

### Phase 3: File Browser Sidebar (Future)
1. [ ] FileBrowserView with OutlineGroup
2. [ ] AppsListView
3. [ ] File operations context menu

### Phase 4: Git Integration (Future)
1. [ ] GitService for status detection
2. [ ] Status indicators in file browser
3. [ ] Branch display

### Phase 5: Preview Panel (Future)
1. [ ] Text file preview
2. [ ] Git diff preview

## Verification

```bash
# Backend
cd services/api
uv run alembic upgrade head
uv run fastapi dev
curl -X POST http://localhost:8000/api/v1/completions \
  -H "Content-Type: application/json" \
  -d '{"input": "git comm", "cursor_position": 8}'
uv run pytest -v

# Frontend
cd apps/macos-client
swift run AnttuiiClient
# Verify: terminal works, tabs work, completions appear
```

## Status
- [ ] Spec approved
- [ ] Phase 1 complete
- [ ] Phase 2 complete
- [ ] PR created
