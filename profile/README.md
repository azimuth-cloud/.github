# Azimuth Cloud

Azimuth is a self-service portal for managing cloud resources, aimed at high-performance computing (HPC) and artificial intelligence (AI) workloads.

It works with [OpenStack](https://www.openstack.org/) clouds.
Users can create workstations with web console and desktop access, [Slurm](https://slurm.schedmd.com/) clusters, and apps running on [Kubernetes](https://kubernetes.io/).

## Get started

* **Try Azimuth:** [deploy a demo on OpenStack](https://azimuth-config.readthedocs.io/en/stable/try/) or read the [user guide](https://azimuth-cloud.github.io/azimuth-user-docs/).
* **Contribute code or documentation:** choose a component from the [repository map](#repositories) and follow its development instructions.
  For the portal, start with [CONTRIBUTING.md](https://github.com/azimuth-cloud/azimuth/blob/master/CONTRIBUTING.md) and the [local development guide](https://github.com/azimuth-cloud/azimuth/blob/master/docs/local-development.md).
  The portal's API tests and UI builds can be run locally without a cloud account.
  To work against a running deployment, follow the [OpenStack and Tilt development workflow](https://azimuth-config.readthedocs.io/en/stable/developing/).
* **Find a component:** use the [repository map](#repositories) below.
* **Operate Azimuth:** follow the [deployment and operations documentation](https://azimuth-config.readthedocs.io/en/stable/).

The main deployment setup uses OpenStack.
There is also a [standalone mode](https://azimuth-config.readthedocs.io/en/stable/configuration/17-standalone-mode/) for running apps on an existing Kubernetes cluster with OpenID Connect authentication.
This is still experimental (alpha).
CaaS and Cluster API are not available in standalone mode.

## Repositories

Azimuth is made up of several components.
The [portal repository](https://github.com/azimuth-cloud/azimuth) contains the API, UI and their Helm chart.

| Component | Source | What it does |
| --- | --- | --- |
| API, UI and portal chart | [azimuth](https://github.com/azimuth-cloud/azimuth): [api](https://github.com/azimuth-cloud/azimuth/tree/master/api), [ui](https://github.com/azimuth-cloud/azimuth/tree/master/ui), [chart](https://github.com/azimuth-cloud/azimuth/tree/master/chart) | Django REST API, React UI and Helm chart. |
| Kubernetes clusters | [azimuth-capi-operator](https://github.com/azimuth-cloud/azimuth-capi-operator) | Manages Kubernetes clusters through Cluster API. |
| Kubernetes cluster charts | [capi-helm-charts](https://github.com/azimuth-cloud/capi-helm-charts) | Helm charts for Kubernetes clusters and addons. |
| Cluster API addons | [cluster-api-addon-provider](https://github.com/azimuth-cloud/cluster-api-addon-provider) | Manages addons for Cluster API clusters. |
| Cluster-as-a-Service | [azimuth-caas-operator](https://github.com/azimuth-cloud/azimuth-caas-operator) | Runs Ansible appliances to provision and configure platforms. |
| Kubernetes apps | [azimuth-apps-operator](https://github.com/azimuth-cloud/azimuth-apps-operator) | Manages apps using App and AppTemplate resources, including in standalone mode. |
| Platform access | [zenith](https://github.com/azimuth-cloud/zenith) | Exposes platform services through authenticated tunnels. |
| Platform identity | [azimuth-identity-operator](https://github.com/azimuth-cloud/azimuth-identity-operator) | Manages platform identity integration. |
| Platform scheduling | [azimuth-schedule-operator](https://github.com/azimuth-cloud/azimuth-schedule-operator) | Manages platform leases and expiry. |
| Deployment and development setup | [azimuth-config](https://github.com/azimuth-cloud/azimuth-config) | Deployment configuration, docs and Tilt setup. |
| Deployment playbooks | [ansible-collection-azimuth-ops](https://github.com/azimuth-cloud/ansible-collection-azimuth-ops) | Ansible roles and playbooks for deploying Azimuth and its dependencies. |
| User documentation | [azimuth-user-docs](https://github.com/azimuth-cloud/azimuth-user-docs) | Guides for Azimuth users. |

## Contents <!-- omit in toc -->

- [Features](#features)
- [Try Azimuth](#try-azimuth)
- [Architecture](#architecture)
- [Deploying Azimuth](#deploying-azimuth)
- [Azimuth History](#azimuth-history)
- [Timeline](#timeline)

## Features

  * Supports multiple Keystone authentication methods simultaneously:
    * Username and password, e.g. for LDAP integration.
    * [Keystone federation](https://docs.openstack.org/keystone/latest/admin/federation/introduction.html) for integration with existing [OpenID Connect](https://openid.net/connect/) or [SAML 2.0](http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html) identity providers.
    * [Application credentials](https://docs.openstack.org/keystone/latest/user/application_credentials.html) for distributing easily revocable credentials, e.g. for training, or for integrating with federated clouds where the required trust cannot be established.
  * Monitoring of performance, utilization and cloud capacity.
  * On-demand Platforms
    * Site-specific application catalogs and resource limits, with optional maximum application runtime.
    * Unified interface for managing Kubernetes and CaaS platforms.
    * Kubernetes-as-a-Service and Kubernetes-based platforms
      * Operators provide a curated set of templates defining available Kubernetes versions, networking configurations, custom addons etc.
      * Uses [Cluster API](https://cluster-api.sigs.k8s.io/) to provision Kubernetes clusters.
      * Supports Kubernetes version upgrades with minimal downtime using rolling node replacement.
      * Supports auto-healing clusters that automatically identify and replace unhealthy nodes.
      * Supports multiple node groups, including auto-scaling node groups.
      * Supports clusters that can utilise GPUs and accelerated networking (e.g. SR-IOV).
      * Installs and configures addons for monitoring + logging, system dashboards and ingress.
      * Kubernetes-based platforms as first-class citizens in the platform catalog.
    * Cluster-as-a-Service (CaaS)
      * Operators provide a curated catalog of appliances.
      * The [CaaS operator](https://github.com/azimuth-cloud/azimuth-caas-operator) runs Ansible appliances using [Ansible Runner](https://ansible.readthedocs.io/projects/runner/).
      * Appliances use [OpenTofu](https://opentofu.org/) to provision infrastructure and Ansible to configure it.
        Setup is covered in the [CaaS documentation](https://azimuth-config.readthedocs.io/en/stable/configuration/12-caas/) (the older AWX implementation is legacy).
  * Application proxy using Zenith:
    * Single sign-on access to desktops and applications.
    * Securely share applications with external users.
    * Zenith uses SSH tunnels to expose services running behind NAT or a firewall to the internet using operator-controlled, random domains.
      * Exposed services do not need to be directly accessible to the internet.
      * Exposed services do not consume a floating IP.
    * Zenith supports an auth callout for proxied services, which Azimuth uses to secure proxied services.
    * Used by Azimuth to provide access to platforms, e.g.:
      * Web-based console / desktop access using [Apache Guacamole](https://guacamole.apache.org/)
      * Monitoring and system dashboards
      * Platform-specific interfaces such as [Jupyter Notebooks](https://jupyter.org/) and [Open OnDemand](https://openondemand.org/)
  * (Deprecated) Simplified interface for managing basic OpenStack resources:
    * Automatic detection of networking, with auto-provisioning of networks and routers if required.
    * Create, update and delete machines with automatic network detection.
    * Create, delete and attach volumes.
    * Allocate, attach and detach floating IPs.
    * Configure instance-specific security group rules.

## Try Azimuth

To try Azimuth, follow the [demo deployment guide](https://azimuth-config.readthedocs.io/en/stable/try/).
You'll need OpenStack credentials, enough project capacity and a machine that can run the deployment tools.
The demo is for short-lived use.
For use in production, see the [production checklist](https://azimuth-config.readthedocs.io/en/stable/production/).

## Architecture

The React UI calls the Django REST API in the [portal repository](https://github.com/azimuth-cloud/azimuth).
In an OpenStack deployment, the API manages cloud resources on behalf of the logged-in user.
The main platform components are:

* The [CAPI operator](https://github.com/azimuth-cloud/azimuth-capi-operator) manages Kubernetes clusters through Cluster API and its infrastructure providers.
* The [CaaS operator](https://github.com/azimuth-cloud/azimuth-caas-operator) reconciles appliance resources and runs Ansible jobs.
* Kubernetes apps use either the [Cluster API addon provider](https://github.com/azimuth-cloud/cluster-api-addon-provider)'s HelmRelease resources or the [Apps operator](https://github.com/azimuth-cloud/azimuth-apps-operator)'s App resources, depending on the configured [apps provider](https://github.com/azimuth-cloud/azimuth/blob/master/chart/values.yaml).
  With Cluster API enabled, the default is HelmRelease.
* Zenith exposes platform services through tunnels, with authentication integrated into Azimuth.
  Service registrations are stored as Kubernetes resources.

See the [component repositories](#repositories) and the configuration guides for [Kubernetes clusters](https://azimuth-config.readthedocs.io/en/stable/configuration/10-kubernetes-clusters/), [Kubernetes apps](https://azimuth-config.readthedocs.io/en/stable/configuration/11-kubernetes-apps/) and [CaaS](https://azimuth-config.readthedocs.io/en/stable/configuration/12-caas/) for details.

## Deploying Azimuth

Azimuth runs on Kubernetes, with components installed using [Helm](https://helm.sh/).
The deployment setup uses configuration from [azimuth-config](https://github.com/azimuth-cloud/azimuth-config) and Ansible playbooks from [azimuth-ops](https://github.com/azimuth-cloud/ansible-collection-azimuth-ops) to install Azimuth and its dependencies, including operators and Zenith.
The components installed depend on which features you enable.

Follow the [deployment documentation](https://azimuth-config.readthedocs.io/en/stable/) for cloud prerequisites, Kubernetes, storage, ingress and configuration.
The [portal chart](https://github.com/azimuth-cloud/azimuth/tree/master/chart) is one part of that deployment.

## Azimuth History

Azimuth started as a simpler alternative to the [OpenStack Horizon](https://docs.openstack.org/horizon/latest/) dashboard for the [JASMIN Cloud](https://jasmin.ac.uk/).
It now also provides a platform catalog for deploying workstations, Slurm clusters and Kubernetes apps, with a focus on scientific computing.

The [Zenith](https://github.com/azimuth-cloud/zenith) application proxy exposes services to users without consuming floating IPs or requiring SSH keys.

Stig Telfer and Matt Pryor from [StackHPC](https://www.stackhpc.com/) gave this introduction to Azimuth at the [OpenInfra Summit in Berlin in 2022](https://openinfra.dev/summit/berlin-2022):

[![Azimuth - self service cloud platforms for reducing time to science](https://img.youtube.com/vi/FRbpI7ZsvMw/0.jpg)](https://www.youtube.com/watch?v=FRbpI7ZsvMw)

## Timeline

Major milestones in the development of Azimuth and its components:

  * **Autumn 2015**: Development begins on the JASMIN Cloud Portal, targeting JASMIN's VMware cloud.
  * **Spring 2016**: JASMIN Cloud Portal goes into production.
  * **Early 2017**: JASMIN Cloud plans to move to OpenStack, and development begins on Cloud Portal v2.
  * **Summer 2017**: JASMIN's OpenStack cloud goes into production, with the JASMIN Cloud Portal v2.
  * **Spring 2019**: Work begins on JASMIN Cluster-as-a-Service (CaaS) with [StackHPC](https://www.stackhpc.com/).
    * Initial work presented at [UKRI Cloud Workshop](https://cloud.ac.uk/workshops/feb2019/).
  * **Summer 2019**: JASMIN CaaS beta rollout.
  * **Spring 2020**: JASMIN CaaS in use by customers, e.g. the [ESA Climate Change Initiative Knowledge Exchange](https://climate.esa.int/en/) project.
    * Production system presented at [UKRI Cloud Workshop](https://cloud.ac.uk/workshops/mar2020/).
  * **Summer 2020**: Production rollout of JASMIN CaaS.
  * **Spring 2021**: StackHPC fork JASMIN Cloud Portal to develop it for [IRIS](https://www.iris.ac.uk/).
  * **Summer 2021**: [Zenith application proxy](https://github.com/azimuth-cloud/zenith) developed and used to provide web consoles in Azimuth.
  * **November 2021**: StackHPC fork detached and rebranded to Azimuth.
  * **December 2021**: StackHPC Slurm appliance integrated into CaaS.
  * **January 2022**: Native Kubernetes support added using Cluster API (previously supported by JASMIN as a CaaS appliance).
  * **February 2022**: Support for exposing services in Kubernetes using Zenith.
  * **March 2022**: Support for exposing services in CaaS appliances using Zenith.
  * **June 2022**: Unified platforms interface for Kubernetes and CaaS.
  * **October 2022**: Support for Kubernetes platforms in the unified platforms interface.
  * **January 2023**: Work begins on the [CaaS operator](https://github.com/azimuth-cloud/azimuth-caas-operator/pull/1) to replace AWX.
  * **March 2023**: [Identity operator and Keycloak integration](https://github.com/azimuth-cloud/ansible-collection-azimuth-ops/pull/44) added for platform access through Zenith.
  * **June 2023**: [CaaS deployments move from AWX to the CaaS operator](https://github.com/azimuth-cloud/ansible-collection-azimuth-ops/pull/85).
  * **March 2024**: [Schedule operator](https://github.com/azimuth-cloud/ansible-collection-azimuth-ops/pull/440) added to the deployment.
  * **April 2024**: [Zenith service registrations move from Consul to Kubernetes custom resources](https://github.com/azimuth-cloud/ansible-collection-azimuth-ops/pull/470).
  * **May 2024**: Exploration of standalone Kubernetes clusters using [FluxCD](https://github.com/stackhpc/capi-helm-fluxcd-config) and [capi-helm-charts](https://github.com/azimuth-cloud/capi-helm-charts).
  * **May 2024**: First upstream release of [OpenStack Magnum driver](https://opendev.org/openstack/magnum-capi-helm/) that uses [capi-helm-charts](https://github.com/azimuth-cloud/capi-helm-charts).
  * **July 2024**: [CaaS state moves to Kubernetes Secrets](https://github.com/azimuth-cloud/ansible-collection-azimuth-ops/pull/564), removing the remaining Consul dependency.
  * **August 2024**: Support for scheduled leases added to [CaaS](https://github.com/azimuth-cloud/azimuth-caas-operator/pull/159) and [Kubernetes](https://github.com/azimuth-cloud/azimuth-capi-operator/pull/252) platforms.
  * **February 2025**: [Apps operator](https://github.com/azimuth-cloud/azimuth-apps-operator/commit/5419f4bc29fb008f66d84b119401506e8e24c6e8) introduced to deploy Kubernetes apps using Flux.
  * **June 2025**: [OpenID Connect authentication and tenancy membership](https://github.com/azimuth-cloud/azimuth/pull/372) added to the portal, along with [Apps operator integration](https://github.com/azimuth-cloud/azimuth/pull/362).
  * **September 2025**: Experimental [standalone mode](https://github.com/azimuth-cloud/azimuth-config/pull/241) added for running Azimuth on an existing Kubernetes cluster without OpenStack.
  * **July 2026**: [AMD GPU Operator addon](https://github.com/azimuth-cloud/capi-helm-charts/pull/677) added for Kubernetes clusters.
  * **September 2026**: [Headlamp dashboard](https://github.com/azimuth-cloud/azimuth/pull/626) added for viewing and managing Kubernetes cluster resources.
  * **September 2026**: [Maximum platform lifetimes](https://github.com/azimuth-cloud/azimuth/pull/640) can be configured for individual CaaS appliances and Kubernetes cluster templates.
