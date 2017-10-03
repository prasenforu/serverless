<p align="center">
  <img src="https://github.com/prasenforu/serverless/blob/master/images/fission.png">
</p>

Fission is a new open-source, Function as a Service framework built on top of Kubernetes. This serverless framework focuses on developer productivity and high performance and takes care of the details at the container or Kubernetes level. Fission enables developers to write short-lived functions in any language, and map them to HTTP requests or event triggers. With this framework, you can simply create functions and instantly deploy them via CLI. Kubernetes offers a powerful and flexible orchestration system for Fission to upload and run code without having to worry about building containers and managing Docker registries. The framework is extensible to any programming language; it currently supports NodeJS and Python.

Fission is open source under the Apache License. Fission can run on any platform where Kubernetes can run, in Data Centre, Public & Private Cloud.

## Architecture 

<p align="center">
  <img src="https://github.com/prasenforu/serverless/blob/master/images/fission-arch.png">
</p>

## How Fission works on Kubernetes

Fission is designed as a set of microservices. A Controller keeps track of functions, HTTP
routes, event triggers, and environment images. A Pool Manager manages pools of idle environment containers, the loading of functions into these containers, and the killing of function instances when they're idle. A Router receives HTTP requests and routes them to function instances, requesting an instance from the Pool Manager if necessary.

The controller serves the fission API. All the other components watch the controller for updates. The router is exposed as a Kubernetes Service of the LoadBalancer or NodePort type, depending on where the Kubernetes cluster is hosted.

When the router gets a request, it looks up a cache to see if this request already has a service it should be routed to. If not, it looks up the function to map the request to, and requests the poolmgr for an instance. The poolmgr has a pool of idle pods; it picks one, loads the function into it (by sending a request into a sidecar container in the pod), and returns the address of the pod to the router. The router  proxies over the request to this pod. The pod is cached for subsequent requests, and if it's been idle for a few minutes, it is killed.

## Installation

After Installation of Kubernetes cluster, make sure cluster running.

#### Step-1 Verify access to the cluster:

```kubectl version```

#### Step-2 Build Fission setup

```kubectl create -f fission.yml```

Set up services with NodePort in case you are deploying it on Minikube or on  cloud service provider. These files run on 31313 and 31314 ports.

```kubectl create -f fission-nodeport.yml```

Set the FISSION_URL and FISSION_ROUTER environment variable. The fission command line interface uses the FISSION_URL to find the server. The URL has to be prefixed with http://.

```
export FISSION_URL=http://$(server ip):31313
export FISSION_ROUTER=$(server ip):31314
```

#### Step-3 Installing fission CLI (Command Line Interface)

```curl http://fission.io/linux/fission > fission && chmod +x fission && sudo mv fission/usr/local/bin/```

#### Step-4 Installing Fission UI (User Interface)

```kubectl create -f fission-ui.yml```

## Sample Deployment code using fission

###### Create new python file

```
cat hello_world.py
def main():
    return “Hello, World!\n”
```    

###### Check if the environment is already created for  python application

```
fission env get --name python

Failed to get environment: (Error 0) HTTP error 500
```

###### Create new environment for python application

```
fission env create --name python --image fission/python-env

environment 'python' created
```

###### Check if image env is created:

```fission env get --name python```

###### Create new function that uses the image

```
fission function create --name hello_world --env python --code hello_world.py --url /hello_world --method GET

function 'hello_world' created
route created: GET /hello_world -> hello_world
```

###### Test if the function runs

```
curl http://$FISSION_ROUTER/hello_world
Hello, World!
```
