---
title: "Kubernetes for platform teams: Leveraging k0s and k0rdent"
url: "https://www.cncf.io/blog/2026/04/27/kubernetes-for-platform-teams-leveraging-k0s-and-k0rdent/"
date: "Mon, 27 Apr 2026 11:00:00 +0000"
author: "Prithvi Raj (CNCF Ambassador) & Shivani Rathod (Bacancy Technology)"
feed_url: "https://www.cncf.io/blog/feed/"
---
<p>In our previous <a href="https://www.cncf.io/blog/2026/04/17/k3s-on-on-prem-infrastructures-the-gitops-way-writing-a-custom-k0rdent-template-from-scratch/">blog</a>, we explored a GitOps use case for on-premises infrastructure, managing multiple clusters hosted on the k3s Kubernetes distribution using k0rdent.&nbsp;</p>



<p>But the platform engineering ecosystem is vast, and one blog barely scratches the surface of what it takes to manage multi-cluster environments at ease, or to make the most of different Kubernetes distributions.</p>



<p>Ultimately, success isn&#8217;t about running Kubernetes, it&#8217;s about running it at scale, efficiently, and consistently.</p>



<p>That&#8217;s exactly what hosted control planes are designed to achieve.</p>



<h3 class="wp-block-heading">The scale problem nobody talks about enough</h3>



<p>How do you manage dozens or hundreds of clusters without costs and complexity spiralling out of control?</p>



<p>Open infrastructure is neither small nor shrinking. In fact, most practitioners I encounter day-to-day are running their workloads on OpenStack. And if you&#8217;re on OpenStack, the challenge of managing multi-cluster applications doesn&#8217;t just exist, it compounds. Every new cluster adds overhead, and that overhead adds up fast.</p>



<p>This blog explores how combining k0s, k0rdent, and Hosted Control Planes (HCP) can give you a scalable, cost-efficient, and production-ready Kubernetes platform on OpenStack.</p>



<h3 class="wp-block-heading">What are we solving?</h3>



<p>In a typical Kubernetes setup, every cluster ships with its own dedicated control plane — meaning at least 3 nodes per cluster just for the control plane itself. Multiply that across dev, staging, and production environments, and you&#8217;re burning through resources before your first workload even lands.</p>



<p>This is the problem Hosted Control Planes were built to solve.</p>



<p>Instead of running the API server, etcd, and controllers on dedicated nodes per cluster, HCP runs all of them inside a central management cluster. The result is fewer VMs, lower costs, simpler upgrades, and a single pane of control across your entire fleet.</p>



<figure class="wp-block-image size-full"><img alt="A flow char depicting the management cluster of k0s and k0rdent moving through the hosted control planes, to two OpenStack VM work clusters.
" class="wp-image-164096" height="549" src="https://www.cncf.io/wp-content/uploads/2026/04/image-12.png" width="816" /></figure>



<p>The combination of:</p>



<ul class="wp-block-list">
<li>k0s (lightweight Kubernetes)</li>



<li>k0rdent (multi-cluster orchestration)</li>



<li>OpenStack (private cloud infrastructure)</li>
</ul>



<p>creates a <strong>powerful platform engineering stack</strong>.</p>



<h4 class="wp-block-heading">1. Prepare your environment</h4>



<p>Before you touch Kubernetes, make sure your base environment is correct. Most failures happen here.</p>



<p><strong>Infrastructure requirements</strong></p>



<p>You need:</p>



<ul class="wp-block-list">
<li>One Linux VM (Ubuntu 20.04/22.04 recommended) for the management cluster
<ul class="wp-block-list">
<li>Minimum: 4 CPU, 8 GB RAM (more is better)</li>
</ul>
</li>



<li>Access to an OpenStack project with:
<ul class="wp-block-list">
<li>Enough quota (instances, volumes, floating IPs)</li>



<li>A working network + subnet + router</li>



<li>A usable image (Ubuntu cloud image)</li>



