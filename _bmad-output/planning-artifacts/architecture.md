---
stepsCompleted: [1, 2, 3, 4, 5, 6]
inputDocuments: ['prd.md', 'prd-validation-report.md', 'product-brief-p2p-tunnel.md']
workflowType: 'architecture'
project_name: 'peertunnel'
user_name: 'Root'
date: '2026-04-08'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
46 FRs across 9 domains:
- Tunnel Hosting (FR1-FR8): CLI exposes local HTTP port, generates share link, tracks connections, handles graceful shutdown and reconnection
- Tunnel Viewing (FR9-FR16): Browser-based access via shared link, real-time status, full HTTP/WebSocket interactivity, reconnection UX, connection type indicator
- Viewer Welcome & Manual Connect (FR17-FR19): Landing page with manual connection input, auto-connect via URL fragment
- P2P Networking (FR20-FR26): Multi-transport P2P (WebTransport + WebRTC-Direct), relay fallback, NAT hole-punching, peer discovery, E2E encryption, token auth
- Content Proxying (FR27-FR30): Service Worker intercepts all HTTP requests, sub-resources, and client-side routing — transparent proxy through P2P channel. WebSocket proxying requires a shimming approach (see Technical Constraints).
- Relay Operations (FR31-FR37): Same binary in relay mode, configurable limits (connections, duration, bandwidth), peer allowlist, protocol filtering
- Port Validation (FR38-FR39): Soft HTTP check before tunneling, warn-don't-block
- Telemetry (FR40-FR43): Opt-in anonymous tracking, local random ID, disable option
- Configuration (FR44-FR46): Persistent user-level config for telemetry, CLI flags for session settings

**Non-Functional Requirements:**
20 NFRs across 5 categories:
- Performance: <3s direct connect, <5s relay, <100ms HTTP overhead, <50ms WebSocket overhead, <10ms SW proxy overhead
- Security: E2E encryption (Noise), token auth, relay peer verification, no PII in telemetry, config file permissions 600
- Scalability: 5 viewers/tunnel (configurable), 50 relay sessions/node (configurable), no central bottleneck
- Reliability: 5+ hour stable sessions, auto-reconnect, graceful degradation to relay, clean shutdown on signals
- Compatibility: Linux/macOS/Windows (amd64+arm64), all Chromium browsers + Safari, mobile-friendly viewer shell

**Scale & Complexity:**

- Primary domain: Developer tools / P2P networking
- Complexity level: Medium
- Estimated architectural components: 6 (CLI core, libp2p networking layer, relay mode, web viewer shell, Service Worker proxy, shared protocol/auth)

### Technical Constraints & Dependencies

- Go CLI must produce a single static binary with zero external runtime dependencies
- Web viewer must be 100% static (no backend) — deployable to any static host (GitHub Pages, Netlify, etc.)
- libp2p Go and JS implementations must interoperate on chosen transports and protocols
- **WebSocket proxying limitation:** Service Workers cannot intercept `new WebSocket()` calls (the `fetch` event does not fire for WebSocket connections). WebSocket support for tunneled content (TTYD, VS Code Server, etc.) requires a WebSocket constructor shimming/patching approach in the viewer — intercepting WebSocket creation in the tunneled page context and bridging frames over libp2p streams. This is a parallel mechanism alongside the Service Worker HTTP proxy, not part of it.
- Browser transport support dictates viewer capability: WebTransport (Chromium, Firefox) vs WebRTC-Direct (Safari)
- Community relay infrastructure (1-2 VPS nodes) must be hardcoded as defaults in the CLI

### Cross-Cutting Concerns Identified

