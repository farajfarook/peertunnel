---
stepsCompleted: [1, 2]
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
