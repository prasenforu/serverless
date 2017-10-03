# serverless
Fission a Function as a Service (Serverless) framework on top of Kubernetes

## Fission

Fission is a new open-source, Function as a Service framework built on top of Kubernetes. This serverless framework focuses on developer productivity and high performance and takes care of the details at the container or Kubernetes level. Fission enables developers to write short-lived functions in any language, and map them to HTTP requests or event triggers. With this framework, you can simply create functions and instantly deploy them via CLI. Kubernetes offers a powerful and flexible orchestration system for Fission to upload and run code without having to worry about building containers and managing Docker registries. The framework is extensible to any programming language; it currently supports NodeJS and Python.

Fission is open source under the Apache License. Fission can run on any platform where Kubernetes can run, in Data Centre, Public & Private Cloud.

## Architecture 


## How Fission works on Kubernetes

Fission is designed as a set of microservices. A Controller keeps track of functions, HTTP
routes, event triggers, and environment images. A Pool Manager manages pools of idle environment containers, the loading of functions into these containers, and the killing of function instances when they're idle. A Router receives HTTP requests and routes them to function instances, requesting an instance from the Pool Manager if necessary.

The controller serves the fission API. All the other components watch the controller for updates. The router is exposed as a Kubernetes Service of the LoadBalancer or NodePort type, depending on where the Kubernetes cluster is hosted.

When the router gets a request, it looks up a cache to see if this request already has a service it should be routed to. If not, it looks up the function to map the request to, and requests the poolmgr for an instance. The poolmgr has a pool of idle pods; it picks one, loads the function into it (by sending a request into a sidecar container in the pod), and returns the address of the pod to the router. The router  proxies over the request to this pod. The pod is cached for subsequent requests, and if it's been idle for a few minutes, it is killed.

(For now, Fission maps one function to one container; autoscaling to multiple instances is planned for a later release. Re-using function pods to host multiple functions is also planned, for cases where isolation isn't a requirement.)

## Installation