- **Connection lifecycle management** — connect, authenticate, stream, reconnect, disconnect — must be consistent across CLI and viewer, across direct and relay paths
- **Transport negotiation** — the system must transparently select and fall back between WebTransport, WebRTC-Direct, and relay without user intervention
- **Error propagation across P2P boundary** — CLI errors must surface meaningfully in the browser viewer (e.g., "tunnel closed by host" vs generic disconnect)
- **Authentication** — token generation (CLI side) and validation (viewer side) must use a shared scheme embedded in the URL fragment
- **Reconnection state machine** — both CLI and viewer need coordinated reconnection logic with the <3s silent / >3s visible threshold
- **Protocol framing** — HTTP request/response serialization over libp2p streams must handle headers, bodies, chunked transfer, and WebSocket upgrade consistently
- **Dual proxy mechanism** — Service Worker handles HTTP traffic; WebSocket shim handles WebSocket traffic. Both route through the same libp2p connection but use different interception strategies. The architecture must ensure these two mechanisms don't conflict and share connection state cleanly.

## Starter Template Evaluation

### Primary Technology Domain

Two-component monorepo: Go CLI tool + static TypeScript web application. Technology choices are PRD-specified (Go + Lit/TypeScript/Vite), so starter evaluation focuses on scaffolding tools, not technology selection.

### Starter Options Considered

**Go CLI:**
- **cobra-cli** (v1.3.0, Cobra library v1.9.x) — standard Go CLI scaffolding. Generates `main.go` + `cmd/` subcommand files. Used by kubectl, gh, hugo. Minimal scaffold — project layout (`internal/`, etc.) built manually.
- **Manual setup** (`go mod init` + cobra as library dependency) — equivalent outcome, slightly more control over initial structure.

**Web Viewer:**
- **Vite lit-ts template** (Lit 3.3.x, Vite 9.x) — official Vite template for Lit + TypeScript. Generates `index.html`, sample component, `tsconfig.json`, `vite.config.ts`. Minimal — no testing, linting, or routing.
- **Open WC generator** — more opinionated (testing, linting, Storybook). Heavier than needed for a thin viewer shell.

### Selected Starters

**Go CLI: cobra-cli + conventional Go layout**

**Rationale:** Cobra handles subcommand routing (`serve`, `relay`, `config`, `version`), flag parsing, help generation, and future shell completion. The conventional Go project layout (`internal/`, `cmd/`) provides clear separation. Minimal scaffolding matches a greenfield project where most code is custom (libp2p networking, tunnel logic).

**Web Viewer: Vite lit-ts template**

**Rationale:** Official, actively maintained, minimal footprint. The viewer is a thin shell around tunneled content — no routing framework, no state management library, no component library needed. Lit's Web Components approach keeps the viewer lightweight with minimal dependencies.

### Initialization Commands

```bash
# Go CLI (from project root)
mkdir -p cli && cd cli
go mod init github.com/farajfarook/peertunnel
go install github.com/spf13/cobra-cli@latest
cobra-cli init
cobra-cli add serve
cobra-cli add relay
cobra-cli add config

# Web Viewer (from project root)
npm create vite@latest viewer -- --template lit-ts
```

### Architectural Decisions Provided by Starters

**Language & Runtime:**
- Go (latest stable) for CLI — single static binary, cross-compilation via GOOS/GOARCH
- TypeScript (strict mode) for viewer — Vite handles transpilation and bundling

**Build Tooling:**
- Go: `go build` produces static binary. Cross-platform builds via `GOOS=linux/darwin/windows GOARCH=amd64/arm64`
- Viewer: Vite dev server for development, `vite build` for production static output

**Code Organization:**
- CLI: `cmd/` (Cobra subcommands) + `internal/` (tunnel, relay, p2p, config packages)
- Viewer: `src/` (Lit components, p2p connection, proxy layer)

**Development Experience:**
- Go: standard `go run`, `go test`, `go vet`
- Viewer: Vite HMR dev server, TypeScript type checking

### Monorepo Structure

