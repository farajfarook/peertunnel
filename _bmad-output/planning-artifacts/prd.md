---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
inputDocuments: ['product-brief-p2p-tunnel.md']
workflowType: 'prd'
documentCounts:
  briefs: 1
  research: 0
  brainstorming: 0
  projectDocs: 0
classification:
  projectType: cli_tool + web_app
  domain: developer_tools_networking
  complexity: medium
  projectContext: greenfield
---

# Product Requirements Document - peertunnel

**Author:** Faraj
**Date:** 2026-04-08
**Project Type:** CLI Tool + Web App (Go CLI + Lit/TypeScript static web viewer)
**Domain:** Developer Tools / Networking
**Complexity:** Medium | **Context:** Greenfield

## Executive Summary

peertunnel is a decentralized localhost tunneling tool that enables developers to share self-hosted web applications with remote colleagues over direct peer-to-peer connections. A Go CLI exposes any local HTTP port over libp2p, and a static web viewer — hosted at a public URL — lets the receiver access the tunneled application from any browser with zero install. One command to serve, one link to share, no accounts, no third-party servers in the data path.

The primary use case is collaborative remote development: running self-hosted tools like TTYD (web terminals), VS Code Server, or other web-based development environments locally and giving colleagues direct, interactive access. Existing solutions (ngrok, Cloudflare Tunnel, bore) either impose bandwidth caps and session limits that break sustained work sessions, or route all interactive traffic through third-party infrastructure — adding latency, cost pressure, and privacy concerns where none need to exist.

### What Makes This Special

The zero-install browser viewer is the key innovation. No other P2P tunneling tool has solved the "one end is just a browser" problem. The receiver clicks a link and is inside the locally-hosted application — interacting with a terminal, an IDE, or any web app live — without installing anything. Combined with direct P2P connectivity (relay only as fallback for restrictive networks), this eliminates the entire class of problems caused by relay-dependent architectures: no degrading free tiers, no bandwidth bills that scale with usage, no session expirations mid-pairing-session.

libp2p browser transports (WebTransport, WebRTC-Direct) reached production maturity in 2025, ngrok's free tier continues to shrink, and no existing tool combines P2P tunneling with a zero-install browser viewer for sustained collaborative use.

## Success Criteria

### User Success

- A developer can expose a local web application with a single command and share it via a single link in under 30 seconds
- Colleagues can access shared self-hosted tools (TTYD, VS Code Server, etc.) from a browser with zero install and full interactivity including WebSocket support
- Tunnels remain stable for 5+ hour collaborative sessions
- Disconnections under 3 seconds reconnect silently; longer disconnections show a "reconnecting..." indicator in the viewer
- Users feel zero friction: no accounts, no configuration, no third-party dependency awareness

### Business Success

- 1,000+ unique CLI installs within 6 months of launch
- Repeat usage: 20%+ of opt-in telemetry users run 5+ sessions in their first month
- Active community contributors (PRs, issues, discussions) growing month-over-month
- The tool is genuinely useful — developers adopt it for real collaborative work, not just novelty

### Technical Success

- Direct P2P connection success rate >80%; >95% including relay fallback
- Sub-second request latency after initial connection establishment
- Full WebSocket support for interactive tools (TTYD, VS Code Server)
- Service Worker proxying handles complex SPAs including client-side routing, API calls, and WebSocket connections
- Opt-in anonymous telemetry (local random ID, no personal data, user-disableable) provides aggregate usage metrics

### Measurable Outcomes

| Metric | Target | Timeframe |
|---|---|---|
| Unique CLI installs | 1,000+ | 6 months |
| Repeat session rate (opt-in) | 20%+ run 5+ sessions/month | 6 months |
| P2P direct connection rate | >80% | Launch |
| Connection + relay success rate | >95% | Launch |
| Tunnel stability | 5+ hours sustained | Launch |
| Request latency (post-connect) | <1 second | Launch |

## Product Scope

### MVP Strategy