<li>At least one flavor (e.g., m1.medium)</li>
</ul>
</li>
</ul>



<h4 class="wp-block-heading">Tools to install on your VM:</h4>



<pre class="wp-block-code"><code class="">sudo apt update
sudo apt install -y curl wget jq unzip</code></pre>



<p>Install kubectl:</p>



<pre class="wp-block-code"><code class="">curl -LO https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/</code></pre>



<p>Install Helm:</p>



<pre class="wp-block-code"><code class="">curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash</code></pre>



<p>Install OpenStack CLI:</p>



<pre class="wp-block-code"><code class="">pip install python-openstackclient</code></pre>



<h4 class="wp-block-heading">2. Create the management cluster using k0s</h4>



<p>k0s is used because it is lightweight and simple, which is ideal for a management cluster.</p>



<h4 class="wp-block-heading">Install k0s:</h4>



<pre class="wp-block-code"><code class="">curl -sSLf https://get.k0s.sh | sudo sh</code></pre>



<p>Initialize and start the controller:</p>



<pre class="wp-block-code"><code class="">sudo k0s install controller --single
sudo k0s start
</code></pre>



<p>Wait for about 30–60 seconds, then export kubeconfig:</p>



<pre class="wp-block-code"><code class="">sudo k0s kubeconfig admin > ~/.kube/config</code></pre>



<p>Verify:</p>



<pre class="wp-block-code"><code class="">kubectl get nodes</code></pre>



<p>You should see one node in Ready state.</p>



<h4 class="wp-block-heading">3. Install k0rdent on the management cluster</h4>



<p>k0rdent runs as controllers inside your management cluster and handles cluster lifecycle.</p>



<h4 class="wp-block-heading">Add Helm repository:</h4>



<pre class="wp-block-code"><code class="">helm repo add k0rdent https://charts.k0rdent.io
helm repo update</code></pre>



<p>Install k0rdent:</p>



<pre class="wp-block-code"><code class="">helm install kcm oci://ghcr.io/k0rdent/kcm/charts/kcm --version 1.8.0 -n kcm-system --create-namespace</code></pre>



<p>Verify installation:</p>



<pre class="wp-block-code"><code class="">kubectl get pods -n kcm-system</code></pre>



<p>Wait until all pods are in Running state. If they are not, check logs before proceeding.</p>



<h4 class="wp-block-heading">4. Configure OpenStack access</h4>



<p>This step is critical. k0rdent needs credentials to create VMs on your behalf.</p>



<p>Load your OpenStack credentials</p>



<p>You should have an openrc.sh file from your OpenStack environment.</p>



<p>source openrc.sh</p>



<p>Verify access:</p>



<pre class="wp-block-code"><code class="">openstack server list
openstack network list
openstack image list
</code></pre>



<p>If these commands fail, fix OpenStack access before moving forward.</p>



<h4 class="wp-block-heading">5. Create Kubernetes secret for OpenStack credentials</h4>



<p>k0rdent expects a clouds.yaml format.</p>



<p>Create a file:</p>



<pre class="wp-block-code"><code class="">apiVersion: v1
kind: Secret
metadata:
 name: openstack-cloud-config
 namespace: kcm-system
type: Opaque
stringData:
 clouds.yaml: |
   clouds:
     openstack:
       auth:
         auth_url: https://&lt;OPENSTACK_AUTH_URL>
         username: &lt;USERNAME>
         password: &lt;PASSWORD>
         project_name: &lt;PROJECT_NAME>
         user_domain_name: Default
         project_domain_name: Default
       region_name: RegionOne
       interface: public
       identity_api_version: 3

</code></pre>



<p>Apply it:</p>



<pre class="wp-block-code"><code class="">kubectl apply -f cloud-config.yaml</code></pre>



<h4 class="wp-block-heading">6. Create k0rdent credential object</h4>



<p>This tells k0rdent to use the OpenStack secret.</p>



<p>apiVersion: k0rdent.mirantis.com/v1beta1</p>