```
peertunnel/
├── cli/                    # Go CLI
│   ├── cmd/                # Cobra subcommands (serve, relay, config)
│   ├── internal/
│   │   ├── tunnel/         # Tunnel hosting logic
│   │   ├── relay/          # Relay mode logic
│   │   ├── p2p/            # libp2p networking layer
│   │   └── config/         # Config file management
│   ├── go.mod
│   └── main.go
├── viewer/                 # Static web viewer
│   ├── src/
│   │   ├── components/     # Lit components (welcome, logs, status)
│   │   ├── p2p/            # js-libp2p connection management
│   │   ├── proxy/          # Service Worker + WebSocket shim
│   │   └── index.ts
│   ├── public/
│   ├── package.json
│   └── vite.config.ts
└── docs/
```

**Note:** Project initialization using these commands should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- P2P stream strategy (hybrid: control stream + per-request data streams)
- HTTP serialization format (Protocol Buffers)
- Token authentication scheme (crypto-random)
- Tunneled content rendering (Service Worker full-page takeover)
- WebSocket shim injection (SW script injection into HTML responses)

**Important Decisions (Shape Architecture):**
- Share link format (dot-separated fragment)
- Viewer state management (Lit reactive properties)
- CLI distribution (GitHub Releases + go install)
- Viewer hosting (GitHub Pages)
- CI/CD (GitHub Actions + GoReleaser)

**Deferred Decisions (Post-MVP):**
- Shell completion generation
- HTTPS on local side
- JSON output mode
- Named/persistent tunnels
- Custom viewer domains

### P2P Protocol Design

**Stream Multiplexing Strategy: Hybrid**
- Decision: Persistent control stream for tunnel lifecycle messages + new libp2p stream per HTTP request for data
- Rationale: Control messages (tunnel closed, viewer count, shutdown signals) need a persistent channel. Per-request streams keep HTTP proxying simple — libp2p's yamux handles underlying multiplexing. No custom multiplexing protocol needed.
- Affects: CLI tunnel package, viewer P2P module

**HTTP Serialization: Protocol Buffers**
- Decision: Define HTTP request/response messages in protobuf, shared between Go and TypeScript
- Rationale: Type-safe, cross-language, handles serialization edge cases (binary bodies, header encoding). Minimal dependency footprint. Shared .proto files ensure CLI and viewer agree on wire format.
- Affects: CLI tunnel package, viewer proxy module, shared proto definitions

**WebSocket Frame Bridging: Dedicated Stream Per WebSocket**
- Decision: Each WebSocket connection from tunneled content gets its own libp2p stream
- Rationale: Clean 1:1 mapping to WebSocket semantics. Each connection is independently managed (open/close without affecting others). libp2p handles multiplexing at the transport layer.
- Affects: CLI tunnel package, viewer WebSocket shim

### Authentication & Security

**Token Format: Crypto-Random Bytes**
- Decision: 32 bytes, base64url-encoded, generated on tunnel startup
- Rationale: Token's only purpose is preventing unauthorized viewers. No expiry needed (tunnel lifetime = token lifetime). No server to verify against. 256 bits of entropy is more than sufficient. CLI validates by simple comparison.
- Affects: CLI serve command, viewer P2P connection module

**Share Link Format: Dot-Separated Fragment**
- Decision: `https://viewer.peertunnel.dev/#<peerID>.<token>`
- Rationale: Matches PRD examples. Human-scannable (clear boundary between peer ID and token). Extensible via additional dot-separated segments post-MVP. Fragment is never sent to server.
- Affects: CLI link generation, viewer URL parsing

### Frontend Architecture

**Tunneled Content Rendering: Full-Page Takeover**
- Decision: Once connected, viewer navigates to a Service Worker-controlled URL that serves tunneled content directly. Viewer shell disappears. Shell reappears on disconnect/error.
- Rationale: PRD user journeys describe the tunneled app "filling the browser." Full-page takeover gives tunneled apps the complete viewport, avoids iframe sandboxing issues (some apps detect and refuse iframe embedding), and provides the most transparent proxy experience.
- Affects: Viewer components, Service Worker, reconnection UX

**Viewer State Management: Lit Reactive Properties**
- Decision: Connection state machine managed via reactive properties in the root Lit component. No external state library.
- Rationale: Viewer state is simple and linear (idle → connecting → connected → disconnected/reconnecting → error). No shared state across unrelated components. Lit's built-in reactivity is sufficient.
- Affects: Viewer components

