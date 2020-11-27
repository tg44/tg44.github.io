author_profile: true
read_time: true
comments: null
share: false
categories:
  - DevOps
tags:
  - mqtt
  - mosquitto
  - ssl/tls
  - k8s
  - kubernetes
  - hosting
---

This is a post I looked for in the last week... I just wanted a secure mqtt on my k8s cluster :(

## Things I had

I moved from scaleway to digital ocean. I started a managed cluster there, and started to learn some k8s. Also, I needed a cloud facing mqtt server.

Most of my tutorial will be cloud provider agnostic, but I could be there some DO specific things.

I had an nginx-inngress installed by DO, also I requested a load balancer from them.

I have kubeadmin installed, to modify yamls more easily (also this was already in the default DO managed cluster).

I had a properly installed [cert-manager](https://github.com/jetstack/cert-manager).

I had an `mqtt.home1.example.com` like domain pointed to the load balancer.

I started a big yaml and `kubectl apply -f mqtt.yaml`-ed it every time when I don't needed to patch an already created file manually.

## Open the gates

First things first, I wanted to open a port on my loadbalancer. I went to the services menu in kubeadmin, found the service which had external endpoints (`nginx-ingress-ingress-nginx-controller`), and added a new port block;
```yaml
spec:
  ports:
    - name: mqtt
      protocol: TCP
      port: 1883
      targetPort: 1883
```

I read some [documentations](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/) and created a new ConfigMap;
```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  '1883': 'home1/mosquitto-service:1883'
```
And also created a new namespace and added the mosquitto service to it;
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: home1
---
apiVersion: v1
kind: Service
metadata:
  name: mosquitto-service
  namespace: home1
spec:
  selector:
    app: mosquitto
  ports:
    - protocol: TCP
      port: 1883
      targetPort: 1883
  type: ClusterIP
  sessionAffinity: None
```

(You can change the names and namespaces as you wish. I used the home1 namespace.)

Lastly, I needed to edit the nginx-ingress-ingress-nginx-controller Deployment, 
and add the `- '--tcp-services-configmap=ingress-nginx/tcp-services'` line to the `containers/args` list.

After these modifications my nginx-ingress and load balancer 
was ready to relay tcp:1883 to whatever app I write to 
the mosquitto service app selector!

## Start and test a simple mqtt config

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-conf
  namespace: home1
data:
  mosquitto.conf: |-
    allow_anonymous true
    log_dest stdout
    listener 1883
    protocol mqtt
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  namespace: home1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
      - name: mosquitto
        image: eclipse-mosquitto
        imagePullPolicy: Always
        resources:
          limits:
            cpu: 0.5
            memory: 500Mi
        ports:
        - containerPort: 1883
        volumeMounts:
        - name: config
          mountPath: /mosquitto/config/
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: mosquitto-conf
          items:
          - key: mosquitto.conf
            path: mosquitto.conf
```
After applying this yaml, we should connect to the loadbalancer:1883 from any mqtt client.

I wrote config maps for the first time, so it was some rounds of google until I was confident enough to apply it, but it worked for the first time :)

## Reload and certs

The two biggest problems starts here (btw opening the ports was not so easy either), because there are no prewritten tutorial about what works and how.
We need certificates, and also when the certificates are claimed we need to redeploy the app with the new certs.

For the second problem I installed [Reloader](https://github.com/stakater/Reloader) with;
```bash
kubectl create namespace reloader
helm repo add stakater https://stakater.github.io/stakater-charts
helm repo update
helm install --namespace=reloader reloader stakater/reloader
```

I already had [cert-manager](https://github.com/jetstack/cert-manager) installed at this point.

I also had some problems with the cafile, so I added it as a config-map after I downloaded it from the [let's encrypt site](https://letsencrypt.org/certs/trustid-x3-root.pem.txt).

The full yaml;
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: mosquitto-cert
  namespace: home1
spec:
  secretName: mosquitto-cert-tls
  renewBefore: 48h
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: mqtt.home1.example.com
  dnsNames:
  - mqtt.home1.example.com
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-conf
  namespace: home1
data:
  mosquitto.conf: |-
    allow_anonymous true
    log_dest stdout
    listener 1883
    protocol mqtt
    cafile /mosquitto/tls-ca/trustid-x3-root.pem
    certfile /mosquitto/tls-server/tls.crt
    keyfile /mosquitto/tls-server/tls.key
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: trusted-x3-root-cert
  namespace: home1
data:
  trustid-x3-root.pem: |-
    -----BEGIN CERTIFICATE-----
    MIIDSjCCAjKgAwIBAgIQRK+wgNajJ7qJMDmGLvhAazANBgkqhkiG9w0BAQUFADA/
    MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMT
    DkRTVCBSb290IENBIFgzMB4XDTAwMDkzMDIxMTIxOVoXDTIxMDkzMDE0MDExNVow
    PzEkMCIGA1UEChMbRGlnaXRhbCBTaWduYXR1cmUgVHJ1c3QgQ28uMRcwFQYDVQQD
    Ew5EU1QgUm9vdCBDQSBYMzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
    AN+v6ZdQCINXtMxiZfaQguzH0yxrMMpb7NnDfcdAwRgUi+DoM3ZJKuM/IUmTrE4O
    rz5Iy2Xu/NMhD2XSKtkyj4zl93ewEnu1lcCJo6m67XMuegwGMoOifooUMM0RoOEq
    OLl5CjH9UL2AZd+3UWODyOKIYepLYYHsUmu5ouJLGiifSKOeDNoJjj4XLh7dIN9b
    xiqKqy69cK3FCxolkHRyxXtqqzTWMIn/5WgTe1QLyNau7Fqckh49ZLOMxt+/yUFw
    7BZy1SbsOFU5Q9D8/RhcQPGX69Wam40dutolucbY38EVAjqr2m7xPi71XAicPNaD
    aeQQmxkqtilX4+U9m5/wAl0CAwEAAaNCMEAwDwYDVR0TAQH/BAUwAwEB/zAOBgNV
    HQ8BAf8EBAMCAQYwHQYDVR0OBBYEFMSnsaR7LHH62+FLkHX/xBVghYkQMA0GCSqG
    SIb3DQEBBQUAA4IBAQCjGiybFwBcqR7uKGY3Or+Dxz9LwwmglSBd49lZRNI+DT69
    ikugdB/OEIKcdBodfpga3csTS7MgROSR6cz8faXbauX+5v3gTt23ADq1cEmv8uXr
    AvHRAosZy5Q6XkjEGB5YGV8eAlrwDPGxrancWYaLbumR9YbK+rlmM6pZW87ipxZz
    R8srzJmwN0jP41ZL9c8PDHIyh8bwRLtTcm1D9SZImlJnt1ir/md2cXjbDaJWFBM5
    JDGFoqgCWjBH4d1QB7wCCZAA62RjYJsWvIjJEubSfZGL+T0yjWW06XyxV3bqxbYo
    Ob8VZRzI9neWagqNdwvYkQsEjgfbKbYK7p2CNTUQ
    -----END CERTIFICATE-----

---
apiVersion: v1
kind: Service
metadata:
  name: mosquitto-service
  namespace: home1
spec:
  selector:
    app: mosquitto
  ports:
    - protocol: TCP
      port: 1883
      targetPort: 1883
  type: ClusterIP
  sessionAffinity: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  namespace: home1
  annotations:
    reloader.stakater.com/auto: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
      - name: mosquitto
        image: eclipse-mosquitto
        imagePullPolicy: Always
        resources:
          limits:
            cpu: 0.5
            memory: 500Mi
        ports:
        - containerPort: 1883
        volumeMounts:
        - name: config
          mountPath: /mosquitto/config/
          readOnly: true
        - name: tls-ca
          mountPath: /mosquitto/tls-ca
          readOnly: true
        - name: tls-server
          mountPath: /mosquitto/tls-server
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: mosquitto-conf
          items:
          - key: mosquitto.conf
            path: mosquitto.conf
      - name: tls-ca
        configMap:
          name: trusted-x3-root-cert
          items:
          - key: trustid-x3-root.pem
            path: trustid-x3-root.pem
      - name: tls-server
        secret:
          secretName: mosquitto-cert-tls
```

At this point I could connect to the `mqtt.home1.example.com` on the 1883 port with [MQTT Explorer](http://mqtt-explorer.com/)! (To reach this point I had about 3 hours trial and error, and also 2 days of googleing in this project so feel blessed that sb found out the ways before you ;) )

Last but not least, you should add authentication as described in [this blogpost](http://www.steves-internet-guide.com/mqtt-username-password-example/). After you created the password file you can add it as a config map, and mount as shown before.

## Other sources;
 - exposing tcp on nginx-ingress; https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/
 - DO mosquitto with let's encrypt; https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-the-mosquitto-mqtt-messaging-broker-on-ubuntu-18-04-quickstart
 - random "help" from 2015; https://mosquitto.org/blog/2015/12/using-lets-encrypt-certificates-with-mosquitto/
 - services and labels tutorial; https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/
 - medium post which opened my eyes about .crt and .key; https://medium.com/cloudnesil/securing-mqtt-broker-on-kubernetes-5503420b84a8
 - a good tutorial about mqtt on k8s with helm; https://gbaeke.gitbooks.io/open-source-iot/content/chapter1.html
 - cert-manager usage docs; https://cert-manager.io/docs/usage/certificate/
 
