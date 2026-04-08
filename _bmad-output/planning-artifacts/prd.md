---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation']
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

## Executive Summary

peertunnel is a decentralized localhost tunneling tool that enables developers to share self-hosted web applications with remote colleagues over direct peer-to-peer connections. A Go CLI exposes any local HTTP port over libp2p, and a static web viewer — hosted at a public URL — lets the receiver access the tunneled application from any browser with zero install. One command to serve, one link to share, no accounts, no third-party servers in the data path.

The primary use case is collaborative remote development: running self-hosted tools like TTYD (web terminals), VS Code Server, or other web-based development environments locally and giving colleagues direct, interactive access. Existing solutions (ngrok, Cloudflare Tunnel, bore) either impose bandwidth caps and session limits that break sustained work sessions, or route all interactive traffic through third-party infrastructure — adding latency, cost pressure, and privacy concerns where none need to exist.

### What Makes This Special

The zero-install browser viewer is the key innovation. No other P2P tunneling tool has solved the "one end is just a browser" problem. The receiver clicks a link and is inside the locally-hosted application — interacting with a terminal, an IDE, or any web app live — without installing anything. Combined with direct P2P connectivity (relay only as fallback for restrictive networks), this eliminates the entire class of problems caused by relay-dependent architectures: no degrading free tiers, no bandwidth bills that scale with usage, no session expirations mid-pairing-session.

The timing is right: libp2p browser transports (WebTransport, WebRTC-Direct) reached production maturity in 2025, ngrok's free tier continues to shrink, and no existing tool combines P2P tunneling with a zero-install browser viewer for sustained collaborative use.

## Project Classification

- **Project Type:** CLI Tool + Web App (hybrid — Go CLI as primary developer interface, Lit/TypeScript static web viewer as receiver interface)
- **Domain:** Developer Tools / Networking
- **Complexity:** Medium (libp2p P2P networking, NAT hole-punching, Service Worker proxying, multi-transport browser support — no regulatory concerns)
- **Project Context:** Greenfield

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

### MVP - Minimum Viable Product

- **CLI (Go):** Single HTTP port forwarding, shareable link generation, libp2p networking with P2P-first + relay fallback, auto-reconnection logic, multi-viewer support (multiple simultaneous browser connections)
- **Web Viewer (Lit/TS):** Static app, token auth, Service Worker-based content proxying, full SPA support, WebSocket proxying (critical for TTYD/VS Code Server), reconnection UI (silent <3s, visible indicator >3s)
- **Transports:** WebTransport + WebRTC-Direct fallback
- **Telemetry:** Opt-in anonymous usage tracking with local random ID, first-run prompt, disable option
- **Hosting:** Viewer deployed at a public URL
- **Docs:** Getting-started guide, README

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

**Rising Action:** They deploy a standard libp2p Circuit Relay v2 server — an existing open-source implementation from go-libp2p. Configuration is minimal: set the relay address, optionally restrict access. Documentation references the upstream go-libp2p relay setup.

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
| Relay operator | Standard libp2p Circuit Relay v2, self-hostable, configurable relay address in CLI |
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

### Risk Mitigation

- **Transport fallback is simple and predictable** — two steps only (direct → relay), no complex negotiation chain.
- **Relay is a known quantity** — libp2p Circuit Relay v2 is battle-tested across 30+ networks. Community relays + self-hosted option ensures fallback availability.