**WebSocket Shim Injection: Service Worker Script Injection**
- Decision: Service Worker injects a shim script at the top of HTML responses before serving them. The shim overrides `window.WebSocket` with a custom implementation that bridges frames to the P2P connection via MessageChannel.
- Rationale: Full-page takeover means no iframe `contentWindow` access. SW script injection runs before any tunneled app code, ensuring all WebSocket connections are captured. MessageChannel provides efficient communication between the shim and SW.
- Affects: Service Worker proxy, WebSocket shim module

### Infrastructure & Deployment

**Viewer Hosting: GitHub Pages**
- Decision: Deploy static viewer to GitHub Pages with custom domain (viewer.peertunnel.dev)
- Rationale: Zero cost, zero vendor lock-in, automatic deploy via GitHub Actions. 100% static site requires no server-side configuration. Users can self-host on any static provider.
- Affects: CI/CD pipeline, viewer build

**CLI Distribution: GitHub Releases + go install**
- Decision: GoReleaser builds cross-platform binaries attached to GitHub Releases. Go developers can also use `go install github.com/farajfarook/peertunnel/cli@latest`.
- Rationale: GitHub Releases covers non-Go users (pre-built binaries). `go install` requires zero setup — Go modules pull directly from the public Git repo via proxy.golang.org. No registry accounts or API keys needed.
- Affects: CI/CD pipeline, release process

**CI/CD: GitHub Actions + GoReleaser**
- Decision: GitHub Actions for CI (Go build/test, Vite build/test on PRs), GoReleaser on tag push for releases
- Rationale: Free for public repos, standard for open-source Go projects, native GitHub integration.
- Affects: Repository CI configuration

### Decision Impact Analysis

**Implementation Sequence:**
1. Shared proto definitions (wire format for HTTP req/res and control messages)
2. CLI libp2p networking layer (stream management, transport setup)
3. CLI tunnel hosting (HTTP proxying to localhost, WebSocket bridging)
4. Viewer P2P connection module (js-libp2p, transport negotiation)
5. Viewer Service Worker proxy (HTTP interception, proto deserialization)
6. Viewer WebSocket shim (constructor patching, MessageChannel bridge)
7. CLI relay mode (Circuit Relay v2 configuration)
8. Integration: share link generation → URL parsing → auth → full flow

**Cross-Component Dependencies:**
- Proto definitions must be shared/synchronized between CLI and viewer
- Token format must match between CLI generation and viewer validation
- Stream protocol IDs must be identical on both sides
- Control message schema affects both CLI connection tracking and viewer reconnection UX

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**15 conflict points identified** where AI agents could make different implementation choices. Patterns below ensure all agents produce compatible, consistent code.

### Naming Patterns

**Go Package Naming (internal/):**
- Use domain-oriented names: `tunnel`, `relay`, `p2p`, `config`
- NOT library-oriented names (`libp2p`, `cobra`, `yaml`)
- Package names are single lowercase words (Go convention)

**TypeScript File Naming:**
- Kebab-case for all files: `connection-status.ts`, `peer-connection.ts`
- PascalCase for class/component names: `class ConnectionStatus`
- Follows Lit community conventions

**Lit Component Tag Naming:**
- Prefix all custom elements with `pt-`: `<pt-connection-status>`, `<pt-welcome-page>`, `<pt-connection-log>`
- Avoids collisions with tunneled content's custom elements
- Short, recognizable namespace

**Proto Message Naming:**
- PascalCase messages: `HttpRequest`, `HttpResponse`, `ControlMessage`
- snake_case fields: `request_url`, `status_code`, `header_entries`
- Follows standard protobuf style guide

**libp2p Protocol IDs:**
- Format: `/peertunnel/<protocol>/<version>`
- HTTP tunnel: `/peertunnel/http/1.0.0`
- WebSocket tunnel: `/peertunnel/ws/1.0.0`
- Control channel: `/peertunnel/control/1.0.0`
- Versioned to allow future protocol changes without breaking compatibility

