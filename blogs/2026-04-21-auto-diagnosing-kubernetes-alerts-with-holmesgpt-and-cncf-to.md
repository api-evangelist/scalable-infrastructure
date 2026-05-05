---
title: "Auto-diagnosing Kubernetes alerts with HolmesGPT and CNCF tools"
url: "https://www.cncf.io/blog/2026/04/21/auto-diagnosing-kubernetes-alerts-with-holmesgpt-and-cncf-tools/"
date: "Tue, 21 Apr 2026 15:06:09 +0000"
author: "Grace Park and Ihyeok Song, DevOps Engineer, STCLab SRE Team"
feed_url: "https://www.cncf.io/blog/feed/"
---
<p><em>What a two-person SRE team learned building an AI investigation pipeline. Spoiler: the runbooks mattered more than the model.</em></p>



<h3 class="wp-block-heading">Why we built this</h3>



<p>At STCLab, our SRE team supports multiple Amazon EKS clusters running high-traffic production workloads. We&#8217;ve got the full observability stack in place: OpenTelemetry feeding into Mimir, Loki, and Tempo. Robusta OSS enriches Prometheus alerts with error logs, Grafana links, and team mentions before dropping them into Slack.</p>



<p>So the data was never the problem. The problem was what happened next. Every alert meant the same drill: check the pod, query Prometheus, dig through Loki, pull traces, try to correlate. Fifteen to twenty minutes, every single time. We wanted that first pass to happen automatically and show up in the same Slack thread.</p>



<h3 class="wp-block-heading">HolmesGPT: Letting the LLM decide what to investigate</h3>



<figure class="wp-block-image size-full"><img alt="A cooler image (flow chart) of the SRE Alerts Pipeline" class="wp-image-163848" height="1280" src="https://www.cncf.io/wp-content/uploads/2026/04/image-10.png" title="Alerts Pipeline" width="1600" /></figure>



<p>Alerts Pipeline</p>



<p>We went with HolmesGPT (CNCF Sandbox) because of how it works: the ReAct pattern. The LLM reads an alert, picks a tool, reads the result, then decides what to check next. If a pod restarts, it might start with the exit code, pull Loki logs across clusters through VPC peering, then look at CPU pressure in Prometheus. The path isn&#8217;t scripted it depends on what the model actually finds.</p>



<p>That matters in our case, because not every namespace looks the same. Some have the full picture: centralized logs, distributed traces, the works. Multi-tenant workloads often have none of that; for those namespaces, it&#8217;s kubectl and Prometheus only. We capture these differences in markdown runbooks, each with a metadata header:</p>



<ol class="wp-block-list">
<li><code>## Meta</code></li>



<li><code>scope: namespace=&lt;target&gt; only</code></li>



<li><code>tools: kubectl, prometheus, loki, tempo</code></li>



<li><code>caution: some containers excluded from log collection → use kubectl logs</code></li>
</ol>



<p>Holmes calls <code>fetch_runbook</code> early in its investigation. The metadata tells it which tools are available and which ones to skip.</p>



<h3 class="wp-block-heading">Making it work with Robusta</h3>



<p>Our custom playbook is about 200 lines of Python. It covers what HolmesGPT doesn&#8217;t.</p>



<p>Robusta posts the alert to Slack before Holmes is done investigating, so our playbook has to find the right thread after the fact and post results as a reply. When Prometheus fires one alert per pod during a rollout, the playbook fingerprints at the workload level and suppresses repeats for 30 minutes. And since Robusta routes to different Slack channels by namespace, the playbook replicates that mapping to find where to post.</p>



<h3 class="wp-block-heading">Runbooks changed everything</h3>



<p>We started by focusing on model selection. What actually determined investigation quality was the runbooks.</p>



<p>Without runbooks, the model just guesses. It might check Istio metrics in namespaces that have no sidecars, or query Loki where nothing is being collected. Eventually it loops, burns through its step budget, and comes back with &#8220;I need more information.&#8221;</p>



<p>What fixed this wasn&#8217;t a better model. It was telling the model what <em>not</em> to do. Once we added exclusion rules to our runbooks (&#8220;no Loki, no Tempo, no Istio here; use kubectl and PromQL only&#8221;), wasted tool calls dropped from 16 to 2 per investigation.</p>



