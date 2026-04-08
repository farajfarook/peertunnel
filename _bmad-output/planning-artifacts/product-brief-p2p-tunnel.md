---
title: "Product Brief: p2p-tunnel"
status: "complete"
created: "2026-04-08"
updated: "2026-04-08"
inputs: [user brain dump, web research — competitive landscape and libp2p ecosystem]
---

# Product Brief: p2p-tunnel

## Executive Summary

Every developer has faced the same friction: you're building something locally, and you need someone else to see it — a teammate, a client, a stakeholder. Today's answer is tunneling tools like ngrok, but they all route your traffic through someone else's servers, cap your bandwidth, expire your sessions, and require accounts you didn't want to create.

**p2p-tunnel** is a decentralized localhost tunneling tool that cuts out the middleman. A Go CLI exposes any local HTTP port over a direct peer-to-peer connection using libp2p. A lightweight static web viewer — hosted at a public URL — connects to the CLI node from any browser, authenticates with a token, and renders the tunneled site live. Sharing is a single link. No account. No relay server. No bandwidth caps. No session limits.

The timing is right: ngrok is alienating developers with its enterprise pivot, libp2p browser transports reached production maturity in 2025, and no existing tool combines P2P tunneling with a zero-install browser viewer.

## The Problem

Developers exposing localhost to the outside world face a frustrating set of trade-offs:

- **ngrok** — the de facto standard — now caps free users at 1GB/month bandwidth and 2-hour sessions. URLs regenerate on restart. Frequent disconnections disrupt demos and testing. The DDEV project publicly reconsidered ngrok as their default in early 2026.
- **Cloudflare Tunnel** — free and unlimited, but locks you into Cloudflare's ecosystem, requires account setup and DNS configuration, and routes all traffic through their infrastructure. Privacy-conscious developers and regulated teams can't accept that — your traffic never leaves a third party's hands.
- **Self-hosted alternatives** (bore, frp) — eliminate the vendor dependency but introduce a new one: you need to run and maintain a relay server.
- **P2P VPN tools** (Tailscale Funnel, EdgeVPN) — require native clients on both ends, accounts, or network membership. Not designed for ad-hoc sharing.

The common thread: every existing solution either depends on a centralized relay, requires infrastructure, or demands more setup than the task warrants.

## The Solution

p2p-tunnel is two components:

**CLI Tool (Go)**
- Single binary, zero dependencies. Run `p2p-tunnel serve --port 3000` and you're live.
- Exposes any local HTTP port over libp2p's peer-to-peer network.
- Generates a shareable tunnel link — a URL to the hosted viewer with the peer ID and security token encoded in the fragment. One link to copy and share.
- Connects directly to the viewer's browser via WebTransport or WebRTC-Direct. P2P first — relay only as a last resort for restrictive networks (similar to Tailscale's DERP approach).

**Static Web Viewer (Lit + TypeScript + Vite)**
- A 100% static web application hosted at a public URL (domain TBD). Zero backend.
- The receiver clicks the shared link, the viewer extracts the connection details from the URL, authenticates, and the tunneled site renders live — full SPAs, self-hosted apps, the works.
- A Service Worker intercepts all sub-resource requests (CSS, JS, images, API calls, WebSocket connections) and routes them through the P2P tunnel, enabling full-fidelity rendering of complex web applications.
- Built with Lit (Web Components) for minimal footprint. The viewer itself is a thin shell around the tunneled content.

**The zero-install receiver is the key innovation.** No other P2P tool has solved the "one end is just a browser" problem. The person viewing your localhost installs nothing — they click a link and see your app.

## What Makes This Different

**Zero-knowledge privacy by architecture.** Your traffic flows directly between your machine and the viewer's browser. It never touches a third-party server. This isn't a privacy policy — it's a structural guarantee. libp2p's Noise protocol encrypts all traffic end-to-end.

**Sustainably free.** Every relay-based competitor faces an inexorable cost curve: more users means more server costs means aggressive monetization. ngrok's free tier has degraded every year since 2020. p2p-tunnel has no server bill that scales with usage. It will never need a degrading free tier because there's nothing to pay for.

**Single-link sharing.** The CLI outputs a link like `https://<viewer-domain>/#<peer-id>.<token>`. Share it in Slack, email, or a Post-it. The receiver clicks it and sees your localhost. No special setup on their end.

| | p2p-tunnel | ngrok | Cloudflare Tunnel | bore |
|---|---|---|---|---|
| Central server required | No | Yes | Yes | Yes |
| Account required | No | Yes | Yes | No |
| Bandwidth limits | None | 1GB/mo free | None | None |
| Session limits | None | 2 hours free | None | None |
| Receiver installs anything | No (browser only) | No (URL) | No (URL) | Yes (CLI) |
| Open source | Yes | No | No | Yes |

## Who This Serves

**Solo developers** — Building locally, need to show work to a client. One command, one link, done.

**Development teams** — Sharing in-progress work for review, QA, or pairing. Long-lived tunnels that don't expire mid-sprint.

**Demo-givers** — Showing a live local build to stakeholders without deploying to staging. The tunnel shouldn't be the thing that breaks during the demo.

## Technical Approach

- **libp2p** for P2P networking — production-proven across 30+ networks, mature Go and JS implementations.
- **WebTransport** as primary browser transport (Chrome, Firefox, Edge, Opera). **WebRTC-Direct** as fallback for Safari.
- **P2P first, relay last.** Direct connections via NAT hole-punching are always attempted first. libp2p Circuit Relay v2 is used only when direct connection fails (restrictive corporate firewalls, symmetric NAT). Same philosophy as Tailscale's DERP relays — the relay exists for reliability, not as the default path.
- **End-to-end encryption** via libp2p's Noise protocol. Traffic is encrypted between the CLI and the browser regardless of transport.
- **Token-based authentication** — CLI generates a token bundled with the peer ID into the shareable link. The viewer extracts and validates it before the tunnel opens.
- **Service Worker proxying** — the viewer registers a Service Worker that intercepts all HTTP requests from the tunneled content and routes them through the P2P channel, enabling full SPA support including client-side routing, API calls, and WebSocket connections.

## Success Criteria

- **Adoption:** 500+ GitHub stars and 1,000+ unique CLI installs within 6 months of launch.
- **Reliability:** >80% direct P2P connection success rate. >95% including relay fallback.
- **Performance:** Sub-second request latency after initial connection. Stable tunnels over hours/days.
- **Developer experience:** One command to serve. One link to share. Zero accounts.

## Scope

**v1 — In scope:**
- CLI: single HTTP port forwarding, shareable link generation, libp2p networking (P2P first, relay fallback)
- Web viewer: static Lit app, token auth, Service Worker-based content proxying, full SPA support
- Transports: WebTransport + WebRTC-Direct
- Hosted viewer at a public URL
- Documentation and getting-started guide

**v1 — Explicitly out of scope:**
- Custom domains or memorable URLs
- Multi-port forwarding
- Persistent or named tunnels
- Team management or access control beyond token auth
- Paid tiers or hosted infrastructure

## Vision

p2p-tunnel starts as a simple, free, open-source tool that does one thing well: expose localhost over P2P with zero infrastructure. Where it goes from there will be shaped by the community and real-world usage. The foundation — libp2p networking, browser-native P2P, zero-server architecture — is designed to support whatever comes next.
