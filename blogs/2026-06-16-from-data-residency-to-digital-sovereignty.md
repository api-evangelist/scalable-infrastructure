---
title: "From data residency to digital sovereignty: Architectural patterns for cloud native platforms"
url: "https://www.cncf.io/blog/2026/06/16/from-data-residency-to-digital-sovereignty-architectural-patterns-for-cloud-native-platforms/"
date: "2026-06-16"
author: "Hrittik Roy"
feed_url: "https://www.cncf.io/blog/feed/"
---
Digital sovereignty has become a practical platform engineering concern as regulations like the EU Data Act (applicable since January 2025), NIS-2, DORA, and the UK Data Use and Access Act 2025 shape infrastructure decisions, requiring control over control planes, encryption keys, administrative access, and auditability beyond regional placement. The article proposes tenant clusters as sovereignty primitives, giving each jurisdiction its own Kubernetes control plane running as pods on shared infrastructure for jurisdictional containment, cryptographic control, and workload portability. Examples include using vCluster for multi-jurisdiction SaaS and bare-metal GPU infrastructure for sovereign AI platforms.
