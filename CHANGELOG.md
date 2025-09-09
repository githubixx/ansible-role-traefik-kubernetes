# Changelog

## 9.0.0+37.1.0

### Important notes for Traefik Helm chart v37.1.0

This update contains a rather big update of the Traefik Helm chart from v33.2.1 to v37.1.0. Please check [Traefik Proxy Helm Chart](https://github.com/traefik/traefik-helm-chart/releases) for potential breaking changes:

- [Traefik Helm chart v34.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v34.0.0)
- [Traefik Helm chart v35.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v35.0.0)
- [Traefik Helm chart v36.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v36.0.0)
- [Traefik Helm chart v37.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v37.0.0)


### Other changes in 9.0.0+37.1.0

- update Traefik from version `3.2.3` to `3.5.2`
- update Custom Resource Definitions (CRDs)

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

## 7.1.1+31.1.1

- update Traefik from version `3.1.4` to `3.1.5`

## 7.1.0+31.1.1

- update Helm chart to version `31.1.1`
- update Traefik from version `3.1.2` to `3.1.4`

## 7.0.0+30.1.0

### Important note

This is a major release upgrade of Traefik proxy `2.11.x` to `3.1.x` and the Traefik helm chart from `23.0.1` to `30.1.0`. This version of this role is ONLY compatible with Traefik `3.x.x`. If you need to stay with Traefik `2.11.x` please stay with version `6.0.1+23.2.0` of this role. Please read the following documentations before you upgrade to this role version:

- [Traefik Helm Chart Change Log](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/Changelog.md)
- [Traefik Proxy Change Log](https://github.com/traefik/traefik/blob/master/CHANGELOG.md)
- [Configuration Details for Migrating from Traefik v2 to v3](https://doc.traefik.io/traefik/migration/v2-to-v3-details/)
- [Migration Guide: From v2 to v3](https://doc.traefik.io/traefik/migration/v2-to-v3/)
- [Traefik v3 minor migrations](https://doc.traefik.io/traefik/migration/v3/)

### Further interesting readings

- [Traefik 3.0 GA Has Landed: Here's How to Migrate](https://traefik.io/blog/traefik-3-0-ga-has-landed-heres-how-to-migrate/)
- [Announcing Traefik Proxy v3.1](https://traefik.io/blog/announcing-traefik-proxy-v3-1/)
- [Getting started with Kubernetes Gateway API and Traefik](https://traefik.io/blog/getting-started-with-kubernetes-gateway-api-and-traefik/)
- [Monitor Your Production at a Glance With Traefik 3.0 and OpenTelemetry](https://traefik.io/blog/monitor-your-production-at-a-glance-with-traefik-3-0-and-opentelemetry/)

### Update CRDs

As mentioned in the changelog for version [5.0.0+23.0.1](#5002301) of this role already, Custom Resource Definition (CRD) API Group `traefik.containo.us` was deprecated and is now removed. So **before** you upgrade to this release, make sure your Traefik resources are changed accordingly. Please use the API Group `traefik.io` instead. E.g.:

- `ingressroutes.traefik.containo.us` -> `ingressroutes.traefik.io`
- `ingressroutetcps.traefik.containo.us` -> `ingressroutetcps.traefik.io`
- `ingressrouteudps.traefik.containo.us` -> `ingressrouteudps.traefik.io`
- `middlewares.traefik.containo.us` -> `middlewares.traefik.io`
- `middlewaretcps.traefik.containo.us` -> `middlewaretcps.traefik.io`
- `serverstransports.traefik.containo.us` -> `serverstransports.traefik.io`
- `tlsoptions.traefik.containo.us` -> `tlsoptions.traefik.io`
- `tlsstores.traefik.containo.us` -> `tlsstores.traefik.io`
- `traefikservices.traefik.containo.us` -> `traefikservices.traefik.io`

E.g.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
...
```

needs to be changed to

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
...
```

Do not delete the now obsolete `traefik.containo.us` CRDs yet! This should be done after Traefik 3.x was installed and the `IngressRoutes` and the other Traefik resources are still working. Here is an example to delete the obsolete CRDs after the upgrade:

```bash
kubectl delete crds \
  ingressroutes.traefik.containo.us \
  ingressroutetcps.traefik.containo.us \
  ingressrouteudps.traefik.containo.us \
  middlewares.traefik.containo.us \
  middlewaretcps.traefik.containo.us \
  serverstransports.traefik.containo.us \
  tlsoptions.traefik.containo.us \
  tlsstores.traefik.containo.us \
  traefikservices.traefik.containo.us
```

Check if any `traefik.containo.us` CRDs are left:

```bash
kubectl get crds  | grep traefik.containo.us
```

### Helm values

A few options were added to `templates/traefik_values_default.yml.j2`. E.g. added possibility to enable the Gateway API supported by Traefik v3 (disabled by default):

```yaml
providers:
  kubernetesGateway:
    # Enable Traefik "Gateway" provider for Gateway API
    enabled: true

# When providers.kubernetesGateway.enabled, deploy a default gateway
gateway:
  enabled: false
  ...
```

`updateStrategy.rollingUpdate.maxSurge` was added add value set to `0` by default (needed when `maxUnavailable` is set to a value different to `0`):

```yaml
updateStrategy:
  rollingUpdate:
    maxSurge: 0
```

If you migrate from Traefik v2 to v3 you definitely should consider setting the variable `traefik_default_path_matcher_syntax: v2`. This sets

```yaml
core:
  defaultRuleSyntax: v2
```

in `templates/traefik_values_default.yml.j2` to ensure backward compatibility with v2 path matcher syntax in the [dynamic configuration](https://doc.traefik.io/traefik/v3.0/migration/v2-to-v3-details/#dynamic-configuration-changes). Also see [Migration Guide: From v2 to v3](https://doc.traefik.io/traefik/migration/v2-to-v3/).

`ports.(traefik|web|websecure).expose` was changed to `ports.(traefik|web|websecure).expose.default`. This was a change in the Helm chart values file.

### Other changes

- update Helm chart to version `31.0.0`
- update Traefik from version `2.11.2` to `3.1.2`
- update `.yamllint` to meet `ansible-lint` [requirements](https://ansible.readthedocs.io/projects/lint/rules/yaml/#yamllint-configuration)

## 6.1.0+23.2.0

- update Traefik from version `2.11.2` to `2.11.8`
- refactor Molecule test
- `templates/traefik_values_default.yml.j2`: add `updateStrategy.rollingUpdate.maxSurge` with value `0`

## 6.0.1+23.2.0

- update Traefik from version `2.11.0` to `2.11.2`

## 6.0.0+23.2.0

- **POTENTIALLY BREAKING**: add `updateStrategy` to `templates/traefik_values_default.yml.j2`. This differs from the default [values.yaml](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml). Setting `maxUnavailable: 1` and removing `maxSurge: 1` to make rolling update work with the default `DaemonSet`
- update Helm chart to version `23.2.0`
- update Traefik from version `2.10.7` to `2.11.0`
- update Github workflow

## 5.1.3+23.1.0

- update Traefik from version `2.10.4` to `2.10.7`

## 5.1.2+23.1.0

- rename `githubixx.harden-linux` to `githubixx.harden_linux`

## 5.1.1+23.1.0

- rename `githubixx.kubernetes-ca` to `githubixx.kubernetes_ca`
- rename `githubixx.kubernetes-controller` to `githubixx.kubernetes_controller`
- rename `githubixx.kubernetes-worker` to `githubixx.kubernetes_worker`

## 5.1.0+23.1.0

- update Helm chart to version `23.1.0`
- update Traefik from version `2.10.1` to `2.10.4`
- `.yamllint`: ignore `files/crds/` directory
- fix ansible-lint issues

## 5.0.0+23.0.1

**Important note:** This release updates Traefik Helm chart from version `16.2.0` to `23.0.1`. Please read the following release notes carefully if you upgrade as it contains breaking changes esp. from chart version `16.x` to `17.x`:

- [Upgrading from 16.x to 17.x](https://github.com/traefik/traefik-helm-chart#upgrading-from-16x-to-17x)
- [Upgrading from 17.x to 18.x](https://github.com/traefik/traefik-helm-chart#upgrading-from-17x-to-18x)

TBH: If you can afford a small downtime I'd recommend to delete the old setup and roll out a new one. Or you do what is mentioned in the upgrade document: `You may also upgrade by deploying another Traefik to a different namespace and removing after your first Traefik.`. Personally I'd stay away from setting `instanceLabelOverride`. Sooner or later it will come back to you ;-)

Additionally also new [CRDs](https://github.com/traefik/traefik/pull/9765) were introduced with Traefik v2.10. That makes upgrading even more "fun". For more information see [Traefik Migration Documentation - v2.10](https://doc.traefik.io/traefik/v2.10/migration/v2/#v210). This role handles the required CRD (custom resource definition) updates mentioned in the upgrade documentation. Also the RBAC update mentioned there is handled by the Traefik Helm chart itself.

From Helm chart version `18.x` all changes of the chart are documented in the [releases page](https://github.com/traefik/traefik-helm-chart/releases). For the changelog of Traefik see [releases page](https://github.com/traefik/traefik/releases).

CRDs API Group `traefik.containo.us` is deprecated, and its support will end starting with Traefik `v3`. Please use the API Group `traefik.io` instead.

CRDs API Version `traefik.io/v1alpha1` will not be supported in Traefik `v3` itself. However, an automatic migration path to the next version will be available.

Helm chart version >= `23.0.0` requires Kubernetes >= `1.22`.

Besides that:

- update Helm chart to version `23.0.1`
- update Traefik from version `2.9.4` to `2.10.1`
- `templates/traefik_values_default.yml.j2`: `image.name` was renamed to `image.repository` (see [v22.0.0](https://github.com/traefik/traefik-helm-chart/releases/tag/v22.0.0))
- `templates/traefik_values_default.yml.j2`: introduce `image.registry`
- update Traefik CRDs
- refactor Molecule test

## 4.0.1+16.2.0

- fix yamllint issues (remove blank lines in CRD files)
- add `.yamllint`

## 4.0.0+16.2.0

- update Helm chart to `v16.2.0`
- use Traefik `v2.9.4`

## 3.1.1+10.24.4

- fix yamllint issues (remove blank lines in CRD files)

## 3.1.0+10.24.4

- update Helm chart to `v10.24.4`
- use Traefik `v2.8.7`
- add Github release action to push new release to Ansible Galaxy

## 3.0.0+10.24.2

- update Helm chart to `v10.24.2`
- use Traefik `v2.8.4`
- introduce `traefik_delegate_to` variable. Previously this was hard coded to `127.0.0.1` and it's also the default of this variable.
- introduce `traefik_helm_show_commands` variable (see README for more information)
- introduce `traefik_template_output_directory` variable (see README for more information)
- introduce `Molecule` tests
- don't use `shell` module anymore to execute `helm` command. Now `kubernetes.core.helm*` modules are used.
- use two underscores for internal variables
- ansible-lint: fix various issues like using FQDN Ansible module names now
- add `requirements.yml`
- add `collections.yml`

## 2.8.3+10.24.1

- update Helm chart to `v10.24.1`
- use Traefik `v2.8.3`
- update Traefik CRDs to v2.8.x

## 2.6.6+10.19.5

- update Helm chart to `v10.19.5`
- use Traefik `v2.6.6`
- introduce `traefik_install_crds` variable (see README for more information)

## 2.3.0+10.14.1

- update Helm chart to `v10.14.1`
- use Traefik `v2.6.1`

**Important note:**: Make sure that the `middlewarestcp` CRD is updated before you upgrade e.g. via `curl https://raw.githubusercontent.com/traefik/traefik-helm-chart/v10.14.1/traefik/crds/middlewarestcp.yaml | kubectl apply -f -`

## 2.2.0+10.9.1

- update Helm chart to `v10.9.1`
- use Traefik `v2.5.6`
- fix linting issues
- remove unneeded directories

## 2.1.1+10.3.2

- update Helm chart to `v10.3.2`
- use Traefik `v2.5.2`

## 2.1.0+10.3.0

- update Helm chart to `v10.3.0`
- use Traefik `v2.5.0`

The following note is only relevant if you upgrade from a chart version `< v10.3.0` (which introduced Traefik `v2.5.0`). For new installations it's not relevant.

**Important note:** Before upgrading read this issue [](https://github.com/traefik/traefik-helm-chart/issues/359) as it may hit you if you upgrade from a chart version `< v10.3.0` (which introduced Traefik `v2.5.0`). If you already have a `CustomResourceDefinition` (CRD) called `middlewaretcps.traefik.containo.us` (check with `kubectl describe crds middlewaretcps.traefik.containo.us`) upgrade shouldn't be an issue.

You've basically two options:

- make sure that the CRD is installed before you upgrade e.g. via `curl https://raw.githubusercontent.com/traefik/traefik-helm-chart/master/traefik/crds/middlewarestcp.yaml | kubectl apply -f -`
- delete the Traefik resources via `ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=delete k8s.yml` and install again via `ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=install k8s.yml`. This also adds the new CRD `middlewaretcps.traefik.containo.us`. This of course will uninstall Traefik! So all ingress traffic will be offline until you install Traefik again.

## 2.0.1+10.1.1

- update README

## 2.0.0+10.1.1

- update Helm chart to `v10.1.1`
- Traefik Helm chart version >= `v10.x.x` needs at least Kubernetes `v1.16`
- use Traefik `v2.4.13`
- Default Traefik version is now specified in `templates/traefik_values_default.yml.j2`

## 1.1.0+9.19.1

- update Helm chart to `v9.19.1` (uses Traefik `v2.4.8`)

**Important note:** Before upgrading read this issue [](https://github.com/traefik/traefik-helm-chart/issues/359) as it may hit you if you upgrade from a chart version `< v9.13.0` (which introduced Traefik `v2.4.0`). If you already have a `CustomResourceDefinition` (CRD) called `serverstransports.traefik.containo.us` (check with `kubectl get crds serverstransports.traefik.containo.us`) upgrade shouldn't be an issue.

You've basically two options:

- make sure that the CRD is installed before you upgrade e.g. via `curl https://raw.githubusercontent.com/traefik/traefik-helm-chart/master/traefik/crds/serverstransports.yaml | kubectl apply -f -`
- delete the Traefik resources via `ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=delete k8s.yml` and install again via `ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=install k8s.yml`. This also adds the new CRD `serverstransports.traefik.containo.us`. This of course will uninstall Traefik! So all ingress traffic will be offline until you install Traefik again.

## 1.0.0+9.12.3

- initial commit for Traefik `v2.3.6`
