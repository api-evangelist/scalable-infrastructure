---
title: "From Ingress NGINX to Higress: migrating 60+ resources in 30 minutes with AI"
url: "https://www.cncf.io/blog/2026/04/23/from-ingress-nginx-to-higress-migrating-60-resources-in-30-minutes-with-ai/"
date: "Thu, 23 Apr 2026 13:37:18 +0000"
author: "Tianyi Zhang, Alibaba"
feed_url: "https://www.cncf.io/blog/feed/"
---
<p></p>



<p>With the <a href="https://kubernetes.io/blog/2026/01/29/ingress-nginx-statement/">official retirement of Ingress NGINX</a> that took place in March 2026, enterprise platform teams are facing an urgent security and compliance mandate. Remaining on a retired controller leaves critical infrastructure vulnerable to unpatched security risks. For one infrastructure engineer managing a cluster with over 60 complex Ingress resources, the challenge was clear: find a modern, enterprise-ready replacement that could be implemented without months of manual refactoring.</p>



<p>This blog post explains how full migration validation was achieved in just 30 minutes by leveraging an AI agent and <strong>Higress</strong>, a cloud-native and AI-native API gateway founded by Alibaba that recently joined the CNCF Sandbox.</p>



<h3 class="wp-block-heading">The solution: Why Higress for the AI era?</h3>



<p>Higress, which is built on the industry-standard <strong>Envoy and Istio</strong>, is specifically designed as an <strong>AI-native gateway</strong> that addresses the shortcomings of legacy controllers while providing specialized features for Large Language Models (LLMs).</p>



<ul class="wp-block-list">
<li><strong>AI-Native Architecture:</strong> Unlike traditional gateways, Higress treats LLMs as first-class citizens. It includes specialized features like <strong>Token-based rate limiting</strong> (to manage model costs) and <strong>caching capabilities</strong> (to reduce latency for common AI prompts).</li>



<li><strong>LLM Protocol Governance:</strong> It provides a unified protocol to interface with various LLM providers, enabling teams to switch models behind a single, secure endpoint.</li>



<li><strong>Zero-Downtime Reliability:</strong> Leveraging Envoy’s<a href="https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol"> xDS protocol</a>, Higress allows for configuration updates in milliseconds. This eliminates the &#8220;NGINX reload&#8221; issue, which is critical for maintaining persistent connections in AI streaming and gRPC.</li>



<li><strong>Model Context Protocol (MCP):</strong> Higress supports hosting MCP servers, allowing AI agents to securely interact with enterprise tools and data via the gateway.</li>
</ul>



<p><strong>AI-Assisted Migration Workflow</strong><strong><br /></strong><strong><br /></strong>To accelerate the transition, the Alibaba engineer utilized an AI agent equipped with specialized &#8220;Skills.&#8221; This approach shifted the &#8220;grunt work&#8221; of analysis and validation to the AI, while keeping human engineers in control of the final production execution.<br /></p>



<h4 class="wp-block-heading"><strong>1. Understanding the Current State</strong></h4>



<p>The agent was first tasked with auditing the existing cluster. Using the <a href="https://lobehub.com/skills/alibaba-higress-nginx-to-higress-migration"><strong>nginx-to-higress-migration</strong> <strong>skill</strong></a>, the agent automatically identified all Ingress resources and flagged NGINX-specific annotations that required translation.</p>



<h4 class="wp-block-heading"><strong>2. Risk-Free Simulation</strong><strong><br /></strong></h4>



<p>To ensure the migration wouldn&#8217;t break production traffic, the engineer used the agent to create a simulated environment using Kind (Kubernetes in Docker). Higress was installed with status updates disabled (global.enableStatus=false) to prevent Higress from updating the Ingress status field, allowing it to coexist peacefully with NGINX. This enabled the engineer to test the new routing logic side-by-side with the old NGINX controller.</p>



<h4 class="wp-block-heading"><strong>3. Solving Custom Logic with WASM</strong></h4>



<p>For the complex NGINX snippets flagged during analysis, the engineer utilized the <a href="https://lobehub.com/skills/alibaba-higress-higress-wasm-go-plugin"><strong>higress-wasm-go-plugin</strong></a> skill. This skill allowed the AI to generate high-performance WebAssembly (WASM) plugins that replicated custom Lua or NGINX logic within the Higress sandbox.</p>



<h3 class="wp-block-heading">Outcome: 30 Minutes to compliance</h3>



<p>By leveraging Higress’s native NGINX compatibility and AI-assisted validation, infrastructure migration was achieved at lightning speed:</p>



<figure class="wp-block-table"><table class="has-fixed-layout"><tbody><tr><td><strong>Phase</strong></td><td><strong>AI Agent Task</strong></td><td><strong>Outcome</strong></td></tr><tr><td><strong>Analysis</strong></td><td>Audit 60+ Ingress resources</td><td>Full gap analysis in &lt;1 minute</td></tr><tr><td><strong>Simulation</strong></td><td>Mirror environment in Kind</td><td>Verified &#8220;Digital Twin&#8221; with &lt;10 minute of manual typing</td></tr><tr><td><strong>Plugin Dev</strong></td><td>WASM Plugin Generation</td><td>Custom snippets translated in &lt;2 minutes</td></tr><tr><td><strong>Execution</strong></td><td>Generate Final Runbook</td><td><strong>Production-ready in 30 minutes</strong></td></tr></tbody></table></figure>



<p>The retirement of Ingress NGINX is not just a migration hurdle, but an opportunity to upgrade to a more resilient, AI-ready architecture. By moving to <strong>Higress</strong>, organizations gain an enterprise-grade gateway based on Envoy and Istio that is built for the future of LLM integration.</p>
