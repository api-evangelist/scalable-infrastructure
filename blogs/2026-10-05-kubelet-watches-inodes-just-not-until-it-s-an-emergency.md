---
title: "Kubelet watches inodes. Just not until it’s an emergency."
url: "https://www.cncf.io/blog/2026/10/05/kubelet-watches-inodes-just-not-until-its-an-emergency/"
date: "2026-10-05"
author: "Onkar Raghunath Shelke. SRE. CNCF/LFX mentor across four KubeStellar mentorship projects (2025 Terms 2 and 3). GSoC 2026 contributor with Red Hat, on JBoss Web Server tooling. CKA, RHCSA, AWS SAA. Pune, India."
feed_url: "https://www.cncf.io/blog/feed/"
---
A worker node pages with NodeFilesystemFilesFillingUp. It’s on course to run out of inodes. You check the disk first, because that’s what everyone checks first, and it reads 83% used: 103G used, 22G available.