### Structure Patterns

**Test Location:**
- Go: `*_test.go` co-located with source (Go standard)
- TypeScript: `*.test.ts` co-located with source (e.g., `connection-status.test.ts` next to `connection-status.ts`)
- Consistent co-location across both components

**Proto File Location:**
- `proto/` directory at project root — single source of truth
- Both `cli/` and `viewer/` reference root `proto/` for codegen
- Generated code placed in `cli/internal/proto/` and `viewer/src/proto/`
- `.proto` files are the canonical definition; generated files are not committed

### Format Patterns

**CLI Log Output:**
- User-facing format: `[HH:MM:SS] <event description> — <context>` (matches PRD examples)
- Internal logging: Go `slog` package with structured fields
- Terminal output is human-readable; structured logging available at Debug level

**Control Message Types (Proto Enum):**
- SCREAMING_SNAKE_CASE for proto enum values: `TUNNEL_CLOSED`, `VIEWER_CONNECTED`, `VIEWER_DISCONNECTED`, `RECONNECTING`
- Standard protobuf enum convention

**Error Messages Across P2P Boundary:**
- Structured proto message: `ErrorMessage { ErrorCode code = 1; string detail = 2; }`
- Error codes as proto enum: `TOKEN_INVALID`, `MAX_VIEWERS_REACHED`, `TUNNEL_SHUTTING_DOWN`, `INTERNAL_ERROR`
- Both CLI and viewer handle all error codes explicitly — no generic catch-all

### Communication Patterns

**Viewer Connection State Machine:**
- Defined states: `idle` → `connecting` → `connected` → `disconnected` → `reconnecting` → `error`
- Transitions are explicit — no implicit state changes
- All state transitions logged to connection log UI
- State is a single `connectionState` reactive property on the root component

### Process Patterns

**Go Error Handling:**
- Wrap errors with context: `fmt.Errorf("failed to start tunnel: %w", err)`
- Sentinel errors for expected conditions: `var ErrMaxViewers = errors.New("maximum viewers reached")`
- No panics in library code; panics only in `main()` for unrecoverable startup failures
- All errors from libp2p operations are wrapped with peertunnel context before surfacing

**TypeScript Error Handling:**
- Custom error classes: `class ConnectionError extends Error`, `class AuthError extends Error`
- Async errors caught and propagated to Lit component state (triggers UI update)
- No silent error swallowing — all caught errors either update UI state or are re-thrown

**Graceful Shutdown Sequence (CLI):**
1. Receive SIGINT/SIGTERM
2. Send `TUNNEL_SHUTTING_DOWN` control message to all connected viewers
3. Wait up to 2 seconds for viewer acknowledgments
4. Close all libp2p streams and host
5. Print session summary (duration, total connections)
6. Exit 0

**Logging Levels:**
- Go `slog`: `Debug` (libp2p internals, stream details), `Info` (connection events — user-visible), `Warn` (port validation, reconnection attempts), `Error` (failures)
- TypeScript: `console.debug` / `console.info` / `console.warn` / `console.error` with same semantic mapping
- Default CLI verbosity shows Info and above; `--verbose` flag enables Debug

### Enforcement Guidelines

**All AI Agents MUST:**
- Follow Go naming conventions enforced by `gofmt` and `go vet` — no exceptions
- Use `pt-` prefix for all Lit custom element tags
- Use kebab-case for all TypeScript files
- Reference proto definitions from root `proto/` directory — never duplicate message definitions
- Use versioned protocol IDs for all libp2p streams
- Wrap all Go errors with context before returning
- Map all P2P errors to defined proto error codes — no raw string errors across the boundary

**Anti-Patterns to Avoid:**
- Creating utility/helper packages (`utils/`, `helpers/`) — put code where it belongs by domain
- Using `any` type in TypeScript — all P2P messages are typed via proto-generated code
- Catching errors without handling them (empty catch blocks)
- Hardcoding libp2p multiaddrs outside of config — use constants or config values
- Mixing human-readable log output with structured logging in the same output stream

