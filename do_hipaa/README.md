# Digital Ocean HIPAA Compliant Architecture

This is just a draft based on an initial research. The ideas and architecture diagrams presented here is just a POC (Proof Of Concept).

## Overview

Before going into more details, there are some questions that need to be asked first:

1. What makes a system `Highly Available` or `HA`? 
2. What makes a system `Secure` or less susceptible to malicious attacks (data steal/loss, ranswomware, etc) ?
3. Can `DigitalOcean` be a good candidate?

Next, some true facts need to be stated:

1. `NO` system can be made `100%` HA because nothing is perfect (`DigitalOcean` guarantees `99% uptime` for all provided `Cloud Services`)
2. `NO` system can be `100%` secure due to many factors (software patches, using in 3rd party software or even unknown software sources sometimes, etc). What we can do is to try to make it less susceptible to attacks or harder to break. This is part of the `hardening` process of every system. Often this needs a `dedicated` person or team like `DevSecOps` and such.
3. As with every `Cloud Provider` available out there, some have advantages over the other in terms of the available `features`, `ease` of use, `support` and `maintenance`. And, let's not to forget about `costs` as well. `DigitalOcean` has attractive prices, a simpler `Cloud Services` architecture and interface and lots and lots of resources available delivered as `tutorials`, `how to's`, etc. On the other hand it is more `Developer` oriented/friendly, and getting something to work is really fast and as painful as possible. This doesn't mean that it cannot scale up to meet higher demands for more `Enterprise` oriented applications and architectures. In the end, it all comes down to a tradeoff between `ease` of use and `low costs` vs `Enterprise` ready solutions (like `AWS`), which due to higher features available and `complexity` it can imply `more costs` and even `vendor lock-in` (`hard` to `migrate` the existing infrastructure to another `Cloud Provider` in the future).

Based on the above, let's try to discuss the `pros` and `cons` using some simple diagrams given as POC's.


## HIPAA Architecture 1st Draft

In order for a system or architecture to be `HIPAA` compliant, there are many `"Terms and Conditions"` that it must obey to or comply in terms of security, like:

1. How `secure` is the underlying `infrastructure` starting with all the machines being used and all the `hardware` the application runs on ? 
2. Is `user` data `encrypted` all over the place (starting from the `log in` process until it gets stored in a `database` somewhere) ? 
3. Is any `sensitive` part of the infrastructure `exposed` to the `outside` world ?
4. Is the system `audited` periodically? 
5. How does it handle `software patching` for security? Does it run `vulnerability scans` periodically?
6. Is the `database` less susceptible to `SQL injection` attacks? Is the data stored `encrypted` and decryption keys safe?
7. Do we use `extra safety` for user logins, like `MFA` (Multi Factor Authentication) ?

The above list can go on and on. Let's have a first draft and then observe its strong and weak points:

![DO HIPAA 1st Draft Arch](res/img/do_hipaa_arch_1st.jpg)

Nothing special here, just a plain old and classic architecture.

**Pros:**

1. Highly available `multi-region` setup via `Global Server Load Balancing` (GLSB) (`GLSB` is used as a `failover mechanism` between `Datacenters` from different `Regions`, as well as routing traffic to the closest region for the user). [NGINX Plus](https://www.nginx.com/resources/glossary/global-server-load-balancing/) or [Akamai GTM](https://www.akamai.com/products/global-traffic-management) can be used.
2. `Isolation` between `regions` makes the system easy to adapt in compliance to each region `regulatory` stuff 
3. Each `Public Load Balancer` has `SSL` passthrough and termination enabled
4. `Firewalls` enabled and set up for each `Droplet` (DigitalOcean VM's that run Linux OS)
5. `VPC isolation` between the `Application` Tier and `Database` Tier
6. `Database encryption` and `SQL attacks` protection via `ACRA` Server (available as a `DigitalOcean App`)
7. `Secure Encryption` between each layer via `SSL/TLS`
8. `MFA` for Users

**Cons:**

1. `DigitalOcean` doesn't provide an `Auto Scaler` for their VM's (called `Droplets`). It's available if the `App Platform` is used, but it's kind of limited in functionality. For more details please visit the [App Platform Overview](https://docs.digitalocean.com/products/app-platform) and [App Platform Limitations](https://docs.digitalocean.com/products/app-platform/#limits).
2. Because of the above, the architecture presented in the first draft is kind of `rigid` in terms of `resource usage` and `costs`. It means that no matter if the `load` on the system is `high` or `low`, a `fixed number` of VM's still `run in the background` and maybe doing almost `nothing`.
3. `Identity and Access Management` (IAM) is another feature that's not currently supported on `DigitalOcean` to allow for `fine` control of `who` has access on `what`.
4. Each region application service is isolated, so communication between services from different regions can add more complexity
   
In terms of security and for the sake of simplicity a `VPN setup` was not added, but `OpenVPN` can be added and it's available as a `DigitalOcean App`. `DigitalOcean App` platform let's you easily install and configure `one-click apps` from a `trusted` marketplace. 

`Auditing` tools or `CVE` scanners for the underlying `Linux OS` distributions which `Droplets` use or other software components, were ommited for simplicity as well (this doesn't mean that options and support is not available).

### References:

* Droplet as a [Gateway](https://docs.digitalocean.com/products/networking/vpc/resources/droplet-as-gateway/) for DigitalOcean.
* [VPC Networks](https://docs.digitalocean.com/products/networking/vpc/resources/best-practices/) best practices for DigitalOcean.
* [DNS Round Robin technique](https://www.digitalocean.com/community/tutorials/how-to-configure-dns-round-robin-load-balancing-for-high-availability) for DigitalOcean.
* [ACRA](https://marketplace.digitalocean.com/apps/acra) for `SQL` injection attacks protection and database `encryption` - available as a `DigitalOcean Application Droplet`.
* DigitalOcean [Managed Databases](https://docs.digitalocean.com/products/databases/) support and documentation.
* DigitalOcean [DNS](https://docs.digitalocean.com/products/networking/dns/) support and documentation.
* DigitalOcean [Load Balancers](https://docs.digitalocean.com/products/networking/load-balancers/) support and documentation.
* DigitalOcean [Cloud Firewalls](https://docs.digitalocean.com/products/networking/firewalls/) support and documentation.


## HIPAA Architecture 2nd Draft

A more robust and efficient setup to use is one based on `Kubernetes`. This overcomes all of the above first draft cons, and even more.

![DO HIPAA 2nd Draft Arch](res/img/do_hipaa_arch_2nd.jpg)

`Kubernetes` is widely known for its `High Availability` features, `reliability`, `security` and `ease` of configuration. Not to mention the big `support` available from the `community` and ready to use `tools`.

Most notable features:

* `Role Based Access Control` (RBAC) for fine control of `who` has access on `what`.
* `Rolling Deployments` strategy for `zero` application `downtime` during upgrades.
* `Easy` and `automatic` application `scale up/down` based on the load and needs.
* `Efficient` and `secure` application `packaging` and `distribution` via the `Docker` Container Runtime Interface (CRI) support.
* Setting up `Docker` registries with `vulnerability` scan support to prevent security flaws from early stages of development.
* `Multi-region` setup support available from `Kubernetes`.
* `Communication` between `services` across `diferrent regions` can be accomplished more easy via `Service Mesh` (e.g. [LINKERD](https://linkerd.io)).

What `DigitalOcean` offers in terms of `Kubernetes` support:

* Easy `Kubernetes` cluster deployment and management via [DOKS](https://www.digitalocean.com/products/kubernetes) so you don't need to worry about the `Control Plane` or any other internal stuff for having a `Kubernetes` cluster up and running. All is managed by `DigitalOcean`, you only need to focus on application development and deployment, not the internals of Kubernetes.
* `Multi-region` support was added recently so you can have `High Availabilty` for `DOKS` as well, spanning across `multiple datacenters`
* Each `worker node` comes with `Cloud Firewall` support and it's enabled by default
* `Block Storage` support for `Kubernetes Persistent Volumes`
* `Load Balancer` support

### Overview

The second architecture draft adds some extra flavor like:

* `Ingress` and `API Gateway` support via the  `Ambassador Edge Stack`
* `Monitoring` via `Prometheus`
* `Logs` aggregation and indexing via `Loki`
* `Backup/Restore` via `Velero` (`DOKS` cluster only, `Database Tier` has its own `Backup/Restore` support provided via `DigitalOcean` services)
* `DigitalOcean Spaces` (`S3` like) remote storage for `Loki` data and `Velero` backups
* Although not present in the diagram, `DigitalOcean` offers secure `Docker Container Registry` support for storing application images. It can be integrated with 3rd party tools for periodic image `vulnerability scans`.

This approach is not fully `Kubernetes` based. It's rather a `hybrid` approach. `Application` tier is fully managed via `Kubernetes`, but the `Database Tier` is not. Although, you can deploy databases into `Kubernetes` as well via `StatefulSets` and use `Persistent Volumes` for permanent storage (`DigitalOcean` offers `Block Storage` support for `Volumes`), it adds some overhead in terms of maintenance. `DigitalOcean Managed Database` service simplifies this process so, in the end it's a matter of choice and evaluation of costs between having a `self managed database` inside `Kubernetes` or using the `DigitalOcean` services for this matter.

`Monitoring` and `Logging` can be `centralized` so that it's not replicated across each region and in each cluster. This approach reduces `complexity` and `costs` in general.

**DigitalOcean Kubernetes Security:**

* `DigitalOcean` keeps Kubernetes `secure by default` (`SSL` encryption is enabled) and `patching` new releases as well
* In general, `Kubernetes` clusters can be secured even more via `RBAC`, disabling `unauthenticated` access to the `API server`, setting up and configuring `Admission Controllers`, disabling `Public Access` to the cluster, etc.
* Using an `Ingress` controller like `Ambassador Edge Stack` offers extra layers of security as well which can be configured accordingly

### References:

* DigitalOcean [Managed Kubernetes](https://docs.digitalocean.com/products/kubernetes/) and [security](https://docs.digitalocean.com/products/kubernetes/resources/security/) support and documentation.
* Securing [Ambassador Edge Stack](https://www.getambassador.io/products/edge-stack/api-gateway/security-authentication/) based applications.
* [Kubecost](https://www.kubecost.com) for evaluating `Kubernetes` clusters `costs` and adjusting `limits/requests`.