<pre class="wp-block-code"><code class="">kind: Credential
metadata:
 name: openstack-credential
 namespace: kcm-system
spec:
 type: openstack
 secretRef:
   name: openstack-cloud-config
</code></pre>



<p>Apply:</p>



<pre class="wp-block-code"><code class="">kubectl apply -f credential.yaml</code></pre>



<h4 class="wp-block-heading">7. Identify required OpenStack resources</h4>



<p>You must use real names from your OpenStack environment.</p>



<p>Run:</p>



<pre class="wp-block-code"><code class="">openstack network list
openstack subnet list
openstack router list
openstack image list
openstack flavor list
</code></pre>



<p>You will need:</p>



<ul class="wp-block-list">
<li>External network name (e.g., public)</li>



<li>Image name (e.g., ubuntu-20.04)</li>



<li>Flavor (e.g., m1.medium)</li>
</ul>



<p><em>Note: If these values are wrong, cluster creation will fail silently or partially.</em></p>



<h4 class="wp-block-heading">8. Creating your ClusterDeployment (core step)</h4>



<p>This is where you define your Kubernetes cluster declaratively. This is how we have defined our Cluster Deployment, you can can deploy it according to your needs:</p>



<pre class="wp-block-code"><code class="">apiVersion: k0rdent.mirantis.com/v1beta1
kind: ClusterDeployment
metadata:
 name: openstack-hcp
 namespace: kcm-system
spec:
 template: openstack-hosted-cp
 credential: openstack-credential

 config:
   workersNumber: 2

   flavor: m1.medium

   image:
     filter:
       name: ubuntu-20.04

   externalNetwork:
     filter:
       name: public

   identityRef:
     name: openstack-cloud-config
     cloudName: openstack
</code></pre>



<p>Apply:</p>



<h4 class="wp-block-heading">9. Observe cluster creation</h4>



<pre class="wp-block-code"><code class="">kubectl apply -f clusterdeployment.yaml</code></pre>



<p>Now the system starts working in the background.</p>



<p>Watch resources:</p>



<p>kubectl get clusterdeployments -n kcm-system</p>



<p>kubectl get pods -n kcm-system</p>



<p>Also monitor OpenStack:</p>



<p>openstack server list</p>



<p>You should see worker VMs being created.</p>



<p>Important point:</p>



<ul class="wp-block-list">
<li>Control plane components run inside the management cluster</li>



<li>Only worker nodes are created in OpenStack</li>
</ul>



<h4 class="wp-block-heading">10. Retrieve kubeconfig of the new cluster</h4>



<p>Once the cluster is ready, k0rdent creates a secret containing kubeconfig.</p>



<pre class="wp-block-code"><code class="">kubectl get secret openstack-hcp-kubeconfig \
 -n kcm-system \
 -o jsonpath="{.data.value}" | base64 -d > kubeconfig.yaml
</code></pre>



<p>Use it:</p>



<pre class="wp-block-code"><code class="">kubectl get nodes --kubeconfig=kubeconfig.yaml</code></pre>



<p>You should see your OpenStack worker nodes.</p>



<h4 class="wp-block-heading">11. Validate with a real workload</h4>



<p>Deploy something simple:</p>



<pre class="wp-block-code"><code class="">kubectl run nginx --image=nginx
kubectl get pods
</code></pre>



<p>Expose it if needed:</p>



<pre class="wp-block-code"><code class="">kubectl expose pod nginx --port=80 --type=NodePort</code></pre>



<p>This confirms the cluster is functional.</p>



<h4 class="wp-block-heading">12. Test scaling (important for demo and validation)</h4>



<p>Edit cluster:</p>



<pre class="wp-block-code"><code class="">kubectl edit clusterdeployment openstack-hcp -n kcm-system</code></pre>



<p>Change:</p>



<pre class="wp-block-code"><code class="">workersNumber: 3</code></pre>



<p>Watch OpenStack again:</p>