**Approach:** Problem-solving MVP — deliver the core tunneling experience end-to-end. If a developer can run one command, share one link, and have a colleague interact with their self-hosted web app live in a browser over P2P, the product works.

**Resource:** Solo developer with AI tooling assistance. Feasible given heavy reliance on libp2p's existing infrastructure.

### MVP Capabilities

- **CLI `serve`:** Single HTTP port forwarding, share link generation, multi-viewer (default 5, configurable), auto-reconnect, live connection count/list, port validation (soft warning), session summary on close
- **CLI `relay`:** Built-in relay mode with configurable limits (max connections, max duration, max bandwidth per peer), optional peer allowlist, peertunnel-only protocol negotiation
- **Web Viewer:** Token auth, Service Worker transparent proxy (all HTTP + WebSocket traffic), connection logs with real-time status, welcome/idle page with manual connect input, reconnection UI (silent <3s, visible >3s), "tunnel closed by host" notification, mobile-friendly shell
- **Transports:** WebTransport (Chromium, Firefox) + WebRTC-Direct (Safari)
- **Networking:** libp2p with Noise encryption, NAT hole-punching via DCUtR, peer discovery via IPFS/libp2p bootstrap nodes, Circuit Relay v2 fallback
- **Telemetry:** Opt-in anonymous tracking with local random ID, first-run prompt, disable option
- **Infrastructure:** 1-2 community relay instances on VPS, hardcoded as defaults
- **Config:** `~/.peertunnel/config.yaml` for persistent settings (telemetry ID, opt-in)
- **Docs:** README, getting-started guide
- **SEO:** Essential meta tags and Open Graph on viewer welcome page

### Explicitly Deferred (Post-MVP)

- Shell completion
- HTTPS on local side
- JSON output mode
- Multi-port forwarding
- Named/persistent tunnels
- Custom viewer domains
- Peer ID allowlists on `serve` mode
- Telemetry dashboard

### Risk Mitigation

**Technical Risks:**
- Service Worker interception coverage — the proxy is transparent; all HTTP and WebSocket traffic routes through the P2P channel. The web app doesn't know it's being served over P2P. Risk is in edge cases (specific browser behaviors, exotic request types), mitigated by cross-browser testing.
- P2P connection reliability — mitigated by two-step fallback (direct → relay) and automated NAT simulation testing in CI.
- Sustained session stability (5+ hours) — mitigated by auto-reconnection logic and relay fallback.

**Market Risks:**
- "Does anyone actually need this?" — mitigated by solving a real pain point the developer already experiences (sharing TTYD/VS Code Server with colleagues). The developer IS the first user.
- Discoverability — mitigated by essential SEO, open source presence, and a distinctive name (peertunnel).

**Resource Risks:**
- Solo developer with AI tooling. If scope needs cutting, relay mode is the most independent feature — community relays could launch after the core serve/viewer flow works.

## User Journeys

### Journey 1: Ravi — The Tunnel Host (Happy Path)

**Situation:** Ravi is a backend developer running a self-hosted VS Code Server on his laptop. His colleague Priya needs to review his API changes, but she's remote and he doesn't want to deploy to staging for a quick review.

**Opening Scene:** Ravi has VS Code Server running on port 8080. He opens his terminal and types `peertunnel serve --port 8080`.

**Rising Action:** The CLI starts up, outputs connection details:
```
peertunnel v1.0
Exposing localhost:8080
Peer ID: 12D3KooW...
Share link: https://viewer.peertunnel.dev/#12D3KooW....<token>
Waiting for connections...
```
He copies the link and pastes it in Slack to Priya. The CLI sits ready, showing no active connections yet.

**Climax:** Priya connects. The CLI updates:
```
[14:32:01] Viewer connected (direct P2P) — 1 active connection
```
A second colleague, Amir, joins too:
```
[14:33:15] Viewer connected (direct P2P) — 2 active connections
```
Ravi sees the live connection count and knows his team is in. They pair for 3 hours straight on his local VS Code Server — no disconnections, no bandwidth warnings.

