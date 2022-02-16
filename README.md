traefik-kubernetes
==================

This Ansible role installs [Traefik](https://traefik.io/) edge router for Kubernetes as ingress controller. Behind the doors it uses the official [Helm chart](https://helm.traefik.io/traefik). Currently procedures like installing, updating/upgrading and deleting the Traefik deployment are supported.

The provided default settings are optimized for a bare-metal/on-premise Kubernetes cluster where Traefik is the public entry point for the Kubernetes services. While the configuration can of course be customized as you can do with any Helm chart, the default settings will setup Traefik with the following configuration:

- Traefik instances will be deployed as `DaemonSet`
- Traefik pods uses `hostPort`
- Traefik listens on port `80` on all interfaces of the host for incoming HTTP requests
- Traefik listens on port `443` on all interfaces of the host for incoming HTTPS request
- Traefik dashboard is enabled and is not exposed to the public internet
- TLS certificates are provided by [cert-manager](https://cert-manager.io) (My Ansible role [cert-manager-kubernetes](https://github.com/githubixx/ansible-role-cert-manager-kubernetes) can be used to install cert-manager)

For further information about the Helm chart settings see below.

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `2.3.0+10.14.1` means this is release `2.3.0` of this role and it uses Helm chart version `10.14.1` (the `Traefik` version used is specified in the values file [see below]). If the role itself changes `X.Y.Z` before `+` will increase. If the Traefik chart version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Traefik release.

Requirements
------------

You need to have [Helm 3](https://helm.sh/) binary installed on that host where `ansible-playbook` runs. You can either try to use your favorite package manager if your distribution includes `helm` in its repository or use one of the Ansible `Helm` roles (e.g. [helm](https://galaxy.ansible.com/gantsign/helm) or directly download the binary from [Helm releases)[https://github.com/helm/helm/releases]) and put it into `/usr/local/bin/` directory e.g.) For Archlinux Helm can be installed via `sudo pacman -S helm` e.g.

Also `kubectl` should to be installed. At least you need a proper configured `KUBECONFIG` (which is located at `${HOME}/.kube/config` by default).

And of course you need a Kubernetes Cluster ;-)

Changelog
---------

see [CHANGELOG.md](https://github.com/githubixx/ansible-role-traefik-kubernetes/blob/master/CHANGELOG.md)

Role Variables
--------------

```yaml
# Helm chart version
traefik_chart_version: "10.14.1"

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

# Directory that contains Helm chart values file. If you specify this
# variable Ansible will try to locate a file called "values.yml.j2" or
# "values.yaml.j2" in the specified directory (".j2" because you can
# use the usual Jinja2 template stuff there). The content of this file
# will be provided to "helm install/template" command as values file.
# By default the directory is the users "$HOME" directory plus
# "/traefik/helm". If the task doesn't find such a file it uses
# the values in "templates/traefik_values_default.yml.j2" by default.
traefik_chart_values_directory: "{{ '~/traefik/helm' | expanduser }}"
```

Usage
-----

The first thing to do is to check `templates/traefik_values_default.yml.j2`. This file contains the values/settings for the Traefik Helm chart that are different to the default ones which are located [here](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml). These are the default settings used:

```yaml
# All possible Helm chart values here can be found at:
# https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml

image:
  name: traefik
  tag: "2.6.1"
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
  traefik:
    port: 9000
    expose: false
    protocol: TCP
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
    expose: true
    protocol: TCP
    # redirectTo: websecure
  # Entry point for HTTPS traffic.
  websecure:
    port: 30443
    hostPort: 443
    expose: true
    protocol: TCP

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
```

But nothing is made in stone ;-) To use your own values just create a file called `values.yml.j2` or `values.yaml.j2` and put it into the directory specified in `traefik_chart_values_directory` (which is `$HOME/traefik/helm` by default). Then this role will use that file to render the Helm values. You can use `templates/traefik_values_default.yml.j2` as a template or just start from scratch. As mentioned above you can modify all settings for the Helm chart that are different to the default ones which are located [here](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml).

After the values file is in place and the `defaults/main.yml` values are checked the role can be installed. Most of the role's tasks are executed locally so to say as quite a few tasks need to communicate with the Kubernetes API server or executing [Helm](https://helm.sh/) commands.

The default action is to just render the Kubernetes resources YAML file after replacing all Jinja2 variables and stuff like that. In the `Example Playbook` section below there is an `Example 2 (assign tag to role)`. The role `githubixx.traefik-kubernetes` has a tag `role-traefik-kubernetes` assigned. Assuming that the values for the Helm chart should be rendered (nothing will be installed in this case) and the playbook is called `k8s.yml` execute the following command:

```bash
ansible-playbook --tags=role-traefik-kubernetes k8s.yml
```

One of the final tasks is called `TASK [githubixx.traefik-kubernetes : Output rendered template]`. This allows to check the YAML file before Traefik gets deployed. As you might figure out the output isn't that "pretty". To get around this you can either set `ANSIBLE_STDOUT_CALLBACK=debug` environment variable or `stdout_callback = debug` in `ansible.cfg`. If you run the `ansible-playbook` command again now the YAML output should now look way better.

If the rendered output contains everything you need the role can be installed which finally deploys Traefik:

```bash
ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=install k8s.yml
```

To check if everything was deployed use the usual `kubectl` commands like `kubectl -n <traefik_namespace> get pods -o wide`.

As Traefik issues updates/upgrades every few weeks/months the role also can do updates/upgrades. This method can also be used to change existing values without upgrading the Traefik version e.g. Also see [Traefik releases](https://github.com/traefik/traefik/releases) before updating Traefik. Changes to the Helm chart can be found in the [commit history](https://github.com/traefik/traefik-helm-chart/commits/master).

If you want to upgrade Traefik/Helm chart you basically only need to change `traefik_chart_version` variable e.g. from `9.12.3` to `9.13.0`. If only parameters should be changed update the values accordingly.

So to do the Traefik update or to roll out the new values run

```bash
ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=upgrade k8s.yml
```

And finally if you want to get rid of Traefik you can delete all resources again:

```bash
ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=delete k8s.yml
```

Example Playbook
----------------

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

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
