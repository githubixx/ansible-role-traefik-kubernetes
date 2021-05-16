Changelog
---------

**1.1.0+9.19.1**

- update Helm chart to `v9.19.1` (uses Traefik `v2.4.8`)

**Important note:** Before upgrading read this issue [](https://github.com/traefik/traefik-helm-chart/issues/359) as it may hit you if you upgrade from a chart version `< v9.13.0` (which introduced Traefik `v2.4.0`). If you already have a `CustomResourceDefinition` (CRD) called `serverstransports.traefik.containo.us` (check with `kubectl get crds serverstransports.traefik.containo.us`) upgrade shouldn't be an issue.

You've basically two options:
- make sure that the CRD is installed before you upgrade e.g. via `curl https://raw.githubusercontent.com/traefik/traefik-helm-chart/master/traefik/crds/serverstransports.yaml | kubectl apply -f -`
- delete the Traefik resources via `ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=delete k8s.yml` and install again via `ansible-playbook --tags=role-traefik-kubernetes --extra-vars action=install k8s.yml`. This also adds the new CRD `serverstransports.traefik.containo.us`. This of course will uninstall Traefik! So all ingress traffic will be offline until you install Traefik again.

**1.0.0+9.12.3**

- initial commit for Traefik `v2.3.6`
