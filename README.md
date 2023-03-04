# cert-manager-for-ingress
## Assuming Resources are already created and a continutation of my work from https://github.com/ericausente/eks-cluster-with-NIC-and-KIC


When using Cert-manager with your Nginx ingress controller, you don't need to create your own certificate manually. Cert-manager will handle the creation and management of your SSL/TLS certificates automatically.

Here's what you'll need to do to configure Cert-manager to manage SSL/TLS certificates for your Nginx ingress controller:

1. Install Cert-manager on your Kubernetes cluster. You can follow the installation instructions in the Cert-manager documentation: https://cert-manager.io/docs/installation/

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
```

Resources created: 
```
namespace/cert-manager created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
serviceaccount/cert-manager-cainjector created
serviceaccount/cert-manager created
serviceaccount/cert-manager-webhook created
configmap/cert-manager-webhook created
clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrole.rbac.authorization.k8s.io/cert-manager-view created
clusterrole.rbac.authorization.k8s.io/cert-manager-edit created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrole.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
role.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
role.rbac.authorization.k8s.io/cert-manager:leaderelection created
role.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
rolebinding.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
service/cert-manager created
service/cert-manager-webhook created
deployment.apps/cert-manager-cainjector created
deployment.apps/cert-manager created
deployment.apps/cert-manager-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
```

2. Create a ClusterIssuer resource that defines the issuer you want to use to issue your SSL/TLS certificates. Cert-manager supports a variety of issuers, including Let's Encrypt. Here's an example ClusterIssuer resource that uses the Let's Encrypt staging environment:

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
```

```
% kubectl get secret -A
NAMESPACE      NAME                                            TYPE                             DATA   AGE
cert-manager   cert-manager-webhook-ca                         Opaque                           3      34m
cert-manager   letsencrypt-staging                             Opaque                           1      15s
...
```

In this example, the ClusterIssuer is named letsencrypt-staging and specifies that the Let's Encrypt staging environment should be used to issue certificates. You'll need to replace your-email@example.com with your own email address.

3. Create a Certificate resource that defines the SSL/TLS certificate you want to use for your ingress resources. 

You'll need to specify your FQDN (nginxkic.kushikimi.xyz) in the spec section of your Certificate resource, Here's an example Certificate resource:

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-tls
spec:
  secretName: example-tls-secret
  duration: 2160h # 90 days
  renewBefore: 360h # 15 days
  commonName: nginxkic.kushikimi.xyz
  dnsNames:
    - nginxkic.kushikimi.xyz
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

```
% kubectl get secret nginxkic-tls-secret -o yaml
apiVersion: v1
data:
  tls.crt: ...=
  tls.key: .....=
kind: Secret
metadata:
  annotations:
    cert-manager.io/alt-names: nginxkic.kushikimi.xyz
    cert-manager.io/certificate-name: nginxkic-tls
    cert-manager.io/common-name: nginxkic.kushikimi.xyz
    cert-manager.io/ip-sans: ""
    cert-manager.io/issuer-group: ""
    cert-manager.io/issuer-kind: ClusterIssuer
    cert-manager.io/issuer-name: letsencrypt-staging
    cert-manager.io/uri-sans: ""
  creationTimestamp: "2023-03-04T09:04:37Z"
  labels:
    controller.cert-manager.io/fao: "true"
  name: nginxkic-tls-secret
  namespace: default
  resourceVersion: "346127"
  uid: b3e06662-f83c-4a29-b0ff-a32f3ecaf063
type: kubernetes.io/tls


```
In this example, the commonName field specifies the primary domain name for which you're requesting the SSL/TLS certificate (in this case, nginxkic.kushikimi.xyz), and the dnsNames field specifies any additional domain names you want to include in the certificate.

Note that if you're using Let's Encrypt as your issuer, you'll need to make sure that your domain name is publicly accessible from the internet and that your DNS records are configured correctly. Let's Encrypt uses the ACME protocol to verify domain ownership, and it does so by checking that a specific challenge file can be accessed at a specific URL under your domain name. Cert-manager will automatically create the necessary Kubernetes resources (such as a Challenge resource and an ACME solver) to complete this verification process.