<p>openstack server list</p>



<p>A new VM should be created.</p>



<p>This proves:</p>



<p>&#8220;Declarative scaling works / k0rdent reconciles desired state</p>



<p>It&#8217;s easy to walk away thinking you&#8217;ve just provisioned a cluster. You haven&#8217;t. What we built is fundamentally different:</p>



<h3 class="wp-block-heading">A centralized control plane architecture</h3>



<p>In a conventional setup, every cluster is an island:&nbsp; its own API server, its own etcd, its own everything. Multiply that by fifty and you don&#8217;t have a platform, you have a sprawl. Teams spend more time keeping clusters alive than using them.</p>



<p>Centralized control plane architecture breaks this pattern. All control planes live inside one management cluster, one place for state, one place for API requests, one place to go when something needs attention. The management cluster becomes the brain. Workload clusters become its extensions.</p>



<p>This isn&#8217;t just an infrastructure optimization. It&#8217;s an architectural shift in how responsibility is distributed.</p>



<h3 class="wp-block-heading">A declarative cluster provisioning system</h3>



<p>In the old world, provisioning a cluster meant scripts, runbooks, and CLI commands fired in a specific order. It was imperative and it lived in someone&#8217;s head.</p>



<p>With k0rdent, you describe what you want. The system figures out how to get there. Provisioning becomes reproducible, auditable, and version-controlled by design — not by accident.</p>



<h3 class="wp-block-heading">A multi-cluster platform foundation</h3>



<p>A single cluster is a tool. A well-architected multi-cluster system is a platform built to serve many teams, many workloads, and many environments consistently, without every consumer needing to understand what&#8217;s underneath.</p>



<p>That&#8217;s what we&#8217;ve laid down here. Onboard new clusters without rethinking the architecture. Enforce policies from a single point. Upgrade and observe your entire fleet without treating each cluster as a unique snowflake.</p>



<h3 class="wp-block-heading">The core shift: From cluster-centric to platform-centric</h3>



<p>Before this architecture, each cluster managed itself, your attention and tooling fragmented across however many clusters you had. After this architecture, one system manages all clusters. Every new cluster is just another expected, handled event.</p>



<p>You stop thinking of yourself as someone who manages clusters, and start thinking as someone who operates a system that manages clusters. That shift is what lets teams scale infrastructure without scaling their operational toil at the same rate.</p>



<p>You didn&#8217;t just build a cluster. You built the system that builds clusters.</p>



<h3 class="wp-block-heading">Feel free to check out more on the k0s and k0rdent community</h3>



<p>The stack we&#8217;ve walked through in this blog k0s, k0rdent, and OpenStack, isn&#8217;t just a technical combination. It represents a growing ecosystem of open-source contributors, platform engineers, and infrastructure practitioners who are actively shaping what production Kubernetes looks like at scale.</p>



<p>Both projects are open source, actively maintained, and genuinely community-driven. If anything in this blog sparked a question, a use case idea, or even a disagreement, the communities are the right place to take it.</p>



<p>If you want to explore k0rdent further, dig into the project on <a href="https://github.com/k0rdent/k0rdent">GitHub</a> and bring your questions or feedback to the <strong>#k0rdent</strong> channel on the <a href="http://slack.cncf.io/">CNCF Slack</a>. It&#8217;s an active space where contributors and users are building in the open.</p>



<p>For k0s, the <a href="https://github.com/k0sproject/k0s">GitHub repository</a> is the best place to follow development and contribute. On the <a href="http://slack.kubernetes.io/">Kubernetes Slack</a>, the <strong>#k0s-users</strong> channel is where you&#8217;ll find practitioners sharing real-world experience, and <strong>#k0s-dev</strong> is where the core development conversation happens, worth joining if you want to go deeper or contribute upstream.</p>



<p>The best way to learn this stack isn&#8217;t just to read about it: it&#8217;s to build with it, break it, and ask questions alongside people who are doing the same.</p>