<p>We ran a controlled comparison to confirm this: the same ClickHouse handshake alert, tested four ways. With runbooks, Holmes matched the known error pattern in 3 to 4 tool calls and used the rest of its budget to verify. Without runbooks, it chased three entirely different hypotheses (proxy scaling, schema mismatch, port misconfiguration) and burned through 20+ steps before reaching a conclusion. Same model, same alert. The runbook didn&#8217;t hand it the answer. It just narrowed the search space enough that a 12-step budget was plenty.</p>



<p>We now maintain seven runbooks, organized by namespace and alert type. When an investigation comes back wrong, the first question we ask is &#8220;does the runbook cover this?&#8221; Not &#8220;do we need a better model?&#8221;</p>



<figure class="wp-block-image size-full"><img alt="Screenshots: Same alert, same model. Left: with runbook guidance. Right: without runbooks." class="wp-image-163849" height="515" src="https://www.cncf.io/wp-content/uploads/2026/04/image-11.png" width="1024" /></figure>



<p>Same alert, same model. Left: with runbook guidance. Right: without runbooks.</p>



<h3 class="wp-block-heading">The model journey</h3>



<p>We tested seven models across self-hosted and managed hosting.</p>



<p>Self-hosted came first, running on Spot GPUs managed by KubeAI (CNCF). The 7B model couldn&#8217;t produce valid tool calls. The 9B model&#8217;s thinking mode clashed with the ReAct loop and returned empty responses. A 14B looked promising, but Spot evictions kept killing our runs, and cold starts took 5 to 8 minutes while Karpenter spun up nodes.</p>



<p>Then we tried managed APIs through VPC endpoints, which keeps cluster data inside our infrastructure. Most models didn&#8217;t work; several choked on HolmesGPT&#8217;s prompt caching markers. Only one model family passed everything we needed: Korean output, Slack formatting, runbook retrieval, and cross-cluster log correlation. We also contributed a three-line upstream fix for pod identity authentication (PR #1850, merged).</p>



<p>Today we run a hybrid setup: self-hosted in staging, managed API in production. Switching between them is one YAML block:</p>



<pre class="wp-block-code"><code class="">modelList:

&nbsp;primary:

&nbsp;&nbsp;&nbsp;model: "provider/model-name"&nbsp; # swap provider and model ID

&nbsp;&nbsp;&nbsp;api_base: "https://endpoint"&nbsp; # managed API or self-hosted

&nbsp;&nbsp;&nbsp;temperature: 0</code></pre>



<p>Cost comes out to about $0.04 per investigation, roughly $12 a month. Pipeline, playbook, runbooks — all unchanged regardless of backend.</p>



<h3 class="wp-block-heading">What actually mattered</h3>



<p>Some numbers. Workload-level deduplication takes around 40 raw daily alerts down to about 12 unique investigations. Engineers read a threaded summary in under two minutes instead of spending 15 to 20 on manual triage. Roughly 40% of investigations resolve on their own: OOMKilled, ImagePullBackOff, and other known patterns where Holmes matches a runbook and the root cause is obvious.</p>



<p>Here&#8217;s what we&#8217;d tell another team starting this.</p>



<p><strong>Runbooks over models.</strong> We ran a controlled test where the same model scored 4.6 out of 5 with runbooks and 3.6 without, on the exact same alert. The exclusion rules we wrote into our runbooks moved the needle more than any model swap ever did.</p>



<p><strong>Glue code is real work.</strong> That 200-line playbook handles timing, dedup, routing, and thread matching. HolmesGPT handles reasoning. You need both.</p>



<p><strong>Design for model migration.</strong> We&#8217;ve swapped backends three times now without touching the pipeline. The playbook is the stable core. The model is the part you replace.</p>



<p>What&#8217;s next: we&#8217;re looking at Inspektor Gadget (CNCF) to feed eBPF-level network metrics — TCP retransmits, connection latency — into the same pipeline through Prometheus. The architecture stays the same. Holmes just gets better data to work with.</p>



<p></p>



<p><em>About STCLab: STCLab builds high-performance online traffic management software. NetFUNNEL (Virtual Waiting Room) and BotManager (bot mitigation) serve millions of concurrent users across 200 countries.</em></p>



<p><em>Questions? </em><a href="mailto:grace@stclab.com">grace@stclab.com</a> | <a href="http://WWW.STCLAB.COM">stclab.com</a></p>



<p></p>
