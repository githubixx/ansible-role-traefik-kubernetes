# Changelog

## 5.1.2+23.1.0

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
