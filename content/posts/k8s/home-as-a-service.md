---
layout: post
title: "Home-as-a-service"
date: 2021-03-24
polly: https://madeddu.xyz/mp3/k8s/home-as-a-service.mp3
categories:
- k8s
- aws
- cloudflare
- traefik
tags:
- network
- proxy
- auth
---

### Prelude

Today I'm gonna talk about how you can leverage [Traefik2](https://doc.traefik.io/traefik/v2.0/), [k3s](https://k3s.io/), [Cloudflare](https://www.cloudflare.com/), and [RaspberryPI](https://www.raspberrypi.org/) to get the best out of your...apartment. After the image, I'll go through the setup of everything needed! 

<div class="img_container"><img src="https://i.imgur.com/ikcGSO9.png" style="width: 100%; marker-top: -10px;"/></div>

### Why Traefik2

For those of you who never heard the word *Traefik* before, here it is:

> Traefik is an open-source Edge Router that makes publishing your services seamless. 

I already made some experiments in the past (look at my two-pieces series [I run VSC in the browser and I am just fine - Part I](https://madeddu.xyz/posts/reposted/immutable-vsc/) and [My team run VSC in the browser and they are just fine - Part II](https://madeddu.xyz/posts/aws/cdk/serverless-ocr/)) and I've always been fascinated by the way Traefik *just works*. And... it's written in Golang! Since I recently had to deal a lot with SSO and k8s (I finally made my raspberry cluster with pi4 and k3s, a more specific blog post is coming), I wanted to extend my experience with Traefik moving to the 2nd version of the tool and leveraging `IngressRoute`. Moreover, I used Google and Traefik2 `Middleware` to forward requests and authenticate them using my Google account. The cool thing is that if you already own a domain, a Google account, and a k3s cluster (both running *on-premise* at home and in cloud if they didn't end up *fired*), you can get everything described in this article for free! 🥳

The *high-level* schema of how this is implemented is shown in the diagram below:

<div class="img_container"><img src="https://i.imgur.com/O8b4cX4.png" style="width: 100%; marker-top: -10px;"/></div>

When the user visits app1.mydomain.com, which is behind your Traefik, the request get's redirected to Google OAuth, where the user signs in. If the authorization is successful, the user will be redirected back to Traefik on `auth.mydomain.com/_oauth`. This service then checks based on the cookie to what application the request belongs to and redirects it. If the user is correctly authenticated, the request complete. Traefik will take care of redirect the request to the specific service pointed by the respective `IngressRoute`. The service will balance the request across the pods belonging to the specific app deployment. At the end of the article, I will show a version of the diagram above with `Traefik` exploded to graphically expose what will happen when we'll finish the setup.

As you can see, you can actually reverse-proxy many different services (like Homeassistant or Homebridge for your own home-automation, ndr) using third-level domain and auth them through Google login as you usually do with many other apps that support Google as an external Identity Provider. In the next section, I will go through many steps to get everything working:

- Step 1: Setup your cluster using k3s
- Step 2: Own a domain
- Step 3: Own a Cloudflare account
- Step 4: Nameserver Switch
- Step 5: Setup your Traefik
- Step 6: Setup your Cloudflare secrets
- Step 7: Install Traefik2
- Step 8: Setup your Google App for SSO Login
- Step 9: Setup your Traefik SSO Forwarder
- Step 10: Setup your first route

The first thing to is... setup your cluster!

### Step 1: Setup your cluster using k3s

I'm not a Kubernetes expert, so I decided to go through the simplest way to install Kubernetes - *at least* the simplest I was able to follow without too many troubles. I'm talking about [k8s](https://k3s.io/). K3s is a highly available, certified Kubernetes distribution designed for production workloads in unattended, resource-constrained, remote locations or inside IoT appliances. You can install it pretty easily by following one of the - literally - one thousand tutorial you can find in the web (like [this one](https://ikarus.sg/kubernetes-with-k3s/) - not sure, it's late I'm sorry). In any case, one important thing to do is to avoid the installation of Traefik by providing the `--no-deploy traefik` flag:

```curl -sfL https://get.k3s.io | sh - --no-deploy traefik```

For the operating system, I used Ubuntu server. If you want, you can even use an Ansible Playbook (like [this](https://github.com/itwars/k3s-ansible) - many thanks [https://github.com/michelangelomo](@michelangelomo) for the insight) to go straightforward through the installation setup without getting old doing thing manually. I did manually because Ansible hates me.

### Step 2: Own a domain

If you read some of my old articles, you know I use AWS both at work and for my personal purpose: I bought my domain `madeddu.services` using the Register by AWS but feel free to buy a domain wherever you want.

### Step 3: Own a Cloudflare account

This is not *strictly required* but *strongly suggested*: in fact, using Cloudflare you can leverage so many cool things for free, including owning a Cloudflare account. Since I used Cloudflare as the DNS provider, you can register an account for using it and follow the next step, but if you already know how to setup your own provider, you can jump to Step 5/6.

### Step 4: Nameserver switch

You can setup your zone in Cloudflare by specifying your domain - following the guidelines on the site: if you want to use Cloudflare Nameserver, you have to manually switch Nameserver. If your domain is registered in AWS, keep in mind that you have to go into the Registar Page of AWS and switch nameserver from the right upper corner (usually AWS provides 4 nameservers and Cloudflare 2). Switching them from inside the hosted zone is not sufficient. The image below is shown how the two interfaces look like:

<div class="img_container"><img src="https://i.imgur.com/LpEWF68.png" style="width: 100%; marker-top: -10px;"/></div>

The upper part shows the Registered Domain menu (inside Route53) where you have to switch Nameserver. The second part shows the Hosted Zone (new interface).

### Step 5: Setup your Traefik

To install Traefik, I used [Helm](https://helm.sh/) - a package manager for Kubernetes. After installing it, you can add the repo by running:

```
helm repo add traefik https://containous.github.io/traefik-helm-chart
```

Since I used Cloudflare as the DNS provider, the next steps will assume Cloudflare setup mentioned before but you can use any of the [let's encrypt supported providers](https://doc.traefik.io/traefik/v2.0/https/acme/).
Before doing your first Traefik release, you have to create a `values.yml` file: in this file, I specified the service type to be `ClusterIP`, and the EntryPoints to route 80,443 and admin dashboard (it could be *dangerous* if you want you can remove it).

{{< highlight yaml >}}
service:
  type: ClusterIP
# Configure ports
ports:
  # The name of this one can't be changed as it is used for the readiness and
  # liveness probes, but you can adjust its config to your liking
  traefik:
    port: 9000
    expose: true
    protocol: TCP
    exposedPort: 9000
  web:
    port: 8000
    expose: true
    protocol: TCP
    exposedPort: 80
  websecure:
    port: 8443
    expose: true
    protocol: TCP
    exposedPort: 443
additionalArguments:
  - "--certificatesresolvers.default.acme.email=<YOUR_EMAIL>"
  - "--certificatesresolvers.default.acme.storage=/data/acme.json"
  - "--certificatesresolvers.default.acme.caserver=https://acme-v02.api.letsencrypt.org/directory"
  - "--certificatesResolvers.default.acme.dnschallenge=true"
  - "--certificatesResolvers.default.acme.dnschallenge.provider=cloudflare"
  - "--api.insecure=true"
  - "--accesslog=true"
  - "--log.level=INFO"
env:
  - name: CF_DNS_API_TOKEN
    valueFrom:
      secretKeyRef:
        name: cloudflare
        key: dns-token
{{< / highlight >}}

Since my domain is registered on AWS, I have a certificate created using [ACM](https://aws.amazon.com/certificate-manager/) so I don't *own* my certificate: however, with EntryPoints defined we can extend the chart to use Let's Encrypt as a certificate resolver and automatically generate and renew ACME certificates for my domain using DNS Challenge. To accomplish this, Traefik2 needs to access your Zone in Cloudflare: that's the secret part you see under the `env` section of the YAML.

### Step 6: Setup your Cloudflare secrets

In order for Let's Encrypt provider to use Cloudflare, an API Token with DNS:Edit permissions is required. Thus, log in your Cloudflare Account and under API Tokens section of your domain, you should find Create Token. Use the Edit zone DNS template or a custom token and give two permissions: `Zone DNS Edit`, and `Zone Zone Read`. The status of the token should be `Active` right afterward.

You can now create your own secret in kubernetes by doing:

```kubectl create secret generic cloudflare --from-literal=dns-token=<YOUR_CF_TOKEN>```

### Step 7: Install Traefik2

You can now finally install traefik using Helm just by running:

```helm install traefik traefik/traefik -f values.yml```

After this step, if you run:

```kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000```

you should see the traefik dashboard. Well done! 🥳🥳🥳

### Step 8: Setup your Google App for SSO Login

Before going ahead with the amazing [Traefik Forward Auth](https://github.com/thomseddon/traefik-forward-auth) deploy, you have to setup your Google account to provide you auth mechanism. First, visit the Google Developer Console and create a new project. On the main page, select Credentials → Create Credentials → OAuth client ID. If requested, choose as application name something meaningful like Traefik SSO and specify in the Authorized Domain section your domain (`mydomain.com` in the first picture, `madeddu.services` in my scenario). You should be able to get your OAuth client ID in the Credentials tab: you can select Web application, enter a name, leave Authorized JavaScript origins blank and enter your domain - including the /_oauth path. After that, you should have your Google ClientID and Secret. 

Once again, we need to create a kubernetes Secret containing the acquired info from Google:

```
kubectl create secret generic traefik-sso --from-literal=clientid=<YOUR_GOOGLE_CLIENT_ID>.apps.googleusercontent.com --from-literal=clientsecret=<YOUR_GOOGLE_SECRET> --from-literal=secret=<A_RANDOM_SECRET>
```

where the latest parameter is a random string. We are ready to deploy our Traefik SSO forwarder!

### Step 9: Setup your Traefik SSO Forwarder

To authenticate with Traefik, we are gonna use two resources defined as [CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) - you have accomplished this step by deploying the Helm chart of Traefik2 in Step 7. The resources I'm talking about are the `Middleware` and the `IngressRoute`. The forwarder is available at [https://github.com/thomseddon/traefik-forward-auth](https://github.com/thomseddon/traefik-forward-auth). The repo provides a minimal forward authentication service that provides OAuth/SSO login and authentication for the Traefik reverse proxy/load balancer. A Helm chart is provided as well in the example folder, but I prefer to install it manually to show you how simple it is. Create a file with the following content:

{{< highlight yaml >}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: traefik-sso
  labels:
    app: traefik-sso
spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik-sso
  template:
    metadata:
      labels:
        name: traefik-sso
        app: traefik-sso
    spec:
      containers:
      - name: traefik-sso
        image: thomseddon/traefik-forward-auth:2.2-arm64|<ANY_OTHER_TAG_SUPPORTED_BY_YOUR_ARCH>
        imagePullPolicy: Always
        env:
        - name: PROVIDERS_GOOGLE_CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: traefik-sso
              key: clientid
        - name: PROVIDERS_GOOGLE_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: traefik-sso
              key: clientsecret
        - name: SECRET
          valueFrom:
            secretKeyRef:
              name: traefik-sso
              key: secret
        - name: COOKIE_DOMAIN
          value: <YOUR_DOMAIN>
        - name: AUTH_HOST
          value: auth.<YOUR_DOMAIN>
        - name: INSECURE_COOKIE
          value: "false"
        - name: WHITELIST
          value: <YOUR_EMAIL>
        - name: LOG_LEVEL
          value: debug
        ports:
        - containerPort: 4181
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-sso
spec:
  selector:
    app: traefik-sso
  ports:
  - protocol: TCP
    port: 4181
    targetPort: 4181
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: sso
spec:
  forwardAuth:
    address: http://traefik-sso:4181
    authResponseHeaders: 
        - "X-Forwarded-User"
    trustForwardHeader: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-sso
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`auth.<YOUR_DOMAIN>`)
    kind: Rule
    services:
    - name: traefik-sso
      port: 4181
    middlewares:
      - name: default-sso@kubernetescrd
  tls:
    certResolver: default
    domains:
      - main: "*.<YOUR_DOMAIN>"
    options: {}
{{< / highlight >}}

In this file, there's a `Deployment` of the forwarder - that use the Google secrets created before, a `Service` to expose the deployment, a `Middleware` to forward a request to the service, and finally an `IngressRoute` with the certResolver we defined before during our Traefik setup with Helm. The overall flow should be clear, but for the sake of simplicity I created a closing picture with the Traefik part exploded to show what we have actually deployed:

<div class="img_container"><img src="https://i.imgur.com/KBlGj0a.png" style="width: 100%; marker-top: -10px;"/></div>

### Step 10: Setup your first route

We have everything ready to test our setup: we only need to create our first `IngressRoute` - meaning, a route pointing to a service. If you don't want to deploy the standard `whoami` test deployment, you can actually expose your Traefik dashboard through the Forwarder. Let's create a new file called `dashboard-ingressroute.yml`:

{{< highlight yaml >}}
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-sso-dashboard
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`traefik.madeddu.services`) && (PathPrefix(`/dashboard`) || PathPrefix(`/api`))
    kind: Rule
    services:
    - name: traefik
      port: 9000
    middlewares:
      - name: default-sso@kubernetescrd
  tls:
    certResolver: default
    domains:
      - main: "*.madeddu.services"
    options: {}
{{< / highlight >}}

Then, you can apply it by doing:

```
kubectl apply -f dashboard-ingressroute.yml
```

... and you're finally done. Clap clap clap! If you arrived here, many thanks first of all, and if you didn't become crazy - I did so... please forgive any mistakes and let me know if you had some troubles in your setup.

### One More Thing...

We actually need one more thing... we need to define a CNAME for our services/home. In my case, I have a third-level domain updated automatically by my router that points to my public IP. If you have a fixed public IP, lucky you: you can create a CNAME in Cloudflare pointing to it. Then, create a CNAME for any new `IngressRoute` you want to deploy in your cluster - starred CNAME are not supported by free account, at least if you want to proxy them with Cloudflare, that is a good choice to leverage Cloudflare protection out of the box. Also, don't forget to port-forward the exposed port to your cluster IP :) 

### Conclusion

The next step is to explore any service that you would like to run into your cluster, expose it through your Traefik, and authenticate them safely using your Google Account!

Have fun and stay safe! 🖖