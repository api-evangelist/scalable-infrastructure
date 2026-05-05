---
title: "K3s on On-Prem Infrastructures the GitOps Way: Writing a Custom k0rdent Template from Scratch"
url: "https://www.cncf.io/blog/2026/04/17/k3s-on-on-prem-infrastructures-the-gitops-way-writing-a-custom-k0rdent-template-from-scratch/"
date: "Fri, 17 Apr 2026 11:59:00 +0000"
author: "Shivani Rathod (Improwised Tech) & Prithvi Raj (CNCF Ambassador)"
feed_url: "https://www.cncf.io/blog/feed/"
---
<p>Kubernetes turns 12 this year. In that time, it&#8217;s gone from a Google side project to the operating system of modern infrastructure&nbsp; running everywhere from mainframes to GPUs, across multi-cloud, hybrid, on-prem, and edge environments. The CNCF landscape has grown alongside it, filling in the gaps that Kubernetes left open.</p>



<p>This blog isn&#8217;t about all of those gaps. It&#8217;s about one specific intersection: <strong>lightweight Kubernetes with K3s on an on-premise infrastructure (in this case Proxmox) and declarative multi-cluster management with k0rdent.</strong></p>



<p>If you&#8217;ve run Kubernetes on on-prem infrastructure, you know the pain:</p>



<ul class="wp-block-list">
<li>Manual VM creation</li>



<li>Bash scripts that only <em>you</em> understand</li>



<li>Clusters that work once, and then become untouchable</li>
</ul>



<p>We wanted something declarative, repeatable, and clean, but still friendly to an on-prem setup. That&#8217;s where k0rdent, Proxmox, and K3s came together.</p>



<p>In this blog, we&#8217;ll walk through how we curated a use case for provisioning and used k0rdent to provision a K3s cluster on an On-premise environment&nbsp; by writing our own Helm charts and using k0rdent&#8217;s <strong>Bring Your Own Template (BYOT)</strong> approach. This isn&#8217;t a theoretical post, this is exactly how our cluster gets created.</p>



<h2 class="wp-block-heading"><strong>The Big Picture</strong></h2>



<p>Here&#8217;s what we set out to build:</p>



<ul class="wp-block-list">
<li>Many end-users host their infrastructure layer on-premise, let us build that for you.</li>



<li><strong>Existing VM templates</strong> instead of building images every time</li>



<li><strong>k0rdent</strong> managing the full cluster lifecycle</li>



<li><strong>K3s</strong> as the Kubernetes bootstrap</li>
</ul>



<p>At a high level, the flow looks like this:</p>


<p>User → k0rdent<br />
         ↓<br />
Proxmox Infrastructure (BYOT VMs)<br />
         ↓<br />
Control Plane Provider<br />
         ↓<br />
Bootstrap Provider (K3s)<br />
         ↓<br />
Running Kubernetes Cluster</p>



<p>Each layer does one job, and does it well.</p>



<h2 class="wp-block-heading"><strong>Why On-Prem + k0rdent Makes Sense</strong></h2>



<p>Proxmox is one of the examples of a self-hosted environment, but Kubernetes on-prem often ends up being hand-crafted, hard to scale, and harder to reproduce.</p>



<p>k0rdent changes that mindset. Instead of scripting <em>how</em> things should happen, you describe <em>what</em> you want and let reconciliation do the work.</p>



<p>k0rdent offers a clear separation of concerns:</p>



<ol class="wp-block-list">
<li><strong>Infrastructure</strong> — provision the VMs</li>



<li><strong>Control Plane</strong> — configure the cluster topology</li>



<li><strong>Bootstrap</strong> — install and initialise Kubernetes</li>
</ol>



<p>That separation made Proxmox integration surprisingly clean.</p>



<h2 class="wp-block-heading"><strong>Step 1: Infrastructure Provider (BYOT)</strong></h2>



<p>k0rdent doesn&#8217;t ship with a native Proxmox provider, so we built one.</p>



<p>We created a custom Helm chart that acts as an Infrastructure Provider for Proxmox.</p>



<h3 class="wp-block-heading"><strong>Why Bring Your Own Template?</strong></h3>



<p>Instead of dynamically building VM images on every provision, I used existing Proxmox VM templates. My templates already had cloud-init enabled, SSH access configured, and base OS packages installed.</p>



<p>This gave me:</p>



<ul class="wp-block-list">
<li><strong>Faster VM provisioning</strong> — no image builds in the critical path</li>



<li><strong>Better control over OS hardening</strong> — managed outside the Kubernetes workflow</li>



<li><strong>Easier debugging</strong> — when something goes wrong, the VM layer is a known quantity</li>
</ul>



<h3 class="wp-block-heading"><strong>What the Helm Chart Does</strong></h3>



<p>The Proxmox infrastructure chart is intentionally scoped — it <em>only</em> handles infrastructure:</p>



<ul class="wp-block-list">
<li>Talks to the Proxmox API</li>



<li>Clones VMs from a template</li>



<li>Sets CPU, memory, and networking</li>



<li>Injects SSH keys</li>



<li>Outputs VM metadata back to k0rdent</li>
</ul>



<p>No Kubernetes logic here. Just clean VM provisioning.</p>



