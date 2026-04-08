---
stepsCompleted: [1, 2]
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
