---
title: "ingress-nginx to Envoy Gateway migration on CNCF internal services cluster"
url: "https://www.cncf.io/blog/2026/04/13/ingress-nginx-to-envoy-gateway-migration-on-cncf-internal-services-cluster/"
date: "Mon, 13 Apr 2026 14:01:35 +0000"
author: "Koray Oksay, Kubermatic"
feed_url: "https://www.cncf.io/blog/feed/"
---
<p>CNCF hosts a Kubernetes cluster to run some services for internal purposes (namely; <a href="https://github.com/hackmdio/codimd">codimd</a>, <a href="https://guac.sh/">GUAC</a>, <a href="https://www.kcp.io/">kcp</a>).</p>



<p>The Kubernetes Project announced the <strong><a href="https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/" rel="noreferrer noopener" target="_blank">ingress-nginx retirement</a> </strong>(not to be confused with <a href="https://github.com/nginx/nginx" rel="noreferrer noopener" target="_blank">NGINX</a> or <a href="https://github.com/nginx/kubernetes-ingress" rel="noreferrer noopener" target="_blank">NGINX Ingress Controller</a>), which also affects the above mentioned Cluster. So we started looking into alternatives.</p>



<p>After some discussions, we decided to continue with <a href="https://gateway-api.sigs.k8s.io/guides/getting-started/">gateway-api</a> and its implementation as <a href="https://gateway.envoyproxy.io/">Envoy Gateway</a>.</p>



<p>Envoy Gateway is an CNCF open source project for managing Envoy Proxy as a standalone or Kubernetes-based application gateway. Gateway API resources are used to dynamically provision and configure the managed Envoy Proxies.</p>



<h2 class="wp-block-heading">gateway api and ingress-nginx architectures</h2>



<p><code>ingress-nginx</code> works with one <code>LoadBalancer</code> service; the ingress controller receives all traffic and distributes it based on the Ingress object configuration.</p>



<figure class="wp-block-image size-full"><img alt="Flow chart of the external load balancer working with the ingress controller to receive all traffic and distribute it based on the Ingress object configuration." class="wp-image-162609" height="742" src="https://www.cncf.io/wp-content/uploads/2026/04/image-6.png" width="1600" /></figure>



<p><br />On the other hand, gateway api is designed in multiple layers:<br /><br /></p>



<figure class="wp-block-image size-full"><img alt="A flow chart of the gateway API design " class="wp-image-162610" height="1099" src="https://www.cncf.io/wp-content/uploads/2026/04/image-7.png" width="1600" /></figure>



<p>Based on this design, it&#8217;s possible to create a <code>Gateway</code> object per <code>HTTPRoute</code> and/or <code>TLSRoute</code>. (Each <code>Gateway </code>creates a <code>LoadBalancer</code> type service on the cluster)</p>



<h2 class="wp-block-heading">Configuration for the services cluster</h2>



<p>It&#8217;s possible to configure a shared <code>Gateway</code> object and configure it on multiple <code>HTTPRoutes</code>. This is the closest configuration to the current <code>ingress-nginx</code> deployment with some advantages like:</p>



<ul class="wp-block-list">
<li><em>Cost and Resource Efficiency</em>: A single Gateway means one LoadBalancer service, which translates to one cloud load balancer. Multiple Gateways = multiple load balancers = significantly higher costs.</li>



<li><em>Operational Simplicity</em>: Managing one Gateway is simpler than managing dozens. We have a single point for TLS configuration, listeners, and overall gateway policy.</li>



<li><em>IP Address Management</em>: We get one stable IP for the ingress point. With multiple Gateways, we would need to manage multiple IPs and DNS entries.</li>
</ul>



<p><a href="https://github.com/cncf/automation/tree/main/ci/cluster/services/manifests/envoy-gateway">This folder</a> contains all the settings we implemented:</p>



<ol class="wp-block-list">
<li>GatewayClass to use Envoy Gateway</li>
</ol>