**Resolution:** When they're done, viewers disconnect and Ravi hits Ctrl+C. The CLI shows a session summary. His local environment was never exposed to any third party.

### Journey 2: Priya — The Viewer/Collaborator (Happy Path)

**Situation:** Priya gets a link from Ravi in Slack. She needs to review his API code in his VS Code Server.

**Opening Scene:** Priya clicks the link. Her browser opens the peertunnel viewer.

**Rising Action:** The viewer shows transparent connection logs:
```
Resolving peer...
Attempting direct connection (WebTransport)...
NAT hole-punch in progress...
Connected! (direct P2P)
```
Each step updates in real time — she knows exactly what's happening.

**Climax:** The connection establishes in under 2 seconds. The viewer shell disappears and Ravi's VS Code Server fills her browser. She's inside his IDE — full interactivity, WebSocket-driven, no lag. She navigates files, leaves comments, runs the terminal. It feels local.

**Resolution:** After the review session, she closes the tab. Nothing was installed, no account was created, no data touched a third party's servers.

### Journey 3: Priya — The Viewer (Edge Case: Restrictive Network)

**Situation:** Priya is working from a coffee shop with restrictive WiFi. Direct P2P connection fails.

**Opening Scene:** She clicks the same link from Ravi.

**Rising Action:** The viewer logs show the connection attempt:
```
Resolving peer...
Attempting direct connection (WebTransport)...
Direct connection failed.
Attempting relay fallback...
Connected via relay.
```
The logs are honest — she sees it fell back to relay.

**Climax:** The tunneled app loads. It works. Slightly higher latency than direct P2P, but fully functional. The viewer shows a subtle indicator that this is a relayed connection.

**Resolution:** When she gets back to her home network later and clicks the same link, it connects directly. The viewer logs confirm direct P2P. No action needed from Ravi — same link, better connection.

### Journey 4: Ravi — The Host (Edge Case: Disconnection & Reconnection)

**Situation:** Ravi's WiFi drops briefly during a pairing session with two connected viewers.

**Opening Scene:** The tunnel has been running for 2 hours. Two viewers are connected.

**Rising Action:** Ravi's network drops for 2 seconds. The CLI shows:
```
[16:45:03] Connection interrupted. Reconnecting...
[16:45:04] Reconnected.
```
On the viewer side, the disconnection is under 3 seconds — viewers see nothing. The session continues seamlessly.

**Alternative:** If the drop lasts 5 seconds, viewers see:
```
Reconnecting... (4s)
```
The indicator disappears when the connection restores. No data is lost, the session resumes.

**Resolution:** The pairing session continues uninterrupted. The auto-reconnection handled it.

### Journey 5: Ravi — The Host (Edge Case: Tunnel Closed)

**Situation:** Ravi stops the CLI while Priya is still connected.

**Opening Scene:** Ravi hits Ctrl+C or closes his terminal.

**Rising Action:** The CLI gracefully shuts down:
```
Closing tunnel... Notifying 1 active viewer(s).
Tunnel closed. Session duration: 2h 14m.
```

**Climax:** On Priya's side, the viewer immediately shows:
```
Tunnel closed by host.
```
Clear, unambiguous. Not a vague "connection lost" — she knows the host ended it intentionally.

### Journey 6: Relay Operator — Community/Self-Hosted

**Situation:** A team lead wants to run a private relay for their company's developers working behind corporate firewalls.

**Opening Scene:** They need reliable fallback connectivity for their distributed team. The public community relays work but they want something under their control.

**Rising Action:** They install peertunnel on a VPS and run it in relay mode:
```
peertunnel relay --port 4001 --max-connections 50 --max-duration 6h
```
Configuration is minimal. Optionally restrict access with `--allowed-peers`.