4. To use the SSL/TLS certificate generated by Cert-manager in your Nginx ingress controller, you'll need to add some annotations to your Ingress YAML file to configure the ingress controller to use the certificate. Here's an example YAML file:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - nginxkic.kushikimi.xyz
      secretName: nginxkic-tls-secret
  rules:
  - host: nginxkic.kushikimi.xyz 
    http:
      paths:
      - backend:
          service:
            name: nginx-dep
            port:
              number: 80
        path: /
        pathType: Prefix
status:
  loadBalancer: {}
```

Apply the above yaml file and notice that Ingress Controller is now handling HTTPS traffic on port 443 in addition to HTTP traffic on port 80.

```
 % kubectl get ingress
NAME           CLASS        HOSTS                        ADDRESS                                                                       PORTS     AGE
ingress        nginx        nginxkic.kushikimi.xyz       acf68105edd024bfe9fb72111124bfaf-32909998.ap-southeast-1.elb.amazonaws.com    80, 443   27h
ingress-plus   nginx-plus   nginxplusnic.kushikimi.xyz   a31ce143913004c26a8386cbbe71434b-176863799.ap-southeast-1.elb.amazonaws.com   80        22h
```

cert-manager reflects the state of the process for every request in the certificate object. 
You can view this information using the kubectl describe command:

```
% kubectl get certificate
NAME                  READY   SECRET                AGE
nginxkic-tls          True    nginxkic-tls-secret   10m
nginxkic-tls-secret   True    nginxkic-tls-secret   4m11s

% kubectl describe certificate nginxkic-tls
% kubectl describe certificate nginxkic-tls-secret
```

Do a browser test / curl test to verify: 
```
% curl -Sslvk -I https://nginxkic.kushikimi.xyz
*   Trying 18.136.236.130:443...
* Connected to nginxkic.kushikimi.xyz (18.136.236.130) port 443 (#0)
* ALPN: offers h2
* ALPN: offers http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
* (304) (IN), TLS handshake, Server hello (2):
* (304) (IN), TLS handshake, Unknown (8):
* (304) (IN), TLS handshake, Certificate (11):
* (304) (IN), TLS handshake, CERT verify (15):
* (304) (IN), TLS handshake, Finished (20):
* (304) (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / AEAD-AES256-GCM-SHA384
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=nginxkic.kushikimi.xyz
*  start date: Mar  4 08:04:36 2023 GMT
*  expire date: Jun  2 08:04:35 2023 GMT
*  issuer: C=US; O=(STAGING) Let's Encrypt; CN=(STAGING) Artificial Apricot R3
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multiplexing
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* h2h3 [:method: HEAD]
* h2h3 [:path: /]
* h2h3 [:scheme: https]
* h2h3 [:authority: nginxkic.kushikimi.xyz]
* h2h3 [user-agent: curl/7.86.0]
* h2h3 [accept: */*]
* Using Stream ID: 1 (easy handle 0x7fbb3f00bc00)
> HEAD / HTTP/2
> Host: nginxkic.kushikimi.xyz
> user-agent: curl/7.86.0
> accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 200 
HTTP/2 200 
< date: Sat, 04 Mar 2023 09:20:00 GMT
date: Sat, 04 Mar 2023 09:20:00 GMT
< content-type: text/html
content-type: text/html
< content-length: 615
content-length: 615
< last-modified: Tue, 13 Dec 2022 15:53:53 GMT
last-modified: Tue, 13 Dec 2022 15:53:53 GMT
< etag: "6398a011-267"
etag: "6398a011-267"
< accept-ranges: bytes
accept-ranges: bytes
< strict-transport-security: max-age=15724800; includeSubDomains
strict-transport-security: max-age=15724800; includeSubDomains
```