## Project Structure & Boundaries

### Complete Project Directory Structure

```
peertunnel/
├── .github/
│   └── workflows/
│       ├── ci.yml                      # Go test/vet + Vite build/typecheck on PRs
│       ├── release.yml                 # GoReleaser on tag push
│       └── deploy-viewer.yml           # Build + deploy viewer to GitHub Pages on main
├── .gitignore
├── .goreleaser.yml                     # GoReleaser config for cross-platform builds
├── LICENSE
├── README.md
├── CLAUDE.md
│
├── proto/                              # Shared protobuf definitions (source of truth)
│   ├── tunnel.proto                    # HttpRequest, HttpResponse messages
│   ├── control.proto                   # ControlMessage, ErrorMessage, enums
│   └── buf.yaml                        # Buf configuration for linting/codegen
│
├── cli/                                # Go CLI
│   ├── main.go                         # Entry point, Cobra root command
│   ├── go.mod
│   ├── go.sum
│   ├── cmd/
│   │   ├── root.go                     # Root command, global flags, version
│   │   ├── serve.go                    # `peertunnel serve` subcommand
│   │   ├── relay.go                    # `peertunnel relay` subcommand
│   │   └── config.go                   # `peertunnel config` subcommand
│   └── internal/
│       ├── p2p/
│       │   ├── host.go                 # libp2p host creation, transport config
│       │   ├── host_test.go
│       │   ├── transports.go           # WebTransport, WebRTC-Direct setup
│       │   └── discovery.go            # Bootstrap peer discovery, DCUtR
│       ├── tunnel/
│       │   ├── server.go               # Tunnel server: accept viewers, manage streams
│       │   ├── server_test.go
│       │   ├── proxy.go                # HTTP proxy: forward requests to localhost
│       │   ├── proxy_test.go
│       │   ├── websocket.go            # WebSocket bridge: libp2p stream ↔ local WS
│       │   ├── websocket_test.go
│       │   ├── validate.go             # Port validation (soft HTTP check)
│       │   └── token.go                # Token generation (crypto-random 32 bytes)
│       ├── relay/
│       │   ├── server.go               # Circuit Relay v2 node with resource limits
│       │   ├── server_test.go
│       │   └── limiter.go              # Per-peer bandwidth, connection, duration limits
│       ├── config/
│       │   ├── config.go               # Config file read/write (~/.peertunnel/config.yaml)
│       │   └── config_test.go
│       ├── telemetry/
│       │   ├── telemetry.go            # Opt-in anonymous telemetry, local random ID
│       │   └── telemetry_test.go
│       └── proto/                      # Generated Go protobuf code (from root proto/)
│           ├── tunnel.pb.go
│           └── control.pb.go
│
├── viewer/                             # Static web viewer (Lit + TypeScript + Vite)
│   ├── index.html                      # Entry point, loads app shell
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── public/
│   │   ├── favicon.ico
│   │   └── og-image.png               # Open Graph image for link previews
│   └── src/
│       ├── index.ts                    # App bootstrap, SW registration
│       ├── components/
│       │   ├── pt-app-shell.ts         # Root component, connection state machine
│       │   ├── pt-app-shell.test.ts
│       │   ├── pt-welcome-page.ts      # Welcome/idle state, manual connect input
│       │   ├── pt-connection-log.ts    # Real-time connection status log
│       │   ├── pt-reconnecting.ts      # Reconnection indicator (>3s)
│       │   └── pt-error-view.ts        # Error states (tunnel closed, auth failed)
│       ├── p2p/
│       │   ├── connection.ts           # js-libp2p connection management
│       │   ├── connection.test.ts
│       │   ├── transports.ts           # WebTransport + WebRTC-Direct setup
│       │   └── url-parser.ts           # Parse peer ID + token from URL fragment
│       ├── proxy/
│       │   ├── service-worker.ts       # SW: intercept fetch, proxy via P2P, inject WS shim
│       │   ├── ws-shim.ts             # WebSocket constructor override (injected into pages)
│       │   ├── message-channel.ts      # MessageChannel bridge between SW and WS shim
│       │   └── request-handler.ts      # Serialize HTTP req to proto, deserialize response
│       ├── proto/                      # Generated TypeScript protobuf code (from root proto/)
│       │   ├── tunnel.ts
│       │   └── control.ts
│       └── styles/
│           └── theme.css               # Minimal viewer shell styles
│
└── docs/                               # Project documentation
    └── getting-started.md
```

