---
stepsCompleted: [1, 2, 3]
inputDocuments: ['product-brief-p2p-tunnel.md', 'prd.md', 'architecture.md', 'prd-validation-report.md']
workflowType: 'ux-design'
project_name: 'peertunnel'
user_name: 'Root'
date: '2026-04-13'
---

# UX Design Specification peertunnel

**Author:** Root
**Date:** 2026-04-13

---

<!-- UX design content will be appended sequentially through collaborative workflow steps -->

## Executive Summary

### Project Vision

peertunnel is a decentralized localhost tunneling tool with two components: a Go CLI that exposes any local HTTP port over libp2p peer-to-peer connections, and a static web viewer that lets anyone access the tunneled application from any browser with zero install. The product's UX north star is **one command, one link, zero friction** — the host runs a single CLI command, shares a single link, and the viewer clicks it and is inside the application. No accounts, no third-party servers, no software to install on the receiving end.

The zero-install browser viewer is the key innovation and the primary UX surface. No other P2P tunneling tool has solved the "one end is just a browser" problem. The viewer must handle the full lifecycle — from welcome/discovery through connection establishment to full-page content rendering — while feeling effortless.

### Target Users

**Primary: Development teams sharing self-hosted tools**
Developers running tools like TTYD (web terminals), VS Code Server, or other web-based development environments locally, sharing interactive access with remote colleagues for pairing, review, and QA sessions lasting hours. These users are technically proficient, expect CLI tools to be predictable, and value transparency in networking behavior.

**Secondary: Solo developers and demo-givers**
Developers showing in-progress work to clients or stakeholders. Speed of setup matters — they need to go from "running locally" to "shareable link" in seconds. The viewer recipient may be less technical.

**Tertiary: Relay operators**
Team leads or infrastructure-minded developers running private relay nodes for corporate environments. Comfortable with server administration and CLI configuration.

**Ambient: New visitors**
Developers discovering peertunnel by landing on the viewer URL directly. The welcome page must explain the product and provide a manual connection path.

### Key Design Challenges

1. **The disappearing UI** — The viewer shell (welcome page, connection logs, status indicators) must be present and informative during connection, then completely vanish during full-page takeover of tunneled content. It must reappear instantly and clearly on disconnect, error, or tunnel closure. The transition between "peertunnel UI" and "tunneled content" is the most critical UX moment.

2. **Connection transparency without intimidation** — Real-time connection logs showing peer resolution, transport negotiation, NAT hole-punching, and relay fallback must feel trustworthy and informative to both technical and non-technical viewers. P2P networking jargon needs to be surfaced honestly but accessibly.

3. **Dual entry points, unified experience** — Auto-connect via URL fragment and manual connect via the welcome page's input field must feel seamless. The welcome/idle state serves triple duty: landing page for new visitors, manual connect interface, and idle state between sessions.

4. **CLI output as UX** — The terminal is a first-class interface. Share link generation, live connection status, and session summary must be scannable, copy-friendly, and informative without being noisy.

### Design Opportunities

1. **Trust through transparency** — Honest connection status indicators (direct P2P vs relayed, real-time connection logs) can differentiate peertunnel from tools that hide networking details. Transparency builds user confidence and makes the P2P architecture a visible feature.

2. **The viral "one link" moment** — The CLI's share link output is the product's growth loop. Making the link output prominent, instantly copyable, and satisfying to share is a high-leverage UX win.

3. **Graceful degradation as polish** — Silent reconnection under 3 seconds, visible reconnection indicator over 3 seconds, and the explicit "tunnel closed by host" notification are opportunities to feel intentional and reliable where competitors feel broken or ambiguous.

4. **Zero-chrome immersion** — Full-page takeover means the tunneled app gets 100% of the viewport. No iframe borders, no persistent toolbars, no "you're using a tunnel" visual noise. The best UX is the one you don't notice.

## Core User Experience

### Defining Experience

peertunnel's core experience is a **two-sided, asymmetric interaction**:

**Host side (CLI):** The developer runs a single command and gets a shareable link. The terminal becomes a live dashboard showing who's connected and how. The core action is `peertunnel serve --port <port>` — everything flows from that moment.

**Viewer side (Web):** The receiver clicks a link and lands inside a working web application. No install, no account, no configuration. The connection establishment (peer resolution, transport negotiation, authentication) happens in seconds with real-time status feedback, then the viewer shell vanishes and the tunneled content takes over completely.

