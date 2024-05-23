# Telepresence


## 参考文档

* [Intercept a service in your own environment](https://www.getambassador.io/docs/telepresence/latest/howtos/intercepts)
* [Share development environments with personal intercepts](https://www.getambassador.io/docs/telepresence/latest/howtos/personal-intercepts)
* [Proxy outbound traffic to my cluster](https://www.getambassador.io/docs/telepresence/latest/howtos/outbound)
* 配置
  * [Laptop-side configuration](https://www.getambassador.io/docs/telepresence/latest/reference/config)
  * [Cluster-side configuration](https://www.getambassador.io/docs/telepresence/latest/reference/cluster-config)
* [Environment variables](https://www.getambassador.io/docs/telepresence/latest/reference/environment) - Telepresence can import environment variables from the cluster pod when running an intercept. 
* Intercepts
  * [Configuring intercept using CLI](https://www.getambassador.io/docs/telepresence/latest/reference/intercepts/cli)
  * [Configuring intercept using specifications](https://www.getambassador.io/docs/telepresence/latest/reference/intercepts/specs)
* [FAQs](https://www.getambassador.io/docs/telepresence/latest/faqs)
  * What operating systems does Telepresence work on?
  * What protocols can be intercepted by Telepresence?
  * When using Telepresence to intercept a pod, are the Kubernetes cluster environment variables proxied to my local machine?
  * When using Telepresence to intercept a pod, can the associated pod volume mounts also be mounted by my local machine?
  * When connected to a Kubernetes cluster via Telepresence, can I access cluster-based services via their DNS name?
  * When connected to a Kubernetes cluster via Telepresence, can I access cloud-based services and data stores via their DNS name?
  * What components get installed in the cluster when running Telepresence?
  * How can I remove all the Telepresence components installed within my cluster?
  * How does Telepresence connect and tunnel into the Kubernetes cluster?
* [Troubleshooting](https://www.getambassador.io/docs/telepresence/latest/troubleshooting)
  * Creating an intercept did not generate a preview URL
  * `too many files open error` when running `telepresence connect` on Linux
  * Connected to cluster via VPN but IPs don't resolve
