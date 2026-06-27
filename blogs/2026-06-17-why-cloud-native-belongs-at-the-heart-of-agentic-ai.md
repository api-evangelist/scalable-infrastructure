---
title: "Why cloud native belongs at the heart of agentic AI"
url: "https://www.cncf.io/blog/2026/06/17/why-cloud-native-belongs-at-the-heart-of-agentic-ai-lessons-from-building-a-multi-agent-security-platform-on-kubernetes/"
date: "2026-06-17"
author: "Willem Berroubache"
feed_url: "https://www.cncf.io/blog/feed/"
---
Lessons from building a real-time security operations platform using agentic AI on cloud native foundations, where each agent is deployed as an individual Kubernetes workload rather than an in-process module. Key practices include implementing mTLS with cert-manager and Cilium instead of service meshes, codifying safety constraints as OPA policies, propagating trace IDs through A2A protocols, and gating LLM invocation with classical anomaly models. The thesis is that "agentic AI inherits all the operational problems cloud native already solved," so AI agents should be treated as normal workloads.