The **viewer's first connection** is the product's defining UX moment. If a non-technical colleague can click a link and be inside a VS Code Server session within 3 seconds, peertunnel has delivered its promise.

### Platform Strategy

**CLI (Host):**
- Terminal-based, keyboard-only interaction
- Linux, macOS, Windows (amd64 + arm64)
- Human-readable output designed for scannability
- No GUI, no TUI — plain text with intentional formatting

**Web Viewer (Receiver):**
- 100% static web application, zero backend
- Primary: Chromium browsers (Chrome, Edge, Opera) + Firefox via WebTransport
- Fallback: Safari via WebRTC-Direct
- Responsive viewer shell (welcome page, connection logs, reconnection UI) — mobile-friendly
- Tunneled content viewport is 100% of the browser window (full-page takeover)
- No offline mode — the product is inherently network-dependent

**Platform interaction model:**
- CLI: keyboard input, text output
- Viewer: mouse/touch for welcome page and manual connect; once tunneled content loads, interaction model is entirely determined by the tunneled application

### Effortless Interactions

1. **Share link generation** — The CLI outputs the link immediately on startup. No separate "generate" step. The link is the most prominent element in the startup output.

2. **Auto-connect on link click** — When the URL fragment contains connection details, the viewer begins connecting immediately. No interstitial, no "click to connect" button. The user's intent is clear from the URL.

3. **Silent reconnection** — Network interruptions under 3 seconds are completely invisible to the viewer. No flicker, no indicator, no state change. The session continues as if nothing happened.

4. **Full-page takeover** — Once the tunnel is established, the viewer shell disappears entirely. The tunneled application owns 100% of the viewport. No toolbar, no frame, no "powered by peertunnel" badge.

5. **Transparent fallback** — If direct P2P fails, relay fallback activates automatically. The viewer sees the connection log update ("Direct connection failed. Attempting relay fallback...") but takes no action. The system handles it.

6. **Clean endings** — When the host closes the tunnel, the viewer immediately sees "Tunnel closed by host." When the viewer closes their tab, the host sees the connection count decrement. Every ending is explicit.

### Critical Success Moments

1. **The "it just works" moment** — Viewer clicks a shared link, sees brief connection logs tick by, and within 2-3 seconds the tunneled application fills their browser. This is the moment that converts curiosity into adoption.

2. **The "zero setup" realization** — The viewer realizes they didn't install anything, didn't create an account, didn't configure anything. The link was the entire experience. This is what makes peertunnel shareable — there's nothing to explain to the recipient.

3. **The "it stayed up" trust moment** — After hours of sustained use (pairing, code review, demo), the tunnel is still stable. No session expiry warnings, no bandwidth cap notifications. This is where peertunnel earns repeat usage.

4. **The "I know what happened" clarity moment** — On any state change, the user knows exactly why. "Connected via relay" vs "Connected (direct P2P)." "Reconnecting..." vs "Tunnel closed by host." No ambiguity, no guessing.

5. **The "one command" host moment** — The developer types one command, sees the share link appear, copies it, and shares it. The CLI didn't ask questions, didn't require flags beyond the port, didn't need configuration. It just worked.

### Experience Principles

1. **Invisible when working** — The best tunnel experience is one where you forget you're using a tunnel. Full-page takeover, silent reconnection, no persistent UI chrome. When everything is working, peertunnel should be completely invisible.

2. **Honest when not** — When the system state changes — connecting, reconnecting, falling back to relay, disconnecting — tell the user exactly what's happening with clear, specific language. No generic spinners. No vague error messages. Transparency builds trust.

3. **One step, not two** — Every user action should feel like a single step. One command to serve. One link to share. One click to connect. If the system can do something automatically (connect, fall back, reconnect), it should. Never ask the user to do what the system can decide.

4. **Terminal is a first-class UI** — CLI output is intentionally designed. The share link is visually prominent. Connection events are timestamped and scannable. The session summary is useful. The terminal is not a debug log — it's the host's interface.

5. **Respect the viewer's context** — The viewer may be less technical than the host. Connection logs should be informative without being intimidating. Error states should be clear without being technical. The viewer didn't choose to use peertunnel — they clicked a link from a colleague.