<ol class="wp-block-list">
<li>A shared <code>Gateway</code> to serve for Guac, codimd, and kcp.</li>



<li><code>EnvoyProxy</code> to configure HPA, service type, and other proxy settings.</li>



<li><code>ReferenceGrants</code> to allow the Gateway to access SSL certificates across namespaces</li>



<li><code>HTTPRoutes</code> for each service</li>



<li><code>BackendTLSPolict</code> to handle existing nginx annotations for backend HTTPS connections</li>
</ol>



<h2 class="wp-block-heading">How we migrated</h2>



<p>We had two options:</p>



<ol class="wp-block-list">
<li>Add Envoy Gateway with another public IP address and configure DNS to perform round-robin between ingress-nginx and Envoy</li>



<li>Configure Envoy Gateway to use the current IP address and move the whole traffic in one go.</li>
</ol>



<p>Although the first option is safer, we chose the second for the simplicity of our operation.</p>



<p>The reserved IP address was pushed to the repo as part of EnvoyProxy configuration:</p>



<pre class="wp-block-code"><code class="">apiVersion: gateway.envoyproxy.io/v1alpha1
kind: EnvoyProxy
metadata:
  name: ha-envoy-proxy
  namespace: envoy-gateway
spec:
  provider:
    type: Kubernetes
    kubernetes:
      envoyService:
        externalTrafficPolicy: Cluster
        type: LoadBalancer
        patch:
          type: StrategicMerge
          value:
            spec:
              loadBalancerIP: "146.235.214.235" # Reserved IP address on the cloud provider
              ports:
              - name: https-443
                port: 443
                targetPort: 10443
                protocol: TCP
                nodePort: 32050 # Fixed NodePort for external LB backend and firewall configuration
...
</code></pre>



<p><strong>Critical: externalTrafficPolicy Setting</strong></p>



<p>We initially encountered connection failures due to <code>externalTrafficPolicy: Local</code> (the default). This setting causes the NodePort to only listen on nodes that have an Envoy pod running. When the Oracle Cloud Load Balancer performed health checks on nodes without pods, they failed, marking all backends as unhealthy.</p>



<h3 class="wp-block-heading">What about certificates?</h3>



<p>We chose to use the existing certificates triggered by <code>ingress-nginx </code>via annotations:</p>



<pre class="wp-block-code"><code class="">---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
...
spec:
  gatewayClassName: envoy
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "*.cncf.io"
    tls:
      mode: Terminate
      certificateRefs:
      - name: guac-tls
        namespace: guac
        kind: Secret
        group: ""
      - name: auth-dex-tls
        namespace: auth
        kind: Secret
        group: ""
...
</code></pre>



<p>However, the certificates have an owner reference to the <code>Ingress object</code>. This means deleting an Ingress would cascade delete the Certificate and its Secret.</p>



<p>Below one-liner, removes the ownerReference from all Certificates that reference an Ingress:</p>



<pre class="wp-block-code"><code class="">kubectl get certificate -A -o json | jq -r '.items[] | select(.metadata.ownerReferences[]? | .kind == "Ingress") | "\(.metadata.namespace) \(.metadata.name)"' | while read NS NAME 
do 
    kubectl patch certificate $NAME -n $NS --type=json \
      -p='[{"op": "remove", "path": "/metadata/ownerReferences"}]' 
done
</code></pre>



<h3 class="wp-block-heading">Cross-namespace certificate access</h3>



<p>Since certificates are stored in different namespaces than the Gateway, we configured <code>ReferenceGrant</code> resources to allow cross-namespace access:</p>



<pre class="wp-block-code"><code class="">apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-gateway-to-certs
  namespace: codimd
spec:
  from:
  - group: gateway.networking.k8s.io
    kind: Gateway
    namespace: envoy-gateway
  to:
  - group: ""
    kind: Secret
    name: codimd-tls
</code></pre>



<p>This pattern was repeated for each namespace containing certificates.</p>



