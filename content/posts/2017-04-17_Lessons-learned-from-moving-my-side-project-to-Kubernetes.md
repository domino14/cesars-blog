---
title: Lessons learned from moving my side project to Kubernetes
description: >-
  This title may already be ringing some faint alarm bells; why move a small
  side project to use Kubernetes? And was it worth it?
date: '2017-04-17T03:31:45.119Z'
categories: []
keywords: []
tags: ["kubernetes"]
---

This title may already be ringing some faint alarm bells; why move a small side project to use Kubernetes? And was it worth it?

I run a small study site for tournament Scrabble players, available at [https://www.aerolith.org](https://www.aerolith.org). (Please excuse the lack of a nice landing page). It consists of a Django app, with PostgreSQL, and a small Go-based microservice that generates some complex word challenges on demand. The reason that service uses Go instead of Python had mainly to do with performance, but partially I was also experimenting with Go at the time. The site also uses SQLite to store the word databases, Gunicorn, Nginx, i.e. a standard Django setup. The site gets a couple hundred players a day, so it’s not that tiny. I have also used this platform to learn; the very first version of Aerolith for the web was completed in 2011 and written in ghastly Javascript. It is now a fairly modern app with React.js and ES6. So it’s a nice project because I get to tinker with technologies that I sometimes end up using at my day job (for example, I did some Redis/Sock.js stuff briefly in the past, as well as Django upgrades, etc).

![](https://cdn-images-1.medium.com/max/800/1*9eSD3uWziVWGjvCrHUTSlA.gif)

Since we use Docker heavily at [work](https://www.appinsights.com), I had taken the time a while ago to move over my app to use Docker. I like Docker because of how easy it is to encapsulate dependencies; before it, deployment was kind of ugly — `git pull` inside my VM, `pip install` whenever I changed any packages, upgrading was an issue etc etc. However all I did was make a docker-compose file and the process was still more or less the same, involving a git pull, mounting the code directory into the container, and the worst part is that changing any environment variable or dependency requires downtime (to rebuild the container). So in a way, this was worse than the previous virtualenv-based approach, even though during local development it’s a pretty solid experience. Since I have limited free time, this has been the setup for many months now and I keep forgetting and breaking the app whenever I upgrade some package.

In any case, I resolved to fix it sometime last year and started working on an HAProxy-based approach. The plan was to build two copies of a container, reverse proxy to them with Nginx, and during a code update, interact with the HAProxy API and change IPs, etc. Because of the real-time nature of the main Aerolith game, I really wanted to have a zero-downtime deploy, and this was actually surprisingly difficult — more on that later. While starting to work on this HAProxy approach, I realized that I probably needed some sort of service discovery and a data store to keep track of the last version of a container that was active, in order to survive random restarts, for example. Realizing that I was reinventing the wheel, I started looking into Docker Swarm.

I briefly played with Docker Swarm and it somehow corrupted my DigitalOcean VM. I can’t really remember the failure mode, but this was also pretty early on (maybe in July-August of 2016) and I gave up on it for a while and focused on my React.js frontend again. We use AWS ECS at work, and I didn’t want to move away from my cheapo $5 DigitalOcean instances, plus I didn’t like a few things about ECS and wanted to learn possible replacements.

The first time I heard of Kubernetes was from this funny [CircleCI article](https://circleci.com/blog/its-the-future/). I remember thinking at the time that I had never heard of it and would never use such an arcane technology, as it’s gotta be overkill, right? But more and more I kept hearing from Hacker News how Kubernetes was the future, and despite [evidence to the contrary](https://www.youtube.com/watch?v=PivpCKEiQOQ), containerization was here to stay. So I installed Minikube locally and started tinkering, and was able to get a fully functioning setup after a few hours. The Kubernetes.io documentation is pretty good.

Now, for some lessons I learned while setting all this up.

#### 1\. Running Minikube is not the same as running a cluster

Actually, my cluster setup right now is not ideal at all. But since this is a side project, all I did was use `kubeadm` , which tells you not to use it on production. Also, my cluster currently consists of one node :)

The Kubernetes.io documentation explains how to setup a kubeadm cluster and it’s actually pretty easy. I also tried conjure-up and Juju and could not get anything functioning. YMMV; I may not have tried hard enough :/

By default, kubeadm does not enable the batch v2 API that allows for CronJobs. If anyone knows how to do this, I’d appreciate any details. It seemed too difficult to figure out, and I have a daily job that I run with `kubectl run` in a crontab instead.

#### 2\. Ingress is kind of confusing

By default, an Ingress does literally nothing. You need to run an “Ingress controller” alongside it, but there are no selectors in the controller manifest file for the ingress by default. It’s kind of strange since it seems like everything else uses selectors (services/deployments/replication controllers/etc).

Finding the right ingress controller is also kind of difficult. I ended up using the one created by [nginxinc](https://github.com/nginxinc/kubernetes-ingress), despite their shameless self-promotion for their Plus version all over the docs. The Kubernetes project itself also has a more [complex Ingress controller](https://github.com/kubernetes/ingress/tree/master/examples/deployment/nginx), but I keep getting occasional SSL errors when I run ApacheBenchmark on it. Anyone know why?

#### **3\. Zero-downtime deploy is NOT the default in Kubernetes**

See this long thread: [https://github.com/kubernetes/contrib/issues/1140](https://github.com/kubernetes/contrib/issues/1140)

Basically, if you have a deployment, and you update the container to a new version, this is what happens in parallel (paraphrased from a comment on that thread):

1.  Kubelet sends a SIGTERM to the pod
2.  A controller removes the pod from the endpoints
3.  kube-proxy removes the Pod from virtual LBs

So `gunicorn`by default will start rejecting connections once it receives a SIGTERM, and wrap up the rest of its connections gracefully. The problem is that for a little bit of time, the controller is still directing traffic to this dying gunicorn, and this results in nginx returning a 502. I had to come up with the following hack. This can probably be made better, but it works for now:

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /opt/webolith/djAerolith/test_requirements.txt
  successThreshold: 1
  failureThreshold: 2
  periodSeconds: 5
  initialDelaySeconds: 5

lifecycle:
  preStop:
    exec:
      # A hack to try to get 0% downtime during deploys. This should
      # help ensure k8s eventually stops giving this node traffic.
      command: ["sh", "-c", "rm /opt/webolith/djAerolith/test_requirements.txt && sleep 20"]
```

Note the initialDelay is required because it takes gunicorn a little bit of time to get ready, and the `sleep 20` ensures that gunicorn has some time to get taken off of the endpoints / virtual load balancers. In a way, the SIGTERM that gets sent doesn’t even matter.

I basically tested the behavior with something like `ab -n 1500 -c 15 [https://minikube.aerolith.org/healthz/](https://minikube.aerolith.org/healthz/)` until I was able to reliably get zero errors during an image update.

Note also that if you only have one replica running (I have a single node right now) you must also set this strategy:

```yaml
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxUnavailable: 0
```

This makes it so there’s always at least 1 replica running. At some point I forgot to add this into prod and I spent a while trying to figure out what was going on and why it was broken, nearly giving up again until I took a careful look at the output of `watch kubectl get po` while doing an image update.

#### 4\. WTF is RBAC?

So there I am about to deploy my project to production when Kubernetes 1.6.0 decides to come out, and suddenly my ingress controller stops working and I can’t downgrade back to 1.5.

I was able to piece together some additions to my ingress controller based on a couple of Github issues and some reading of the documentation. For a while I got very discouraged and almost went back to reinventing the wheel, or even scrapping Docker altogether and going back to `git pull && kill -HUP <gunicorn PID>` before I remembered that I am obstinate and that the Sunk Cost Fallacy is a real thing. Anyway, here are my changes:

[**domino14/Webolith**
_Webolith - Aerolith 2.0 - Aerolith for the web. A word study site - study for Scrabble, Boggle, Words With Frentz, etc._github.com](https://github.com/domino14/Webolith/blob/master/kubernetes/deploy-configs/nginx-ingress-rc.yaml "https://github.com/domino14/Webolith/blob/master/kubernetes/deploy-configs/nginx-ingress-rc.yaml")[](https://github.com/domino14/Webolith/blob/master/kubernetes/deploy-configs/nginx-ingress-rc.yaml)

Note all the RBAC (Role-Based Access Control) stuff early on. Hopefully the projects will update their documentation.

Again I am not a Kubernetes or ops master so maybe these are too permissive, or I’m somehow opening up my cluster to the world (I hope not). Comments welcome!

#### 5\. Kubernetes eats your resources

I could have sworn that there was some funny Hitler uses Docker video that specifically calls out that most of the resources in the VM were taken up by whatever orchestration framework was being used, but I can’t find it. In any case, the $5 DO instance is way too underpowered for the default `kubeadm` cluster, and even the $10 instance seemed to not have much memory or CPU left once all the Kubernetes containers are running (for the API Server, etcd, kubelet, DNS, …). So I decided my time was worth more than $20 a month and sprung for that instance. Still pretty cheap and donations should cover it ;)

![](https://cdn-images-1.medium.com/max/800/1*jF_XQRwWj8sXMDFmPXt5Ww.png)

#### 6\. No need to move everything over

I kept my dedicated $5 Postgres instance. Why mess with it? Hopefully it’ll remain stable for a long time. Also, locally I’m still using docker-compose for local development instead of minikube. Although minikube is nice enough, docker-compose is less friction (just install Docker, no extra dependencies).

#### 7\. How to automate

I use CircleCI to build the docker containers, push to Docker Hub, and also use kubectl from within CircleCI to run the various `apply -f` commands. The kubeconfig file is in a private repo. I generate the YAML files in CircleCI too, from a combination of Mustache-like templates (I call it Curlies: `CURLIES_RE = r'{{\s*(\w+)\s*}}'`) and just YAML. A lot of the config is stored in CircleCI environment variables.

Let’s Encrypt does my SSL. Right now it’s a pseudo-manual process. I may want to learn how to use `kube-lego` soon but I’m a little burned out on Kubernetes at the moment. Also, that’s only compatible with the official ingress controller, which I’m not using because of those occasional SSL handshake issues I saw.

Note that if you modify a TLS secret, this does not auto-reload the ingress. You must also modify something about the ingress. So I have a hack path that looks something like:

```
- path: {{ HACK_PATH }}
  backend:
    serviceName: nginx-static-svc
    servicePort: 80
```

The path ends up being something like `/hackpath-{UUID}` . Very janky, but I don’t know of another way that works better right now.

#### 8\. A couple of additional notes

*   I have two copies of Nginx running, one for the ingress controller, and one for the static file server. It may have been possible to combine them into one, but Nginx is pretty light and maybe this is OK. It does seem a bit silly, though.
*   The static files are built right into my Nginx static server. It would be nice if I could auto-detect if any static files were modified to avoid building multiple copies of the same code, though…
*   Alpine images are small, use them!
*   It’s easy to accidentally commit all sorts of secrets to Github and/or Dockerhub. Especially the latter doesn’t respect ignore rules in the same way as git (i.e. `*.yaml` and `**/*.yaml` are different things in a `.dockerignore` file.)
*   All of this was manually done with `kubeadm` and I have 1 replica of everything. If that node goes down I have to rebuild it. It’s still zero-downtime deploy as long as DigitalOcean cooperates 😊.
*   (Post-mortem) New versions of my requirements are causing a few sporadic errors not caught by testing. Going to have to look into it later.

Anyway, now that I finally have a zero-downtime deploy story, may I never have to touch this setup again, and I can start working on adding multiplayer mode. Yay! And in the end it was worth it; deploying a new container is now as simple as pushing to master.
