---
title: "Benchmarking AI agent retrieval strategies on Kubernetes bug fixes"
url: "https://www.cncf.io/blog/2026/05/08/benchmarking-ai-agent-retrieval-strategies-on-kubernetes-bug-fixes/"
date: "2026-05-08"
author: "Brandon Foley"
feed_url: "https://www.cncf.io/blog/feed/"
---
I’ve been using AI coding agents as part of my daily engineering workflow and wanted to understand how well they actually perform on real-world bugs. To test this, I ran a series of structured experiments using bug reports from the Kubernetes repository, evaluating whether agents could produce correct, complete fixes without guidance in a large, multi-million-line codebase. My initial assumption was simple. Success would largely depend on retrieval. Whether via retrieval-augmented generation (RAG) or filesystem search, a model that finds the right code should be able to generate the right fix.
