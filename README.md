<p align="center">
  <img src="https://github.com/prasenforu/serverless/blob/master/images/fission.png">
</p>

Fission is a new open-source, Function as a Service framework built on top of Kubernetes. This serverless framework focuses on developer productivity and high performance and takes care of the details at the container or Kubernetes level. Fission enables developers to write short-lived functions in any language, and map them to HTTP requests or event triggers. With this framework, you can simply create functions and instantly deploy them via CLI. Kubernetes offers a powerful and flexible orchestration system for Fission to upload and run code without having to worry about building containers and managing Docker registries. The framework is extensible to any programming language; it currently supports NodeJS and Python.

Fission is open source under the Apache License. Fission can run on any platform where Kubernetes can run, in Data Centre (physical or Virtual), Public & Private Cloud.

## Architecture 

<p align="center">
  <img src="https://github.com/prasenforu/serverless/blob/master/images/fission-arch.png">
</p>

#### Fission’s architecture consists of the following component

##### Controller 
Its a CRUD (Create, Read, Update and Delete) APIs for Kubernetes functions, HTTP triggers, environments and watchers. This is the component with which the component talks. keeps track of functions, HTTP routes, event triggers, and environment images

##### Pool Manager
Manages shared resources (generic and function containers). It has a simple API. Manages pools of idle environment containers, the loading of functions into these containers, and the killing of function instances when they're idle.

##### Routers
HTTP requests to function pods. If there is not a service running for a function, it requests one from Pool Manager, while the request remains open; when the service of the function is available, it re-routes the request. The router is stateless, and can be escalated if necessary, depending on the load. Basically receives HTTP requests and routes them to function instances, requesting an instance from the Pool Manager if necessary.

##### Kubewatcher
Responsible for monitoring the Kubernetes’ API and calling watcher-linked functions, forwarding the watch event to the function. The controller keeps track of the number of watchers requested by the user and their associated functions. Kubernetes monitors the API in accordance with said requests; when a watch event takes place, it serializes the object and calls the function through the router.

##### Environment container(s). 
They execute user-defined functions. Environment containers are specific of each language. They must contain an HTTP server and a loader for functions.

##### Logger 
Helps to redirect function logs towards a centralized data base service to ensure the persistence of these logs.

## How Fission works on Kubernetes

Fission is designed as a set of microservices. A ```Controller``` keeps track of functions, HTTP
routes, event triggers, and environment images. A ```Pool Manager``` manages pools of idle environment containers, the loading of functions into these containers, and the killing of function instances when they're idle. A ```Router``` receives HTTP requests and routes them to function instances, requesting an instance from the Pool Manager if necessary.

The controller serves the fission API. All the other components watch the controller for updates. The router is exposed as a Kubernetes Service of the LoadBalancer or NodePort type, depending on where the Kubernetes cluster is hosted.

When the router gets a request, it looks up a cache to see if this request already has a service it should be routed to. If not, it looks up the function to map the request to, and requests the poolmgr for an instance. The poolmgr has a pool of idle pods; it picks one, loads the function into it (by sending a request into a sidecar container in the pod), and returns the address of the pod to the router. The router  proxies over the request to this pod. The pod is cached for subsequent requests, and if it's been idle for a few minutes, it is killed.

#### Lets! break down function’s life cycle creation (by the developer) and its execution (by the user or client application). 

##### First Step. 
After coding the function, the developer uploads it to Fission via the REST API provided by the controller. To facilitate interaction with Fission, we have the fission-client client for Mac and Linux. The controller stores the function and related information in its database (etcd)

##### Second Step.
After loading the function, and also interacting with the controller, we create a new path that will allow us to call the function by means of a URL such as http://FISION_ROUTER/hello_world

Once the function is ready for use, its life cycle consists of the following steps:

##### Step #1.
When the function is called through HTTP, what we do is contact the router, which reroutes our requests to the right address. In our case, a request to http://FISION_ROUTER/hello_world will be handled by function hello_world.py.

##### Step #2.
If the function does not exist in execution, a new container would be instantiated with the adequate execution environment (python is our example). Then, the new container will retrieve the hello_world.py function by means of the fetcher.

##### Step #3.
After a few milliseconds, the container will be ready to handle the HTTP request that it will receive through the router.

##### Step #4.
Once the request is satisfied, the container containing the function will remain alive during a specific period of time. If, during this time no request is received, it will be destroyed. These functions in stand-by are dubbed hot functions.

Delving a bit deeper into Fission’s deployment architecture in Kubernetes, it is important to point out that all components, except environment containers, are deployed in a namespace called “fission”. The environment containers created and instantiated for each language and/or function created are deployed in a different namespace dubbed “fission-function”.

## Installation

After Installation of Kubernetes cluster, make sure cluster running.

#### Step-1 Verify access to the cluster:

```kubectl version```

#### Step-2 Build Fission setup

```kubectl create -f fission.yml```

Set up services & Ingress controller with ClusterIP/NodePort. Setup nodeport on 31313 and 31314 ports.

```kubectl create -f fission-route.yml```

Set the FISSION_URL and FISSION_ROUTER environment variable. The fission command line interface uses the FISSION_URL to find the server. The URL has to be prefixed with http://.

```
export FISSION_URL=http://fission-url.cloudapps.cloud-cafe.in
export FISSION_ROUTER=fission-router.cloudapps.cloud-cafe.in
```

#### Step-3 Installing fission CLI (Command Line Interface)

```curl http://fission.io/linux/fission > fission && chmod +x fission && sudo mv fission /usr/local/bin/```

#### Step-4 Installing Fission UI (User Interface)

```kubectl create -f fission-ui.yml```

#### Step-5 Check Fission UI in browser

```http://fission-ui.cloudapps.cloud-cafe.in```

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
curl http://fission-router.cloudapps.cloud-cafe.in/hello_world
Hello, World!
```
