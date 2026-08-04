---
title: "Is a Pod the right deployment unit for an AI agent?"
url: "https://www.cncf.io/blog/2026/07/14/is-a-pod-the-right-deployment-unit-for-an-ai-agent/"
date: "2026-07-14"
author: "Lin Sun, Solo.io | CNCF Ambassador"
feed_url: "https://www.cncf.io/blog/feed/"
---
When we first started building kagent, we didn’t run every agent in its own Kubernetes Pod, Service, and ServiceAccount. Instead, agents were simply executed inside the kagent runtime. It was the simplest architecture possible: one runtime...
