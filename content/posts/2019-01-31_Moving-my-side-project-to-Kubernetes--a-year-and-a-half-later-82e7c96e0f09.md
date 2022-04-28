---
title: 'Moving my side project to Kubernetes, a year and a half later'
description: >-
  Around April of 2017 I wrote this article about moving my side project to a
  single-node Kubernetes cluster…
date: '2019-01-31T01:54:09.309Z'
categories: []
keywords: []
tags: ["devjournal"]
slug: >-
  /@14domino/moving-my-side-project-to-kubernetes-a-year-and-a-half-later-82e7c96e0f09
---

Around April of 2017 I wrote this article about moving my side project to a single-node Kubernetes cluster: [https://hackernoon.com/lessons-learned-from-moving-my-side-project-to-kubernetes-c28161a16c69](https://hackernoon.com/lessons-learned-from-moving-my-side-project-to-kubernetes-c28161a16c69)

As of today, the infrastructure is still running strong, although I’ve run into a few issues I will talk about later in this article. I initially set up my node as a $10/month node but it was barely not powerful enough. Since then, Digital Ocean seems to have roughly doubled the CPU/memory for each of its instances, so $10 might work out. The Kubernetes docs mention that your nodes should have at least 2 GB of RAM, which means a decent 3-node cluster would only run you about $30 a month. I am still on the $20/month node, but I’ve redeployed it a couple of times and resized it since the initial setup, for various reasons that I’ll go into here.

Now, some notes:

### 1\. kubeadm

kubeadm is cool. It seems to be in General Availability now, whereas it was something like alpha when I started messing around with it. I never had troubles with it before, and although I haven’t used it for a large cluster yet, it seems like it would do the job. Not really much more to say here, but I was just pleasantly surprised with how easy it is to bring up a Kubernetes cluster from scratch; I had to do so at least twice with this cluster, because I wanted to upgrade to the latest version to use features, and to update my instance to the latest Ubuntu LTS / use the newest $20 DigitalOcean nodes.

I never quite learned how to do a rolling upgrade of the Kubernetes API servers / etc, but since I had a single-node cluster anyway, and kubeadm is so easy to use, I just followed the [tutorial on the main Kubernetes site](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) on a brand-new node. I would then move my DNS over for zero-downtime.

### 2\. Multiplayer mode

A little segue into the importance of doing user research. A sizable number of players had wanted a multiplayer mode for the longest time, and over the course of several months (work nights / weekends) I was able to finally get something out there. And… barely anyone used it. It also made my infrastructure unstable. It’s not that the mode was bad, it may have needed to be more discoverable, who knows, but I eventually undid the changes and re-learned a valuable lesson in making sure to build what people actually want.

In particular I used Django Channels to handle the socket stuff, and there must have been some sort of memory leak, because over the course of a few weeks it would grow slowly in memory until it took over my cluster. I would have to keep restarting it and the behavior wouldn’t change. I did spend quite a while trying to figure out what was happening exactly, but I also regretted not just using straight websockets with Go and writing something I could actually understand.

In any case, once I undid the changes, the Kubernetes node became much more stable and I was much happier! And my users barely noticed… they still like the single-player mode a lot seemingly.

### 3\. SSL

Auto-renewing Let’s Encrypt SSL certificates is harder than it should be, but I eventually got something working with cert-manager by following their tutorial. It involves installing Helm and Tiller, and of course using the documented instructions to do so results in some sort of RBAC issue. Googling the exact issues and pasting whatever RBAC voodoo into the command line helped to fix this.

The only problem is that after 2 months my site would just suddenly have no SSL. I had to literally delete the Nginx Replication Controller and recreate it every time to bring SSL back. My pod specs and other yaml files seemed to be ok to the few people on the Kubernetes Slack who saw them, so who knows what the problem was? Note that 2 months is how long it takes for cert-manager to try renewing the Let’s Encrypt cert. It did succeed in renewing it, seemed to have put it in the right place, and so forth, but the cert wouldn’t be applied to my ingress.

Thinking I did something wrong, I took the opportunity to upgrade the cluster and used the latest Kubernetes Nginx Ingress Controller (as opposed to the one maintained by Nginx themselves, which was now way out of date). I am also using cert-manager’s “ingress-shim”, which automatically manages the creation of my certificates with just some annotations on the ingress itself. This actually seemed to work; my certificate was automatically created, renewed, and with no downtime!

But… [http://www.aerolith.org](http://www.aerolith.org) does not redirect to https. I filed an [issue](https://github.com/kubernetes/ingress-nginx/issues/3654), asked on the Kubernetes Slack, and no one knows why. More growing pains, I guess. It seems that most subpaths redirect properly (for example /accounts/login), and HSTS is on so that once the browser sees one https path it will do the redirect itself, so it’s not as big of a deal as it could be, but it’s still very annoying.

### 4\. Migrations

Very recently I realized I had no idea how to migrate my database! So I couldn’t add a new feature until I figured it out. The basics are simple; figure out how to run `./manage.py migrate` inside an updated pod. But the problem is that the updated pod won’t even get to run, since it will have the new database tables in its schema, will see that the database doesn’t match, and will kaputt.

Jobs to the rescue! I created a migration Job, and use it in my deploy process as such:

```python
local('kubectl delete --ignore-not-found=true job webolith-migrate')

local('kubectl apply -f webolith-migrate.yaml')

# Only proceed if the job was successful.

local('kubectl wait --for=condition=complete --timeout=30s '
      'job/webolith-migrate')

# continue with the rest of the deploy...
local('kubectl apply -f webolith-deployment.yaml')  # etc
```

Note, I use `kubectl wait` to wait for the migration job to be done before proceeding with the deploy. If there is nothing to migrate, the job just quits with a successful status code.

### 5\. Kubernetes certs

So a few months ago I was chillin’ when I got a message from a user telling me that the daily medals that my app hands out have not been awarded in a few days. I use a daily cleanup `Cronjob` for this. As a small segue, in my previous article I mentioned that Cronjobs were not, at the time, supported outside of alpha, and there was no `kubectl wait` until recently, or InitContainers (which I almost used for the migration job, until told that this wasn’t the right tool). So it’s nice how many useful features are constantly added to Kubernetes.

Anyway, I immediately tried logging into my node and all of my `kubectl` commands were failing. I couldn’t even see the status of my pods, yet miraculously they were still running (for how long? who knows?). It turns out my Kubernetes CA certificate had expired (or something like that). There is no mention of this anywhere in the docs that I had seen, but apparently they expire once a year, and you can’t connect to your cluster at all after that. So I had to Google how to renew them, ran the arcane command on the command line, and my Cronjobs started working again. It is surprising that Cronjobs would just give up and fail, and I’m glad the damage wasn’t worse. Is there a way to auto-renew these? I’m going to have to worry about it again in like 11 months.

### Is it all worth it?

I’ve asked myself this question now several times while dealing with these random issues. Now, I wouldn’t trade pushing a Docker image full of my code and dependencies, and having that transparently replace my old one with no downtime, for anything. But, surely I can do that with something like AWS ECS/Fargate and some well-written CloudFormation templates. (That’s another story…) Or I can use a managed Kubernetes service.

But, there’s some joy in learning a new skill, and I can put this on a $10 node (maybe even $5 now, I’ve noticed that the newer Kubernetes seems a little less resource-intensive?). The alternatives are surprisingly pricey and I don’t want to pay $50-$100 or more a month to host my toy Scrabble word study app. If this were a company it would be a no-brainer to use a hosted solution, at least until we made enough to bring it back in-house and hire some DevOps engineers to manage the whole thing =)