**Climax:** Team members configure their CLI to use the private relay as a preferred fallback:
```
peertunnel serve --port 3000 --relay /ip4/relay.company.com/tcp/4001/p2p/12D3KooW...
```
Direct P2P is still attempted first. The private relay kicks in only when direct fails.

**Resolution:** The team has reliable tunneling even behind the strictest corporate firewalls, with traffic only touching infrastructure they control. The community default relays remain available for developers without a private relay.

### Journey 7: New Visitor — Viewer Landing Page

**Situation:** A developer hears about peertunnel and visits the viewer URL directly out of curiosity, or a colleague sends them a peer ID over a phone call instead of the full link.

**Opening Scene:** They navigate to `https://viewer.peertunnel.dev` — no fragment in the URL.

**Rising Action:** The viewer shows a clean welcome state:
- Brief intro: what peertunnel is and how it works (2-3 sentences)
- An input field to paste a connection string (peer ID + token)
- A "Connect" button

**Climax:** They paste the connection string their colleague gave them, hit Connect. The viewer transitions to the connection log state:
```
Resolving peer...
Attempting direct connection (WebTransport)...
Connected! (direct P2P)
```
The tunneled app loads.

**Resolution:** Same viewer, two entry points — auto-connect via URL fragment, or manual entry via the welcome state. No separate pages, no navigation. The welcome state is the idle state.

### Journey Requirements Summary

| Journey | Capabilities Revealed |
|---|---|
| Host happy path | CLI port forwarding, link generation, live connection count/list, session summary |
| Viewer happy path | Viewer connection with transparent logs, Service Worker proxying, WebSocket support, zero-install |
| Viewer restrictive network | Relay fallback, connection type indicator, honest status logging |
| Host disconnection | Auto-reconnection (<3s silent, >3s visible), connection resilience |
| Host tunnel close | Graceful shutdown, viewer notification ("tunnel closed by host") |
| Relay operator | `peertunnel relay` mode, self-hostable, configurable relay address in CLI |
| New visitor landing | Viewer welcome/idle state, manual connection input, brief project intro, same-page UX |

## Innovation & Novel Patterns

### Detected Innovation Areas

**Browser-native P2P receiver — the zero-install viewer.** Existing libp2p tunneling tools (portforward-over-libp2p, go-p2ptunnel, Hyprspace) all require software installation on both ends. peertunnel is the first to combine libp2p tunneling with a browser-based receiver that requires nothing from the viewing party — just a link click. This is enabled by libp2p's WebTransport and WebRTC-Direct browser transports reaching production maturity in 2025.

**Service Worker as P2P proxy layer.** Using a Service Worker to intercept all sub-resource requests from tunneled content and route them through the P2P channel is a novel application of Service Worker technology — turning a static web page into a full-fidelity proxy for complex web applications including SPAs, WebSocket-driven tools, and client-side routing.

### Protocol & Validation Design

- **Local protocol: HTTP only (MVP).** Most self-hosted dev tools (TTYD, VS Code Server) serve plain HTTP on localhost. HTTPS on the local side adds TLS certificate complexity with no security benefit — the P2P tunnel is already end-to-end encrypted via libp2p Noise protocol.
- **Port validation:** CLI performs a soft HTTP check against `localhost:<port>` before tunneling. If nothing is listening or the response isn't HTTP, the CLI warns but does not block — the user may start the server after the tunnel.
- **WebSocket support:** Native proxying of WebSocket upgrade requests through the P2P channel. MVP-critical for TTYD, VS Code Server, and similar interactive tools.

### Market Context & Competitive Landscape

| Tool | P2P | Browser Receiver | No Account | Open Source |
|---|---|---|---|---|
| peertunnel | Yes (libp2p) | Yes (zero-install) | Yes | Yes |
| portforward-over-libp2p | Yes (libp2p) | No (CLI both ends) | Yes | Yes |
| go-p2ptunnel / p2ptunnel | Yes (libp2p) | No (CLI both ends) | Yes | Yes |
| Hyprspace | Yes (libp2p/IPFS) | No (native client) | Yes | Yes |
| ngrok | No (relay) | Yes (URL) | No | No |
| Cloudflare Tunnel | No (relay) | Yes (URL) | No | No |
| bore | No (relay) | No (CLI) | Yes | Yes |

