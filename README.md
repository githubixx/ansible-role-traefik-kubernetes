# traefik-kubernetes

This Ansible role installs [Traefik](https://traefik.io/) edge router for Kubernetes as ingress controller. Behind the doors it uses the official [Helm chart](https://helm.traefik.io/traefik). Currently procedures like installing, updating/upgrading and deleting the Traefik deployment are supported.

The provided default settings are optimized for a bare-metal/on-premise Kubernetes cluster where Traefik is the public entry point for the Kubernetes services. While the configuration can of course be customized as you can do with any Helm chart, the default settings will setup Traefik with the following configuration:

- Traefik instances will be deployed as `DaemonSet`
- Traefik pods uses `hostPort`
- Traefik listens on port `80` on all interfaces of the host for incoming HTTP requests
- Traefik listens on port `443` on all interfaces of the host for incoming HTTPS request
- Traefik dashboard is enabled and is not exposed to the public internet
- TLS certificates are provided by [cert-manager](https://cert-manager.io) (My Ansible role [cert-manager-kubernetes](https://github.com/githubixx/ansible-role-cert-manager-kubernetes) can be used to install cert-manager)

For further information about the Helm chart settings see below.

## Versions

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `8.0.0+33.2.1` means this is release `8.0.0` of this role and it uses Helm chart version `33.2.1` (the `Traefik` version used is specified in the values file [see below]). If the role itself changes `X.Y.Z` before `+` will increase. If the Traefik chart version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Traefik release.

## Requirements

You need to have [Helm 3](https://helm.sh/) binary installed on that host where `ansible-playbook` is executed or on that host where you delegated the playbooks to (e.g. by using `traefik_delegate_to` variable). You can either

- use your favorite package manager if your distribution includes `helm` in its repository (for Archlinux use `sudo pacman -S helm` e.g.)
- or use one of the Ansible `Helm` roles (e.g. [helm](https://galaxy.ansible.com/gantsign/helm) - which gets also installed if you use `ansible-galaxy role install -vr requirements.yml`
- or directly download the binary from [Helm releases)[https://github.com/helm/helm/releases]) and put it into `/usr/local/bin/` directory e.g.

A properly configured `KUBECONFIG` is also needed (which is located at `${HOME}/.kube/config` by default). Normally if `kubectl` works with your cluster then you everything should be already fine in this regards.

Additionally the Ansible `kubernetes.core` collection needs to be installed. This can be done by using the `collections.yml` file included in this role: `ansible-galaxy install -r collections.yml`.

And of course you need a Kubernetes Cluster ;-)

## Changelog

**Change history:**

See full [CHANGELOG.md](https://github.com/githubixx/ansible-role-traefik-kubernetes/blob/master/CHANGELOG.md)

**Recent changes:**

## 8.0.0+33.2.1

### Important notes for Traefik Helm chart v33.2.1

There were quite some changes in regards to Kubernetes Gateway API since the last upgrade. If you haven't used Gateway API resources so far, upgrading shouldn't be that much of an issue. But if you do, please read the changelogs before upgrading!

- [Traefik Helm chart v32.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v32.0.0)
- [Traefik Helm chart v33.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v33.0.0)
- [gateway-api v1.2.0](https://github.com/kubernetes-sigs/gateway-api/releases/tag/v1.2.0): In case you're using Kubernetes [Gateway API](https://github.com/kubernetes-sigs/gateway-api/) resources, CRDs will no longer serve the `v1alpha2` versions of `GRPCRoute` and `ReferenceGrant`. So make sure you upgrade that resources to `v1` before the upgrade.
- Kubernetes Gateway Provider Experimental Channel: Because of a breaking change introduced in Kubernetes Gateway API [v1.2.0-rc1](https://github.com/kubernetes-sigs/gateway-api/releases/tag/v1.2.0-rc1), Traefik v3.2 only supports Kubernetes Gateway v1.2.x when experimental channel features are enabled (set variable `traefik_gateway_api_crds` to `experimental`).
- See [Gateway API v1.2 upgrade nodes](https://gateway-api.sigs.k8s.io/guides/#v12-upgrade-notes)
- See [Gateway API v1.1 upgrade nodes](https://gateway-api.sigs.k8s.io/guides/#v11-upgrade-notes)

If you're using the default values file `templates/traefik_values_default.yml.j2` of this role then the `traefik` endpoint stays on port `9000`. The changelog of the Helm chart `v33.0.0` mentions that the default has changed to port `8080`. So if you have your own values file this is something do check!

Also the following (potential) breaking changes are mentioned in the [changelog of Traefik Helm chart v33.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v33.0.0):

- `publishedService` is enabled by default on Ingress provider
- The `POD_NAME` and `POD_NAMESPACE` environment variables are now set by default, without values.
- In values, `certResolvers` specific syntax has been [reworked](https://github.com/traefik/traefik-helm-chart/pull/1214) to align with Traefik Proxy syntax.
- Traefik Proxy 3.2 supports Gateway API v1.2 (standard channel)

Breaking changes mentioned in [changelog of Traefik Helm chart v32.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v32.0.0):

- There is a breaking change on how Redis is configured

### Important changes regarding gateway-api in 8.0.0+33.2.1

As mentioned above in Gateway API v1.2.0 some breaking changes were introduced which should make future upgrades easier. But at least currently these changes are introducing more headache IMHO esp. if one needs to upgrade existing Gateway API resources.

To reflect these changes, setting the variable `traefik_install_crds` to `true` no longer installs `*.gateway.networking.k8s.io` CRDs! To install these CRDs set Ansible variable `traefik_gateway_api_crds` to `standard` or `experimental` (see `defaults/main.yaml` for more information regarding Gateway API CRD settings). If you already using Gateway API resources since earlier versions it might make sense to use the `experimental` channel for now (see [Gateway API v1.1 upgrade nodes](https://gateway-api.sigs.k8s.io/guides/#v11-upgrade-notes) for more information on switching from `experimental` to `standard` channel later).

### Other changes in 8.0.0+33.2.1

- update Traefik from version `3.1.5` to `3.2.3`
- update Custom Resource Definitions (CRDs)

## Role Variables

```yaml
# Helm chart version
traefik_chart_version: "33.2.1"

# Helm release name
traefik_release_name: "traefik"

# Helm repository name
traefik_repo_name: "traefik"

# Helm chart name
traefik_chart_name: "{{ traefik_repo_name }}/{{ traefik_release_name }}"

# Helm chart URL
traefik_chart_url: "https://helm.traefik.io/traefik"

# Kubernetes namespace where Traefik resources should be installed
traefik_namespace: "traefik"

# IMPORTANT: The value "v2" is only meant to be used when you
# migrate from Traefik v2 to v3! This snippet in the static configuration
# makes the v2 format the default rule matchers syntax. If you do a fresh
# install of Traefik set this value to "v3". For more information see:
#
# https://doc.traefik.io/traefik/migration/v2-to-v3/
# https://doc.traefik.io/traefik/migration/v2-to-v3-details/#kubernetes-ingress-path-matching
#
# Also this variable is only relevant if you use the default
# "templates/traefik_values_default.yml.j2" file. If you use your
# own values file you can set this value there. It sets "core.defaultRuleSyntax"
# accordingly.
traefik_default_path_matcher_syntax: "v3"

# Directory that contains Helm chart values file. If you specify this
# variable Ansible will try to locate a file called "values.yml.j2" or
# "values.yaml.j2" in the specified directory (".j2" because you can
# use the usual Jinja2 template stuff there). The content of this file
# will be provided to "helm install/template" command as values file.
# By default the directory is the users "$HOME" directory plus
# "/traefik/helm". If the task doesn't find such a file it uses
# the values in "templates/traefik_values_default.yml.j2" by default.
traefik_chart_values_directory: "{{ '~/traefik/helm' | expanduser }}"

# By default CRDs (CustomResourceDefinitions) are not installed. Set to
# "true" if CRDs should be installed. Also see:
# https://github.com/traefik/traefik-helm-chart/tree/master/traefik/crds
# The following CRDs will be installed:
#
#   - accesscontrolpolicies.hub.traefik.io
#   - apiaccesses.hub.traefik.io
#   - apiportals.hub.traefik.io
#   - apiratelimits.hub.traefik.io
#   - apis.hub.traefik.io
#   - apiversions.hub.traefik.io
#   - ingressroutes.traefik.io
#   - ingressroutetcps.traefik.io
#   - ingressrouteudps.traefik.io
#   - middlewares.traefik.io
#   - middlewaretcps.traefik.io
#   - serverstransports.traefik.io
#   - serverstransporttcps.traefik.io
#   - tlsoptions.traefik.io
#   - tlsstores.traefik.io
#   - traefikservices.traefik.io
#
traefik_install_crds: false

# Specifies which Gateway API CRDs (CustomResourceDefinitions) should
# be installed. This setting is about installing "*.gateway.networking.k8s.io"
# CRDs.
#
# By default this setting is "none" which means no Gateway API CRDs will
# be installed.
#
# The Kubernetes Gateway API has two release channels (see:
# https://gateway-api.sigs.k8s.io/concepts/versioning/#release-channels):
#
#   - "standard"
#   - "experimental"
#
# "standard" will install the following CRDs:
#
#   - gatewayclasses.gateway.networking.k8s.io
#   - gateways.gateway.networking.k8s.io
#   - grpcroutes.gateway.networking.k8s.io
#   - httproutes.gateway.networking.k8s.io
#   - referencegrants.gateway.networking.k8s.io
#
# "experimental" will install the ones listed in "standard" plus a few
# more. Please be aware that the "experimental" channel is subject to
# change and might not be stable. That means that the CRDs might change
# or be removed in future releases. So the CRDs listed above in "standard"
# might also contain fields that are subject to change if the "experimental"
# channel is used. The "experimental" channel installs the following
# additional CRDs:
#
#   - backendlbpolicies.gateway.networking.k8s.io
#   - backendtlspolicies.gateway.networking.k8s.io
#   - gatewayclasses.gateway.networking.k8s.io
#   - gateways.gateway.networking.k8s.io
#   - grpcroutes.gateway.networking.k8s.io
#   - httproutes.gateway.networking.k8s.io
#   - referencegrants.gateway.networking.k8s.io
#   - tcproutes.gateway.networking.k8s.io
#   - tlsroutes.gateway.networking.k8s.io
#   - udproutes.gateway.networking.k8s.io
#
traefik_gateway_api_crds: "none"

# By default all tasks that needs to communicate with the Kubernetes
# cluster are executed on your local host (127.0.0.1). But if that one
# doesn't have direct connection to this cluster or should be executed
# elsewhere this variable can be changed accordingly.
traefik_delegate_to: 127.0.0.1

# Shows the "helm" command that was executed if a task uses Helm to
# install, update/upgrade or deletes such a resource.
traefik_helm_show_commands: false

# Without "action" variable defined this role will only render a file
# with all the resources that will be installed or upgraded. The rendered
# file with the resources will be called "template.yml" and will be
# placed in the directory specified below.
traefik_template_output_directory: "{{ '~/traefik/template' | expanduser }}"
```

## Usage

The first thing to do is to check `templates/traefik_values_default.yml.j2`. This file contains the important values/settings for the Traefik Helm chart that are partly different to the default ones which are located [here](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml). These are the default settings used:

```yaml
# All possible Helm chart values here can be found at:
# https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml

# IMPORTANT: The value "v2" is only meant to be used when you
# migrate from Traefik v2 to v3! This snippet in the static configuration
# makes the v2 format the default rule matchers syntax. If you do a fresh
# install of Traefik this setting isn't needed. For more information see:
# https://doc.traefik.io/traefik/migration/v2-to-v3/
{% if traefik_default_path_matcher_syntax is defined and traefik_default_path_matcher_syntax == "v2" %}
core:
  defaultRuleSyntax: v2
{% endif %}

image:
  registry: docker.io
  repository: traefik
  tag: "3.2.3"
  pullPolicy: IfNotPresent

# These arguments are passed to Traefik's binary. For all options see:
# https://doc.traefik.io/traefik/reference/static-configuration/cli/
#
# First one sets log level accordingly.
#
# Second one sets value of "kubernetes.io/ingress.class"
# annotation to watch for. If a "standard" Kubernetes "Ingress" object
# is submitted to Kubernetes API server  (instead of Traefik's own ingress
# implementation called "IngressRoute"), Traefik will handle these requests
# and route them accordingly.
additionalArguments:
  - "--log.level=INFO"
  - "--providers.kubernetesingress.ingressclass=traefik"

# Global arguments passed to Traefik's binary.
# https://doc.traefik.io/traefik/reference/static-configuration/cli/
#
# First one disables periodical check if a new version has been released.
#
# Second one disables anonymous usage statistics. 
globalArguments:
  - "--global.checknewversion=false"
  - "--global.sendanonymoususage=false"

# This creates the Traefik deployment. As "DaemonSet" is specified here
# this will create a Traefik instance on all Kubernetes worker nodes. If
# only a subset of nodes should be used specify "affinity", "nodeSelector"
# or "toleration's" accordingly. See link to Helm chart values above.
deployment:
  enabled: true
  kind: DaemonSet
  dnsPolicy: ClusterFirstWithHostNet

# Instructs Traefik to listen on various ports.
ports:
  # The name of this one can't be changed as it is used for the readiness
  # and liveness probes, but you can adjust its config to your liking.
  # To access the dashboard you can use "kubectl port-forward" e.g.:
  # kubectl -n traefik port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name -A | head -1) 9000:9000
  # Opening http://localhost:9000/dashboard/ should show the dashboard.
  #
  # NOTE: You SHOULD NOT expose the "traefik" port on production deployments.
  # If you want to access it from outside your cluster,
  # use `kubectl port-forward` or create a secure ingress.
  traefik:
    port: 9000
    protocol: TCP
    expose:
      default: false
  # Unsecured incoming HTTP traffic. If you uncomment "redirectTo: websecure"
  # all traffic that arrives at this port will be redirected to "websecure"
  # entry point which means to the entry point that handles secure HTTPS traffic.
  # But be aware that this could be problematic for cert-manager.
  # Also "hostPort" is used. As "DaemonSet" was specified above that basically
  # means that Traefik pods will answer requests on port 80 and 443 on all
  # Kubernetes worker nodes. So if the hosts have a public IP and port 80/443
  # are not protected by firewall, Traefik is available for requests from the
  # Internet (what you normally want in case of Traefik ;-) ) For other
  # options see link above. These settings are useful for baremetal or
  # on-premise solutions with no further loadbalancer.
  web:
    port: 30080
    hostPort: 80
    expose:
      default: true
    protocol: TCP
    # redirectTo: websecure
  # Entry point for HTTPS traffic.
  websecure:
    port: 30443
    hostPort: 443
    expose:
      default: true
    protocol: TCP
    tls:
      enabled: true

# When providers.kubernetesGateway.enabled, deploy a default gateway
gateway:
  enabled: false
  listeners:
    web:
      # Port is the network port. Multiple listeners may use the same port,
      # subject to the Listener compatibility rules.
      # The port must match a port declared in ports section.
      port: 30080
      # Specify expected protocol on this listener.
      # See [ProtocolType](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.ProtocolType)
      protocol: HTTP
    #websecure:
      # Port is the network port. Multiple listeners may use the same port,
      # subject to the Listener compatibility rules.
      # The port must match a port declared in ports section
      #port: 30443
      # Optional hostname.
      # See [Hostname](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.Hostname)
      #hostname:
      # Specify expected protocol on this listener.
      # See [ProtocolType](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.ProtocolType)
      #protocol: HTTPS
      # Routes are restricted to namespace of the gateway
      # [by default](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.FromNamespaces)
      #namespacePolicy
      # Add certificates for TLS or HTTPS protocols. This value must be specified
      # if "gateway.enabled: true" and "gateway.listeners.websecure" is defined!
      # See [GatewayTLSConfig](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io%2fv1.GatewayTLSConfig)
      #certificateRefs:
      # TLS behavior for the TLS session initiated by the client.
      # See [TLSModeType](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.TLSModeType).
      #mode:

# These security settings are basically best practice to limit attack surface
# as good as possible.
# The attack surface can further limited with "seccomp" which is stable since
# Kubernetes v1.19 and allows to limit system calls to a bare minimum. See:
# https://kubernetes.io/docs/tutorials/clusters/seccomp/"
securityContext:
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true
  runAsGroup: 65532
  runAsNonRoot: true
  runAsUser: 65532

# All processes of the container are also part of this supplementary group ID.
podSecurityContext:
  fsGroup: 65532

# Set log level of general log and enable access log.
logs:
  general:
    level: INFO
  access:
    enabled: true

# As Traefik web/websecure ports are exposed by "hostPort" a service isn't
# needed.
service:
  enabled: false

# CPU and RAM resource limits. These settings should also be set to
# reasonable values in case of a memory leak e.g.
resources:
  requests:
    cpu: "100m"
    memory: "50Mi"
  limits:
    cpu: "300m"
    memory: "150Mi"

# Updates the Pods in the DaemonSet one by one when the DaemonSet was upgraded
updateStrategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 0

providers:
  kubernetesCRD:
    # Load Kubernetes "IngressRoute" provider
    enabled: true
  kubernetesIngress:
    # Load Kubernetes "Ingress" provider
    enabled: true
  kubernetesGateway:
    # Enable Traefik "Gateway" provider for Gateway API
    enabled: false
    # Toggles support for the Experimental Channel resources (Gateway API
    # release channels documentation). This option currently enables
    # support for "TCPRoute" and "TLSRoute".
    # If this setting is set to "true" please also set the Ansible variable
    # "traefik_gateway_api_crds" to "experimental"!
    experimentalChannel: false

# Create a default IngressClass for Traefik
ingressClass:
  enabled: true
  isDefaultClass: true
```

But nothing is made in stone. To use your own values just create a file called `values.yml.j2` or `values.yaml.j2` and put it into the directory specified in `traefik_chart_values_directory` (which is `$HOME/traefik/helm` by default). Then this role will use that file to render the Helm values. You can use `templates/traefik_values_default.yml.j2` as a template or just start from scratch. As mentioned above you can modify all settings for the Helm chart that are different to the default ones which are located [here](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml).

After the values file is in place and the `defaults/main.yml` values are checked the role can be installed. Most of the role's tasks are executed locally so to say as quite a few tasks need to communicate with the Kubernetes API server or executing [Helm](https://helm.sh/) commands.

The default action is to just render the Kubernetes resources YAML file after replacing all Jinja2 variables and stuff like that. In the `Example Playbook` section below there is an `Example 2 (assign tag to role)`. The role `githubixx.traefik-kubernetes` has a tag `role-traefik-kubernetes` assigned. Assuming that the values for the Helm chart should be rendered (nothing will be installed in this case) and the playbook is called `k8s.yml` execute the following command:

```bash
ansible-playbook --tags=role-traefik-kubernetes k8s.yml
```

To render the template into a different directory use `traefik_template_output_directory` variable e.g.:

```bash
ansible-playbook --tags=role-traefik-kubernetes --extra-vars traefik_template_output_directory="/tmp/traefik" k8s.yml
```

If you want to see the `helm` commands and the parameters which were executed in the logs you can also specify `--extra-vars traefik_helm_show_commands=true`.

One of the final tasks is called `TASK [githubixx.traefik_kubernetes : Write templates to file]`. This renders the template with the resources that will be created into the directory specified in `traefik_template_output_directory`. The file will be called `template.yml`. The directory/file will be placed on your local machine to be able to inspect it.

If the rendered template contains everything you need the role can be installed which finally deploys Traefik:

```bash
ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=install k8s.yml
```

To check if everything was deployed use the usual `kubectl` commands like `kubectl -n <traefik_namespace> get pods -o wide`.

As Traefik issues updates/upgrades every few weeks/months the role also can do updates/upgrades. This method can also be used to change existing values without upgrading the Traefik version e.g. Also see [Traefik releases](https://github.com/traefik/traefik/releases) before updating Traefik. Changes to the Helm chart can be found in the [commit history](https://github.com/traefik/traefik-helm-chart/commits/master).

If you want to upgrade Traefik/Helm chart you basically only need to change `traefik_chart_version` variable e.g. from `30.1.0` to `31.0.0`. You might also need to change `image.tag` value in `templates/traefik_values_default.yml.j2`. If only parameters should be changed update the values accordingly.

So to do the Traefik update or to roll out the new values run

```bash
ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=upgrade k8s.yml
```

And finally if you want to get rid of Traefik you can delete all resources again:

```bash
ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=delete k8s.yml
```

## Example Playbook

Example 1 (without role tag):

```yaml
- hosts: traefik
  roles:
    - githubixx.traefik_kubernetes
```

Example 2 (assign tag to role):

```yaml
-
  hosts: traefik
  roles:
    -
      role: githubixx.traefik-kubernetes
      tags: role-traefik-kubernetes
```

The host `traefik` in the example playbook is most probably just `localhost` specified in Ansible's `hosts` file or whatever host you want to use as "runner" so to say. Just make sure that this host has `helm` installed and has a valid `kubeconfig` (which is normally the case if `kubectl` command works with the Kubernetes cluster).

## Testing

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-traefik-kubernetes/tree/master/molecule/default).

Afterwards molecule can be executed. Molecule will setup a few VMs with a complete Kubernetes cluster which makes it possible to test the all the Traefik functionality.

The following command will do a basic setup and create a template of the resources (default action see above) that will be created:

```bash
molecule converge
```

Installing `Traefik` and the required resources:

```bash
molecule converge -- --extra-vars action=install
```

Upgrading `Traefik` or changing parameters:

```bash
molecule converge -- --extra-vars action=upgrade
```

Deleting `Traefik` and its resources:

```bash
molecule converge -- --extra-vars action=delete
```

To run a few tests use

```bash
molecule verify
```

To clean up run

```bash
molecule destroy
```

## License

GNU GENERAL PUBLIC LICENSE Version 3

## Author Information

[http://www.tauceti.blog](http://www.tauceti.blog)