### Architectural Boundaries

**P2P Boundary (CLI ↔ Viewer):**
- All communication flows through libp2p streams using protobuf-encoded messages
- Three protocol channels: `/peertunnel/http/1.0.0`, `/peertunnel/ws/1.0.0`, `/peertunnel/control/1.0.0`
- Token validation occurs at connection establishment — before any stream is opened
- No direct function calls or shared runtime between CLI and viewer

**Proxy Boundary (Viewer Shell ↔ Tunneled Content):**
- Service Worker sits between tunneled content and the network — intercepts all fetch events
- WebSocket shim is injected into HTML responses — runs in tunneled content's context
- MessageChannel bridges SW ↔ WS shim communication
- Tunneled content has no awareness of peertunnel — full transparency

**CLI Internal Boundaries:**
- `cmd/` depends on `internal/` — never the reverse
- `internal/p2p/` is the only package that imports libp2p directly
- `internal/tunnel/` uses `internal/p2p/` for stream management, proxies to localhost independently
- `internal/relay/` uses `internal/p2p/` but has no dependency on `internal/tunnel/`

### Requirements to Structure Mapping

| FR Category | CLI Location | Viewer Location |
|---|---|---|
| Tunnel Hosting (FR1-FR8) | `cli/internal/tunnel/` + `cli/cmd/serve.go` | — |
| Tunnel Viewing (FR9-FR16) | — | `viewer/src/components/`, `viewer/src/proxy/` |
| Viewer Welcome (FR17-FR19) | — | `viewer/src/components/pt-welcome-page.ts` |
| P2P Networking (FR20-FR26) | `cli/internal/p2p/` | `viewer/src/p2p/` |
| Content Proxying (FR27-FR30) | `cli/internal/tunnel/proxy.go` | `viewer/src/proxy/` |
| Relay Operations (FR31-FR37) | `cli/internal/relay/` + `cli/cmd/relay.go` | — |
| Port Validation (FR38-FR39) | `cli/internal/tunnel/validate.go` | — |
| Telemetry (FR40-FR43) | `cli/internal/telemetry/` | — |
| Configuration (FR44-FR46) | `cli/internal/config/` + `cli/cmd/config.go` | — |

### Data Flow

```
Host's localhost:PORT
       ↕ (HTTP/WebSocket)
  cli/internal/tunnel/proxy.go
       ↕ (protobuf over libp2p streams)
  cli/internal/p2p/ ←→ viewer/src/p2p/
       ↕ (protobuf deserialization)
  viewer/src/proxy/service-worker.ts
       ↕ (fetch response / MessageChannel)
  Tunneled content in browser
```

### Development Workflow Integration

**Development Servers:**
- CLI: `cd cli && go run . serve --port 8080`
- Viewer: `cd viewer && npm run dev` (Vite HMR on localhost:5173)
- Proto codegen: `buf generate` from project root (outputs to `cli/internal/proto/` and `viewer/src/proto/`)

**Build Process:**
- CLI: `cd cli && go build -o peertunnel .` (single static binary)
- Viewer: `cd viewer && npm run build` (static files in `viewer/dist/`)
- Proto: `buf generate` must run before either build if `.proto` files changed

**Deployment:**
- CLI binaries: GoReleaser attaches to GitHub Releases on tag push
- Viewer: GitHub Actions builds and deploys `viewer/dist/` to GitHub Pages
- Both triggered independently — viewer deploys on main branch push, CLI releases on version tags