<h3 class="wp-block-heading">HTTPRoutes</h3>



<p><a href="https://github.com/kubernetes-sigs/ingress2gateway">ingress2gateway</a> helped to prepare the <code>HTTPRoute</code> objects from existing <code>Ingress</code> resources.</p>



<p>We had a special case for one ingress with backend HTTPS configuration:</p>



<pre class="wp-block-code"><code class="">nginx.ingress.kubernetes.io/backend-protocol: HTTPS
nginx.ingress.kubernetes.io/proxy-ssl-name: api.services.cncf.io
nginx.ingress.kubernetes.io/proxy-ssl-secret: kdp/kcp-ca
nginx.ingress.kubernetes.io/proxy-ssl-verify: "on"
</code></pre>



<p>To achieve the same behavior with Envoy Gateway, we created a <code>BackendTLSPolicy:</code></p>



<pre class="wp-block-code"><code class="">apiVersion: gateway.networking.k8s.io/v1
kind: BackendTLSPolicy
metadata:
  name: kdp-backend-tls
  namespace: kdp
spec:
  targetRefs:
  - group: ''
    kind: Service
    name: kcp-front-proxy
  validation:
    caCertificateRefs:
    - name: kcp-ca
      group: ''
      kind: Secret
    hostname: api.services.cncf.io

</code></pre>



<p><br /></p>



<h2 class="wp-block-heading">Troubleshooting</h2>



<h3 class="wp-block-heading">TLS handshake failures</h3>



<p>If you encounter <code>SSL_ERROR_SYSCALL</code> errors during TLS handshake:</p>



<ol class="wp-block-list">
<li>Check Gateway listener: Ensure the HTTPS listener is configured on port 443</li>



<li>Verify certificates are loaded: Check that all referenced certificates exist and are accessible</li>



<li>Check ReferenceGrants: Ensure cross-namespace certificate access is allowed</li>



<li>Review Envoy logs:</li>
</ol>



<pre class="wp-block-code"><code class="">kubectl logs -n envoy-gateway-system -l gateway.envoyproxy.io/owning-gateway-name=shared-gateway</code></pre>



<h3 class="wp-block-heading">Load balancer health check failures</h3>



<p>If the cloud load balancer shows backends as unhealthy:</p>



<ol class="wp-block-list">
<li>Verify externalTrafficPolicy: Should be Cluster, not Local</li>



<li>Check NodePort accessibility: Test from a node that the NodePort responds</li>



<li>Review health check configuration: Ensure the LB health check matches the service configuration</li>



<li>Check firewall rules: Verify security groups/NSGs allow traffic from LB subnet to NodePort</li>
</ol>



<h3 class="wp-block-heading">Certificate not being served</h3>



<p>If OpenSSL can&#8217;t retrieve a certificate:</p>



<pre class="wp-block-code"><code class="">echo | openssl s_client -connect &lt;lb-ip&gt;:443 -servername &lt;hostname&gt; 2&gt;/dev/null | openssl x509 -noout -text</code></pre>



<p>This indicates the certificate isn&#8217;t loaded. Check:</p>



<ol class="wp-block-list">
<li>Certificate is referenced in Gateway certificateRefs</li>



<li>ReferenceGrant exists for cross-namespace access</li>



<li>Gateway status shows Programmed: True</li>
</ol>



<h2 class="wp-block-heading">Day 2 operation on certificates</h2>



<p>We had decided to move the certificates later, to narrow the scope of the migration and easily use the current certificates at the time. However, when they expire, we could be in trouble. Here is what you need to do make sure that your certificates are managed by Gateway API + cert-manager:</p>



<h3 class="wp-block-heading">1. Make sure that cert-manager supports Gateway API:</h3>



<p>You need to enable Gateway API support on cert-manager:</p>



<pre class="wp-block-code"><code class="">apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cert-manager
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.jetstack.io
    targetRevision: v1.17.2
    chart: cert-manager
    helm:
      values: |
        config:
          enableGatewayAPI: true ## Make sure this exists!
