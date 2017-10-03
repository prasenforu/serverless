### Build Serverless API Backends

![Alt text](https://github.com/prasenforu/serverless/blob/master/images/apibe.png)

Create API backends for web and mobile apps without managing servers. Just write functions, and map them to HTTP routes. Fission takes care of the rest: deployment, routing, scalability, availability. Use Kubernetes' service discovery and networking to interoperate with other services, like Redis, Postgres, Etcd etc.

### Easily implement Webhooks

![Alt text](https://github.com/prasenforu/serverless/blob/master/images/webhook.png)

Webhooks are a popular way to integrate with third-party services. Slack provides webhooks that are triggered by certain words or messages; Github provides webhooks triggered by events in Git repositories. Fission is a great place to implement webhooks: just write the code, map it to a URL, and point the webhook at that URL.

### Write Kubernetes Event Handlers

![Alt text](https://github.com/prasenforu/serverless/blob/master/images/event.png)

By subscribing to Kubernetes watches, you can write custom automation for your Kubernetes infrastructure. Fission's integration with Kubernetes watches allows you to monitor resources such as Pods and Services, and execute arbitrary functions when the watched set of resources change.
