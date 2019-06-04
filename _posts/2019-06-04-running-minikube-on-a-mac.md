---
published: true
title: Running Minikube on a Mac
layout: post
---

In the next few weeks I am going to work a lot with Kubernetes, which is why I 
am posting the installation and running guide for [Minikube] on a Mac. This will
be a prerequisite for going on with those posts, and other posts will link back
to this one for guidance. 

So for installing Minikube, it's the simplest thing using [brew]:

```bash
brew install virtualbox minikube
```

This will install both [Virtualbox] for running the VM and [Minikube] which
configures the VM for running, and provisions Kubernetes onto it. 

After this you just need to start the cluster:

```bash
minikube start
```

And the output you see should be similar to this one below:

```bash
Starting local Kubernetes cluster...
Running pre-create checks...
Creating machine...
Starting local Kubernetes cluster...
```

## If you have Docker.app installed

The credit for this solution goes to [this StackOverflow answer].

If you run into something similar to the error below: 

```
error: SchemaError(io.k8s.api.autoscaling.v2beta2.MetricTarget): invalid object
doesn't have additional properties
```

Then your Docker.app may be interfering with your configuration. To resolve it,
first check which `kubectl` app is running:

```bash
ls -l $(which kubectl) 
```

If it returns something similar to: 

```bash
/usr/local/bin/kubectl -> /Applications/Docker.app/Contents/Resources/bin/kubectl
```

Then you should overwrite it: 

```bash
rm /usr/local/bin/kubectl
brew link --overwrite kubernetes-cli
```

And this should solve the error.


[brew]: https://brew.sh
[Virtualbox]: https://virtualbox.org
[Minikube]: https://github.com/kubernetes/minikube
[this StackOverflow answer]: https://stackoverflow.com/a/55737973