No existing tool occupies the "P2P + browser receiver" quadrant.

### Validation Approach

- **Automated NAT simulation testing** — Docker containers with `iptables` rules simulating full cone, restricted, and symmetric NAT types. CI-friendly, repeatable tests for P2P connection scenarios across network topologies.
- **Two-step transport validation** — direct (WebTransport/WebRTC-Direct) → relay fallback. Test both paths independently and the failover between them.
- **Cross-browser testing** — WebTransport (Chrome, Firefox, Edge, Opera) and WebRTC-Direct (Safari fallback) coverage.

## Technical Specifications

### CLI — Command Structure

```
peertunnel serve --port <port> [--relay <multiaddr>] [--max-viewers <n>]
peertunnel relay [--port <port>] [--max-connections <n>] [--max-duration <duration>] [--max-bandwidth-per-peer <limit>] [--allowed-peers <peer-id,...>]
peertunnel version
peertunnel config [--reset-telemetry]
```

- `serve` — exposes localhost port, generates share link, shows live connection status.
- `relay` — runs as a Circuit Relay v2 node with relaxed resource limits for sustained tunneling. Same binary, different mode.
- `version` — prints version info.
- `config` — manages persistent configuration (telemetry ID, opt-in status).

**Serve mode flags:**
- `--port` — local HTTP port to expose (required)
- `--relay` — custom relay multiaddr for fallback connectivity
- `--max-viewers` — max simultaneous viewer connections (default 5)

**Relay mode flags:**
- `--port` — listen port (default 4001)
- `--max-connections` — hard cap on simultaneous relay sessions (default 50)
- `--max-duration` — max session duration (default 6h)
- `--max-bandwidth-per-peer` — per-peer bandwidth cap
- `--allowed-peers` — optional peer ID allowlist for private relays. Open by default.

**Output:** Human-readable only for MVP. No `--json` mode.

**Configuration persistence:**
- Per-session values (port, relay address, allowed-peers, max-viewers) are CLI flags.
- Persistent values (telemetry local ID, opt-in status) live in `~/.peertunnel/config.yaml`.

### CLI — Output Formats

**Tunnel startup:**
```
peertunnel v1.0
Exposing localhost:8080
Peer ID: 12D3KooW...
Share link: https://viewer.peertunnel.dev/#12D3KooW....<token>
Waiting for connections...
```

**Connection events:**
```
[14:32:01] Viewer connected (direct P2P) — 1 active connection
[14:33:15] Viewer connected (direct P2P) — 2 active connections
[14:45:03] Viewer disconnected — 1 active connection
```

**Shutdown:**
```
Closing tunnel... Notifying 2 active viewer(s).
Tunnel closed. Session duration: 2h 14m.
```

**Port validation warning:**
```
Warning: No HTTP server detected on localhost:8080. Tunnel will start anyway.
```

**Relay startup:**
```
peertunnel relay v1.0
Listening on /ip4/0.0.0.0/tcp/4001
Relay Peer ID: 12D3KooW...
Max connections: 50 | Max duration: 6h
Access: open
Waiting for relay requests...
```

### Relay Network Architecture

- **Community relays:** 1-2 peertunnel relay instances run on VPS by the project. Their multiaddrs are hardcoded as defaults in the CLI. Users get relay fallback out of the box.
- **Self-hosted relays:** Anyone with a VPS runs `peertunnel relay` — one command, same binary. Teams can restrict access via `--allowed-peers`.
- **Security:** libp2p handles all relay authentication automatically. Every relay has a peer ID derived from its keypair. Connecting clients verify the relay's identity via the Noise protocol. The peer ID in the multiaddr (`/p2p/12D3KooW...`) IS the public key identity — connections fail if the server doesn't hold the matching private key.
- **Abuse prevention:** Rate limiting per peer, max session duration, max total connections, peertunnel-only protocol negotiation.
- **Discovery/hole-punching:** Leverages existing IPFS/libp2p bootstrap infrastructure at `bootstrap.libp2p.io` for peer discovery and NAT traversal (DCUtR). No custom discovery infrastructure needed.