</code></pre>



<h3 class="wp-block-heading">2. Update the <code>ClusterIssuer:</code></h3>



<p>Either update the current issuer or create a new one:</p>



<pre class="wp-block-code"><code class="">apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    preferredChain: ""
    privateKeySecretRef:
      name: letsencrypt-prod
    server: https://acme-v02.api.letsencrypt.org/directory
    solvers:
    - http01:
        gatewayHTTPRoute:
          parentRefs:
          - group: gateway.networking.k8s.io
            kind: Gateway
            name: shared-gateway       ## this is the name of your gateway
            namespace: envoy-gateway   ## where your gateway resides
</code></pre>



<h3 class="wp-block-heading">3. Annotate the Gateway for cert-manager</h3>



<p>You need to add the annotation, just like we do for ingress-nginx:</p>



<pre class="wp-block-code"><code class="">apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: shared-gateway
  namespace: envoy-gateway
  annotations:
    # needs to match with the ClusterIssuer you created/updated on previous step
    cert-manager.io/cluster-issuer: letsencrypt-prod  
spec:
  gatewayClassName: envoy
</code></pre>



<h3 class="wp-block-heading">4. Separate the listeners</h3>



<p>We initially had one listener for all our hosts, but they need to be separated (unless you use DNS solver for a wildcard certificate).</p>



<pre class="wp-block-code"><code class="">apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: shared-gateway
  namespace: envoy-gateway
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  gatewayClassName: envoy
  addresses:
   - type: IPAddress
     value: 146.235.214.235
  listeners:
  - name: https-guac
    protocol: HTTPS
    port: 443
    hostname: guac.cncf.io
    tls:
      mode: Terminate
      certificateRefs:
      - name: guac-tls-gw
        kind: Secret
        group: ""
    allowedRoutes:
      namespaces:
        from: All

    # added for cert-manager HTTP01 solver
  - name: http-guac
    protocol: HTTP
    port: 80
    hostname: guac.cncf.io
    allowedRoutes:
      namespaces:
        from: All

  - name: http-api-guac
    protocol: HTTP
    port: 80
    hostname: api.guac.cncf.io
    allowedRoutes:
      namespaces:
        from: All

    # added for cert-manager HTTP01 solver
  - name: https-notes
    protocol: HTTPS
    port: 443
    hostname: notes.cncf.io
    tls:
      mode: Terminate
      certificateRefs:
      - name: codimd-tls
        kind: Secret
        group: ""
    allowedRoutes:
      namespaces:
        from: All

  - name: http-notes
    protocol: HTTP
    port: 80
    hostname: notes.cncf.io
    allowedRoutes:
      namespaces:
        from: All
...
</code></pre>



<h3 class="wp-block-heading">5. Remove redundant <code>ReferenceGrants</code></h3>



<p>Since the new certificates are created on the same namespace with the Envoy Gateway (<code>shared-gateway</code> in our case), we don&#8217;t need the <code>ReferenceGrants</code> anymore. We removed them:</p>



<pre class="wp-block-code"><code class="">kubectl delete referencegrant --all -A</code></pre>



<h2 class="wp-block-heading">Conclusion</h2>



<p>The migration from ingress-nginx to Envoy Gateway required careful attention to:</p>



<ul class="wp-block-list">
<li>Certificate ownership and cross-namespace access</li>



<li>Cloud load balancer integration (NodePort, health checks, externalTrafficPolicy)</li>



<li>Backend TLS configuration for services requiring HTTPS upstream connections</li>
</ul>



<p>The Gateway API&#8217;s multi-layer architecture provides better separation of concerns compared to ingress-nginx, though it requires understanding additional resources like ReferenceGrants and <code>BackendTLSPolicy.</code></p>



<p>To sum it up, we can say that the cloud native world already provided alternatives before the sun setting of ingress nginx. We hope this small insight can help you in your journey of migrating away from ingress nginx.</p>
