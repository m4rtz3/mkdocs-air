## Kubernetes

Kubernetes, also known as K8s, is an open-source platform designed to automate deploying, scaling, and operating application containers. It was originally designed by Google and is now maintained by the Cloud Native Computing Foundation.

Key features of Kubernetes include:

- **Service discovery and load balancing**: Kubernetes can expose a container using the DNS name or their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic to help the deployment stable.
- **Storage orchestration**: Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.
- **Automated rollouts and rollbacks**: You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers and adopt all their resources to the new container.
- **Automatic bin packing**: You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.
- **Self-healing**: Kubernetes restarts containers that fail, replaces and reschedules containers when nodes die, kills containers that don’t respond to your user-defined health check, and doesn’t advertise them to clients until they are ready to serve.
- **Secret and configuration management**: Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration.


[MiniKube](https://minikube.sigs.k8s.io/docs/start/)


https://cloud.google.com/learn/what-is-kubernetes?hl=pt-br#section-4

https://serverlessland.com/