<h2 class="wp-block-heading"><strong>Step 2: Control Plane Provider</strong></h2>



<p>Helm charts for the Proxmox provider are available here: <a href="https://github.com/Improwised/charts/tree/main/charts/cluster-api-provider-proxmox">https://github.com/Improwised/charts/tree/main/charts/cluster-api-provider-proxmox</a></p>



<p>Once the VMs exist, the Control Plane Provider takes over. Its job is to:</p>



<ul class="wp-block-list">
<li>Decide which nodes are control plane nodes</li>



<li>Apply cluster-level configuration</li>



<li>Coordinate with the bootstrap provider</li>
</ul>



<p>In our setup, control plane nodes run on Proxmox VMs, with VM details flowing directly from the infrastructure provider.</p>



<p>&nbsp;Roles are assigned declaratively, the cluster feels <strong>intentional</strong>, not accidental.</p>



<figure class="wp-block-image size-full"><img alt=" Roles are assigned declaratively, the cluster feels intentional, not accidental." class="wp-image-163578" height="64" src="https://www.cncf.io/wp-content/uploads/2026/04/image-8.png" width="1600" /></figure>



<figure class="wp-block-image size-full"><img alt="Control Plane Provider: Nodes run on Proxmox VMs, with VM details flowing directly from the infrastructure provider." class="wp-image-163579" height="900" src="https://www.cncf.io/wp-content/uploads/2026/04/image-9.png" width="1600" /></figure>



<h2 class="wp-block-heading"><strong>Step 3: Bootstrapping Kubernetes with K3s</strong></h2>



<p>For bootstrapping, I chose K3s — and it fits perfectly here.</p>



<p><strong>Why K3s?</strong> It&#8217;s lightweight, fast to install, has minimal dependencies, and is purpose-built for on-prem and edge environments.</p>



<p>Here&#8217;s what the provider definitions look like:</p>



<pre class="wp-block-code"><code class="">apiVersion: operator.cluster.x-k8s.io/v1alpha2

kind: BootstrapProvider

metadata:

  name: k3s

spec:

  version: v0.3.0

  fetchConfig:

    url: https://github.com/k3s-io/cluster-api-k3s/releases/v0.3.0/bootstrap-components.yaml

  {{- if .Values.configSecret.name }}

  configSecret:

    name: {{ .Values.configSecret.name }}

    namespace: {{ .Values.configSecret.namespace | default .Release.Namespace | trunc 63 }}

  {{- end }}</code></pre>



<p>&#8212;</p>



<pre class="wp-block-code"><code class="">apiVersion: operator.cluster.x-k8s.io/v1alpha2

kind: ControlPlaneProvider

metadata:

  name: k3s

spec:

  version: v0.3.0

  fetchConfig:

    url: https://github.com/k3s-io/cluster-api-k3s/releases/v0.3.0/control-plane-components.yaml

  {{- if .Values.configSecret.name }}

  configSecret:

    name: {{ .Values.configSecret.name }}

    namespace: {{ .Values.configSecret.namespace | default .Release.Namespace | trunc 63 }}

  {{- end }}</code></pre>



<figure class="wp-block-image size-full"><img alt="Bootstrapping Kubernetes with K3s" class="wp-image-163580" height="900" src="https://www.cncf.io/wp-content/uploads/2026/04/image-9.png" width="1600" /></figure>



<h3 class="wp-block-heading"><strong>What the Bootstrap Provider Does</strong></h3>



<p>The Bootstrap Provider Helm chart handles the K3s lifecycle:</p>



<ol class="wp-block-list">
<li>Installs K3s on the first control plane node</li>



<li>Extracts the cluster token</li>



<li>Joins additional control plane and worker nodes</li>



<li>Generates kubeconfig access</li>
</ol>



<p>Once this step completes, Kubernetes is up and running.</p>



<h2 class="wp-block-heading"><strong>How k0rdent Ties It All Together</strong></h2>



<p>By using this method,&nbsp; the system <strong>continuously reconciles</strong> the desired state:</p>



<ol class="wp-block-list">
<li>Provisions Proxmox VMs</li>



<li>Waits for them to become reachable</li>



<li>Passes VM data to the control plane provider</li>



<li>Triggers the K3s bootstrap process</li>



<li>Keeps watching — and corrects drift</li>
</ol>



<p>The result: a <strong>fully declarative and managed K3s cluster on your on-premise environment.</strong></p>



<h2 class="wp-block-heading"><strong>What You End Up With</strong></h2>



<p>After everything settles, you have:</p>



<ul class="wp-block-list">
<li>A working K3s cluster running on On-prem VMs</li>



<li>Control plane and worker nodes provisioned declaratively</li>



<li>A setup you can recreate, scale, or tear down at any time</li>
</ul>



<p>Scaling the cluster is no longer a weekend project. Rebuilding it is no longer terrifying. It&#8217;s just configuration.</p>



<p>Running Kubernetes on-prem doesn&#8217;t have to mean duct tape and bash scripts. With k0rdent&#8217;s BYOT approach, your On-prem infrastructure becomes a first-class citizen; declarative, reproducible, and treated as a first-class citizen.. If your infrastructure isn&#8217;t supported out of the box, build the template. That&#8217;s the whole point.</p>
