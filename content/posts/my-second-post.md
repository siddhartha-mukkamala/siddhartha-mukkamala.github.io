
+++
title = "Bridging the Multi-Cloud Gap -- A Practitioner’s Framework for Secure, Unified Connectivity Across AWS, GCP, and Azure "
date = 2026-04-02T10:00:00-05:00
draft = false
+++

# Bridging the Multi-Cloud Gap

## A Practitioner’s Framework for Secure, Unified Connectivity Across AWS, GCP, and Azure

**By Siddhartha Mukkamala**
*Senior Manager — Systems & Cloud Network Engineering*

---

## Introduction

Most enterprise multi-cloud environments are not intentionally designed — they are the result of acquisitions.

As organizations integrate acquired companies, they inherit heterogeneous cloud footprints: AWS in one business unit, Azure in another, GCP in a third. Each environment evolves independently, typically using the same default private IP ranges (most commonly `10.0.0.0/8`), with no expectation of future interconnectivity.

The challenge emerges when these systems must communicate. What follows is a predictable set of constraints: overlapping IP address space, inconsistent security models, limited end-to-end observability, and incompatible connectivity patterns across providers.

This is not a failure of architecture, but a structural consequence of integrating systems built in isolation across cloud ecosystems that lack a shared networking foundation.

I’ve worked on this problem at enterprise scale. This post distills those experiences into a practical framework that any organization can apply — regardless of which cloud providers they use or how they arrived at their current architecture.

---

## The Problem No One Plans For

When enterprises grow through acquisitions, they inherit multiple cloud environments — each built independently. In practice, this leads to the same underlying networking problem: every business unit and every acquired system uses the RFC 1918 private address space, most commonly `10.0.0.0/8`. It’s the default. Everyone uses it.

This creates a fundamental connectivity problem. When a service in AWS needs to call a service in GCP, which in turn needs to call a system in Azure, you can’t simply route between them — they all share the same IP range.

The default workaround is to route traffic over the public internet. But that introduces unnecessary latency, expands the attack surface by exposing internal services externally, and breaks security models that depend on private connectivity.

At enterprise scale, this challenge becomes operational reality — where thousands of applications across multiple cloud environments must communicate securely and reliably.

---

## The Unified Connectivity Fabric

### Layer 1 — The Non-Overlapping IP Fabric

The first design decision is the most important: choose an IP range for the connectivity fabric itself that will never overlap with anything else in your environment.

The RFC 1918 ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) are almost always already in use. The CGNAT range (`100.64.0.0/10`) may seem appealing, but it is frequently used by Kubernetes pod networking in managed services (GKE, EKS, AKS), making it unsafe to assume availability.

In practice, I used the **RFC 5735 benchmarking range (`198.18.0.0/15`)** for the connectivity fabric. This range is reserved for network device benchmarking and is not routed on the public internet.

Traffic entering the fabric is SNAT’d into this range, enabling communication across overlapping environments without IP address conflicts.

---

### Layer 2 — Domain-Name and URI-Path Routing

Instead of relying on IP addresses, the fabric uses application-layer routing:

* Domain name
* URI path

Example:

```
internal.example.com/bu2/service-api
```

This allows the fabric to:

* Identify the destination service
* Apply routing decisions
* Abstract underlying network complexity

Applications remain unaware of the routing layer.

---

### Layer 3 — The Edge Proxy Layer

Connectivity Edge nodes handle routing and inspection:

* SSL termination
* URI-based routing
* SNAT into fabric space
* HTTP/1.1 → HTTP/2 upgrades
* Inline WAF and DLP inspection

Persistent HTTP/2 connections reduce repeated TCP and TLS handshakes, improving latency across service chains.

---

### Layer 4 — The AI Gateway Extension

As organizations integrate AI services (AWS Bedrock, Vertex AI, Azure OpenAI), new challenges emerge.

The AI Gateway provides:

* Centralized control for AI traffic
* Rate limiting per business unit
* Inline DLP inspection
* WAF enforcement
* Intelligent routing based on compute availability

---

## Observability

The fabric enables end-to-end observability through correlation IDs:

* Generated at ingress
* Propagated across all hops
* Logged at each layer

This allows full request tracing across multi-cloud environments and supports compliance requirements such as PCI-DSS and DORA.

---

## Self-Service Onboarding

At scale, the network team cannot be the bottleneck.

The architecture enables:

* Self-service service registration
* Automated routing configuration
* Policy-driven access control

This transforms connectivity into a platform capability.

---

## Measured Outcomes

* ~25% improvement in P95 latency
* Elimination of public internet paths for internal traffic
* Unified security enforcement
* End-to-end observability
* ~65% reduction in infrastructure costs
* Faster onboarding through self-service

---

## Applicability

This framework applies to organizations that:

* Operate across multiple cloud providers
* Have overlapping private IP spaces
* Require consistent security enforcement
* Are integrating AI workloads
* Need strong observability and compliance

---

## Closing Thoughts

Multi-cloud complexity is not going away — it’s increasing, especially with AI workloads.

The key is not to eliminate that complexity, but to abstract it behind a unified, secure, and observable connectivity layer.

---

*If you’re working on similar challenges, I’d be interested in connecting and exchanging ideas.*