### Web Viewer — Browser Matrix

| Browser | Transport | Support Level |
|---|---|---|
| Chrome (stable) | WebTransport | Primary |
| Edge (stable) | WebTransport | Primary |
| Opera (stable) | WebTransport | Primary |
| Firefox (stable) | WebTransport | Primary |
| Safari (stable) | WebRTC-Direct | Fallback transport |

Minimum target: current stable versions of any Chromium-based browser and Safari.

### Web Viewer — Design Requirements

- **Responsive:** Viewer shell (welcome page, connection logs, reconnection UI) is mobile-friendly. Tunneled content responsiveness is the host app's responsibility.
- **Accessibility:** Semantic HTML, keyboard navigable inputs, screen reader compatible status messages, sufficient color contrast.
- **SEO:** Descriptive `<title>` and `<meta description>`, Open Graph tags for link previews, basic structured data on the welcome page.

### Implementation Considerations

- CLI is a single Go binary with zero external dependencies
- Viewer is a 100% static site (Lit + Vite build) deployable to any static host
- No backend infrastructure for either component (relay is optional infrastructure)
- Config file created on first run with telemetry opt-in prompt
- Relay mode uses go-libp2p's Circuit Relay v2 with configurable resource limits

## Functional Requirements

### Tunnel Hosting

- **FR1:** Host can expose a single local HTTP port as a P2P tunnel with one command
- **FR2:** Host can receive a shareable link containing peer ID and auth token upon tunnel start
- **FR3:** Host can view a live count of active viewer connections
- **FR4:** Host can view a list of active viewer connections with connection type (direct/relay)
- **FR5:** Host can receive a session summary (duration, connection count) on tunnel close
- **FR6:** Host can gracefully shut down the tunnel with notification sent to all connected viewers
- **FR7:** Host can specify a custom relay address for fallback connectivity
- **FR8:** Host can auto-reconnect after a network interruption without restarting the tunnel

### Tunnel Viewing

- **FR9:** Viewer can access a tunneled web application by clicking a shared link in any supported browser
- **FR10:** Viewer can see real-time connection status logs during connection establishment
- **FR11:** Viewer can interact with the tunneled web application with full HTTP and WebSocket support
- **FR12:** Viewer can experience silent reconnection for network interruptions under 3 seconds
- **FR13:** Viewer can see a "reconnecting..." indicator for interruptions longer than 3 seconds
- **FR14:** Viewer can see a "tunnel closed by host" message when the host ends the session
- **FR15:** Viewer can see a connection type indicator (direct P2P vs relayed)
- **FR16:** Multiple viewers can connect to the same tunnel simultaneously

### Viewer Welcome & Manual Connect

- **FR17:** Visitor can see a welcome page with a brief description of peertunnel when accessing the viewer URL without a connection string
- **FR18:** Visitor can manually enter a connection string (peer ID + token) and connect via an input field
- **FR19:** Viewer auto-connects when the URL contains a connection string in the fragment

### P2P Networking

- **FR20:** System can establish direct P2P connections via WebTransport as the primary transport
- **FR21:** System can fall back to WebRTC-Direct for browsers without WebTransport support (Safari)
- **FR22:** System can fall back to Circuit Relay v2 when direct P2P connection fails
- **FR23:** System can perform NAT hole-punching via DCUtR for direct connectivity
- **FR24:** System can discover peers via IPFS/libp2p bootstrap infrastructure
- **FR25:** System can encrypt all traffic end-to-end via libp2p Noise protocol
- **FR26:** System can authenticate connections using a token bundled in the share link

