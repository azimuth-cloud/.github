# Azimuth Cloud

Azimuth is an open source cloud interface to help
scientists transform AI and HPC infrastructure into
something useful.

Azimuth enables scientists to:

* Create a Machine in seconds, a Cluster in minutes
* One click SSO into Desktop and Applications
* Securely share Applications with external users
* Monitoring of Performance, Utilization and Cloud Capacity

For cloud operators, Azimuth helps you:

* Curate application catalog, with site specific optimizations
* Resource limits, with optional max application runtime

Interested?
[Give Azimuth a try](https://azimuth-config.readthedocs.io/en/stable/try/) on your OpenStack cloud!

Not got an OpenStack cloud? We are working on a standalone
version that runs on a kubernetes cluster.

## Azimuth Office hours

To be confirmed. We hope to establish a monthly community meeting.

## Roadmap

As of September 2025, our current draft roadmap includes:

* Restoring our monthly release cadance, to ensure timely updates to all the supplied images
* Adding the option to separate OpenStack and Azimuth authentication and authorization
* Enabling more hybrid cloud use cases, by supporting non-OpenStack k8s clusters
* Adding CI testing for the non-OpenStack and OIDC features
* Enable the use of Azimuth project stroage outside of Azimuth, such as inside a shared Slurm cluster
* Improved OpenStack resource management, leaning on tools such as
  [OpenStack Blazar](https://docs.openstack.org/blazar/latest/) and [Waldur](https://waldur.com/)

## Architecture

Azimuth consists of a [Python](https://www.python.org/) backend providing a REST API (different
to the OpenStack API) and a Javascript frontend written in [React](https://reactjs.org/).

At it's core, Azimuth is just an OpenStack client. When a user authenticates with Azimuth, it
negotiates with [Keystone](https://docs.openstack.org/keystone/latest/) (using either
username/password or federation) to obtain a token which is stored in a cookie in the user's
browser. Azimuth then uses this token to talk to the OpenStack API on behalf of the user when
the user submits requests to the Azimuth API via the Azimuth UI.

When the Zenith application proxy and Cluster-as-a-Service (CaaS) subsystems are enabled,
it allows you securely expose multiple isolated web applications
from a single ingress controller.

All of Azimuth's state is stored in Kubernetes. There are operators that expose CRDs
that allow Azimuth's API to declartively describe the users requests for platforms.

## Azimuth History

Azimuth was originally developed for the [JASMIN Cloud](https://jasmin.ac.uk/) as a simplified
version of the [OpenStack Horizon](https://docs.openstack.org/horizon/latest/) dashboard, with the
aim of reducing complexity for less technical users.

It has since grown to offer additional functionality with a continued focus on simplicity for
scientific use cases, including the ability to provision complex platforms via a user-friendly
interface. The platforms range from single-machine workstations with web-based remote console
and desktop access to entire [Slurm](https://slurm.schedmd.com/) clusters and platforms such
as [JupyterHub](https://jupyter.org/hub) that run on [Kubernetes](https://kubernetes.io/) clusters.

Services are exposed to users without consuming floating IPs or requiring SSH keys using the
[Zenith](https://github.com/azimuth-cloud/zenith) application proxy.

Here, you can see Stig Telfer (CTO) from [StackHPC](https://www.stackhpc.com/)
presenting Azimuth at the
[OpenInfra Summit in Berlin in 2022](https://openinfra.dev/summit/berlin-2022):

[![Azimuth - self service cloud platforms for reducing time to science](https://img.youtube.com/vi/FRbpI7ZsvMw/0.jpg)](https://www.youtube.com/watch?v=FRbpI7ZsvMw)

Key features of Azimuth include:

  * Supports multiple Keystone authentication methods simultaneously:
    * Username and password, e.g. for LDAP integration.
    * [Keystone federation](https://docs.openstack.org/keystone/latest/admin/federation/introduction.html)
      for integration with existing [OpenID Connect](https://openid.net/connect/) or 
      [SAML 2.0](http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html)
      identity providers.
    * [Application credentials](https://docs.openstack.org/keystone/latest/user/application_credentials.html)
      to allowing the distribution of easily-revokable credentials, e.g. for training, or for integrating
      with clouds that use federation but implementing the required trust is not possible.
  * On-demand Platforms
    * Unified interface for managing Kubernetes and CaaS platforms.
    * Kubernetes-as-a-Service and Kubernetes-based platforms
      * Operators provide a curated set of templates defining available Kubernetes versions,
        networking configurations, custom addons etc.
      * Uses [Cluster API](https://cluster-api.sigs.k8s.io/) to provision Kubernetes clusters.
      * Supports Kubernetes version upgrades with minimal downtime using rolling node replacement.
      * Supports auto-healing clusters that automatically identify and replace unhealthy nodes.
      * Supports multiple node groups, including auto-scaling node groups.
      * Supports clusters that can utilise GPUs and accelerated networking (e.g. SR-IOV).
      * Installs and configures addons for monitoring + logging, system dashboards and ingress.
      * Kubernetes-based platforms, such as JupyterHub, as first-class citizens in the platform catalog.
    * Cluster-as-a-Service (CaaS)
      * Operators provide a curated catalog of appliances.
      * Appliances are Ansible playbooks that provision and configure infrastructure.
        * Ansible calls out to [OpenTofu](https://opentofu.org/) to provision infrastructure.
      * [CaaS Operator](https://github.com/azimuth-cloud/azimuth-caas-operator) runs the ansible
        for the appliances such as the [workstation](https://github.com/azimuth-cloud/caas-workstation) and
        [Slurm](https://github.com/stackhpc/ansible-slurm-appliance/tree/main/environments/.caas).
  * Application proxy using [Zenith](https://github.com/azimuth-cloud/zenith):
    * Zenith uses SSH tunnels to expose services running behind NAT or a firewall to the internet
      using operator-controlled, random domains.
      * Exposed services do not need to be directly accessible to the internet.
      * Exposed services do not consume a floating IP.
    * Zenith supports an auth callout for proxied services, which Azimuth uses to secure proxied services.
    * Used by Azimuth to provide access to platforms, e.g.:
      * Web-based console / desktop access using [Apache Guacamole](https://guacamole.apache.org/)
      * Monitoring and system dashboards
      * Platform-specific interfaces such as [Jupyter Notebooks](https://jupyter.org/) and
        [Open OnDemand](https://openondemand.org/)
  * (Deprecated) Simplified interface for managing basic OpenStack resources:
    * Automatic detection of networking, with auto-provisioning of networks and routers if required.
    * Create, update and delete machines with automatic network detection.
    * Create, delete and attach volumes.
    * Allocate, attach and detach floating IPs.
    * Configure instance-specific security group rules.

## Timeline

This section shows a timeline of the significant events in the development of Azimuth:

  * **Autumn 2015**: Development begins on the JASMIN Cloud Portal, targetting JASMIN's VMware cloud.
  * **Spring 2016**: JASMIN Cloud Portal goes into production.
  * **Early 2017**: JASMIN Cloud plans to move to OpenStack, cloud portal v2 development begins.
  * **Summer 2017**: JASMIN's OpenStack cloud goes into production, with the JASMIN Cloud Portal v2.
  * **Spring 2019**: Work begins on JASMIN Cluster-as-a-Service (CaaS) with [StackHPC](https://www.stackhpc.com/).
    * Initial work presented at [UKRI Cloud Workshop](https://cloud.ac.uk/workshops/feb2019/).
  * **Summer 2019**: JASMIN CaaS beta roll out.
  * **Spring 2020**: JASMIN CaaS in use by customers, e.g. the
    [ESA Climate Change Initiative Knowledge Exchange](https://climate.esa.int/en/) project.
    * Production system presented at [UKRI Cloud Workshop](https://cloud.ac.uk/workshops/mar2020/).
  * **Summer 2020**: Production rollout of JASMIN CaaS.
  * **Spring 2021**: StackHPC fork JASMIN Cloud Portal to develop it for [IRIS](https://www.iris.ac.uk/).
  * **Summer 2021**: [Zenith application proxy](https://github.com/azimuth-cloud/zenith) developed and used
    to provide web consoles in Azimuth.
  * **November 2021**: StackHPC fork detached and rebranded to Azimuth.
  * **December 2021**: StackHPC Slurm appliance integrated into CaaS.
  * **January 2022**: Native Kubernetes support added using Cluster API (previously supported by JASMIN as a CaaS appliance).
  * **February 2022**: Support for exposing services in Kubernetes using Zenith.
  * **March 2022**: Support for exposing services in CaaS appliances using Zenith.
  * **June 2022**: Unified platforms interface for Kubernetes and CaaS.
  * **October 2022**: Support for Kubernetes platforms in the unified platforms interface.
  * **January 2023**: Created [new cluster as a service](https://github.com/azimuth-cloud/azimuth-caas-operator) operator, to replace use of AWX.
  * **March 2024**: New [schedule operator](https://github.com/azimuth-cloud/azimuth-schedule-operator) that co-ordinates reservations with OpenStack Blazar
  * **April 2024**: Removed depency on Consul, with Zenith now using CRDs to store state
  * **May 2024**: Explored [standalone FluxCD based](https://github.com/stackhpc/capi-helm-fluxcd-config)
    [capi-helm-chart](https://github.com/azimuth-cloud/capi-helm-charts) K8s clusters
  * **May 2024**: First upstream release of [OpenStack Magnum driver](https://opendev.org/openstack/magnum-capi-helm/)
    that uses [capi-helm-charts](https://github.com/azimuth-cloud/capi-helm-charts)
  * **February 2025**: Creation of new [Apps operator](https://github.com/azimuth-cloud/azimuth-apps-operator) using FluxCD to deploy Kubernetes based Apps
  * **Setempber 2025**: Support merged for OIDC authentication and non-OpenStack Azimuth deployents.