### Content Proxying

- **FR27:** Service Worker can transparently intercept and proxy all HTTP requests from tunneled content through the P2P channel
- **FR28:** Service Worker can transparently proxy WebSocket connections through the P2P channel
- **FR29:** Service Worker can handle sub-resource loading (JS, CSS, images, fonts) through the P2P channel
- **FR30:** Service Worker can support client-side routing in tunneled SPAs

### Relay Operations

- **FR31:** Operator can run a relay node using the same peertunnel binary with a `relay` subcommand
- **FR32:** Operator can configure maximum simultaneous connections on the relay
- **FR33:** Operator can configure maximum session duration on the relay
- **FR34:** Operator can configure maximum bandwidth per peer on the relay
- **FR35:** Operator can restrict relay access to specific peer IDs via an allowlist
- **FR36:** Relay can reject non-peertunnel protocol traffic
- **FR37:** Relay can authenticate connecting peers via libp2p keypair verification

### Port Validation

- **FR38:** CLI can perform a soft HTTP check on the target localhost port before starting the tunnel
- **FR39:** CLI can warn (not block) when no HTTP server is detected on the target port

### Telemetry

- **FR40:** CLI can prompt the user to opt in or out of anonymous telemetry on first run
- **FR41:** CLI can generate and persist a local random ID for anonymous usage tracking
- **FR42:** User can disable telemetry at any time via CLI config
- **FR43:** Telemetry can collect aggregate session data (count, duration) without personal information

### Configuration

- **FR44:** CLI can persist settings (telemetry ID, opt-in status) in a config file at `~/.peertunnel/config.yaml`
- **FR45:** User can reset telemetry identity via a config subcommand
- **FR46:** Host can configure maximum simultaneous viewer connections via `--max-viewers` flag (default 5)

## Non-Functional Requirements

### Performance

- Tunnel connection establishment (direct P2P) completes in under 3 seconds
- Tunnel connection establishment (relay fallback) completes in under 5 seconds
- HTTP request latency through the tunnel adds less than 100ms overhead on top of network RTT
- WebSocket message latency through the tunnel adds less than 50ms overhead on top of network RTT
- Service Worker proxy adds less than 10ms overhead per intercepted request
- Tunnel supports sustained throughput sufficient for interactive web applications (VS Code Server, TTYD)

### Security

- All P2P traffic is end-to-end encrypted via libp2p Noise protocol — no plaintext data on the wire
- Token-based authentication prevents unauthorized viewers from connecting to a tunnel
- Relay nodes verify connecting peers via libp2p keypair authentication
- Relay nodes reject non-peertunnel protocol traffic
- No user data, session content, or traffic passes through any peertunnel-controlled infrastructure (except relay fallback, where traffic is encrypted end-to-end and the relay cannot inspect it)
- Telemetry collects zero personal information — only aggregate session counts and durations tied to a random local ID
- Config file permissions are set to user-only read/write (`600`)

### Scalability

- A single tunnel host supports up to 5 simultaneous viewer connections by default, configurable via `--max-viewers` flag
- A community relay instance on a modest VPS supports up to 50 concurrent relay sessions by default, configurable via `--max-connections` flag
- Self-hosted relay capacity is configurable via CLI flags
- No central server bottleneck — scaling is per-host and per-relay, not centralized

### Reliability

- Tunnels remain stable for 5+ continuous hours under normal network conditions
- Auto-reconnection restores the tunnel after network interruptions without user intervention
- Reconnections under 3 seconds are invisible to viewers
- Graceful degradation: if direct P2P fails, relay fallback activates automatically
- CLI exits cleanly on SIGINT/SIGTERM with viewer notification and session summary

### Compatibility

- CLI binary runs on Linux, macOS, and Windows (amd64 and arm64)
- CLI binary size does not exceed 200MB
- Web viewer works on current stable versions of all Chromium-based browsers and Safari
- Web viewer shell is responsive and functional on mobile devices
