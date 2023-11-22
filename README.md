# Monitoring

A combined Helm Chart to monitor GNUU server.

install with

```bash
helm -n monitoring upgrade -i monitoring --set grafana.adminPassword=xxxxx --set grafana.grafana.ini.auth.github.client_id=xxxx --xet grafana.grafana.ini.auth.github.client_secret=xxxxx ...
```

or provide the credentials in extra secret.yaml

## Components

### Kepler

Monitor Power consumption as described in [sustainable-computing](https://github.com/sustainable-computing-io/kepler)

### Prometheus

A plain prometheus-server without any CRD requirements

### Grafana

Grafana installed from the origin Helm Chart. Includes dashboard for Kepler, K3s, Kubernetes, Logs. Beware of the non static datasource address. This must be the prometheus/loki service endpoint. Look for `SETME: service address `

### Loki-Stack

Loki promtail will collect container logs and push to Loki stack. This is include in Grafana as datasource to query log time series.

### Config parameters:

#### kepler

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `kepler.global.namespace` | string | `"monitoring"` |  |
| `kepler.image.pullPolicy` | string | `"Always"` |  |
| `kepler.image.repository` | string | `"mtr.devops.telekom.de/caas/kepler"` |  |
| `kepler.image.tag` | string | `"release-0.6.1"` |  |
| `kepler.resources.limits.cpu` | string | `"800m"` |  |
| `kepler.resources.limits.memory` | string | `"1024Mi"` |  |
| `kepler.resources.requests.cpu` | string | `"200m"` |  |
| `kepler.resources.requests.memory` | string | `"128Mi"` |  |
| `kepler.serviceMonitor.enabled` | bool | `false` |  |

#### prometheus

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prometheus.alertmanager.enabled` | bool | `false` |  |
| `prometheus.configmapReload.prometheus.image.repository` | string | `"mtr.devops.telekom.de/kubeprometheusstack/prometheus-config-reloader"` |  |
| `prometheus.configmapReload.prometheus.image.tag` | string | `"v0.67.0"` |  |
| `prometheus.extraConfigmapMounts[0].configMap` | string | `"monitoring-scrapings"` |  |
| `prometheus.extraConfigmapMounts[0].mountPath` | string | `"/etc/config"` |  |
| `prometheus.extraConfigmapMounts[0].name` | string | `"monitoring-scrapings"` |  |
| `prometheus.extraConfigmapMounts[0].readOnly` | bool | `true` |  |
| `prometheus.extraConfigmapMounts[0].subPath` | string | `""` |  |
| `prometheus.extraScrapeConfigs` | string | `"- job_name: kepler\n  static_configs:\n  - targets: [\"kepler:9102\"]\n  metric_relabel_configs:\n  - action: replace\n    regex: (.*)\n    source_labels:\n    - container_namespace\n    target_label: namespace\n  relabel_configs:\n  - action: replace\n    regex: (.*)\n    replacement: $1\n    source_labels:\n    - __meta_kubernetes_pod_node_name\n    target_label: instance\n  scrape_interval: 1m\n  scrape_timeout: 30s"` |  |
| `prometheus.image.repository` | string | `"mtr.devops.telekom.de/kubeprometheusstack/prometheus"` |  |
| `prometheus.kube-state-metrics.enabled` | bool | `true` |  |
| `prometheus.prometheus-node-exporter.enabled` | bool | `true` |  |
| `prometheus.prometheus-pushgateway.enabled` | bool | `false` |  |
| `prometheus.releaseNamespace` | bool | `false` |  |

#### loki-stack

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `loki-stack.promtail.config.clients[0].url` | string | `"http://monitoring-loki:3100/loki/api/v1/push"` |  |
| `loki-stack.promtail.config.logLevel` | string | `"info"` |  |
| `loki-stack.promtail.config.serverPort` | int | `3101` |  |
| `loki-stack.promtail.enabled` | bool | `true` |  |
| `loki-stack.test_pod.enabled` | bool | `false` |  |

#### grafana

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `grafana."grafana.ini"."auth.anonymous".enabled` | bool | `false` |  |
| `grafana."grafana.ini"."auth.anonymous".org_role` | string | `"Viewer"` |  |
| `grafana."grafana.ini"."auth.github".enabled` | bool | `true` |  |
| `grafana."grafana.ini"."auth.github".role_attribute_path` | string | `"contains(groups[*], '@gnuu-de/operator') &&  'GrafanaAdmin' || 'Viewer'"` |  |
| `grafana."grafana.ini".analytics.check_for_updates` | bool | `false` |  |
| `grafana."grafana.ini".auth.disable_login_form` | bool | `false` |  |
| `grafana."grafana.ini".server.root_url` | string | `"https://monitoring.gnuu.de"` |  |
| `grafana."grafana.ini".users.auto_assign_org_role` | string | `"Viewer"` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".apiVersion` | int | `1` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".providers[0].disableDeletion` | bool | `false` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".providers[0].editable` | bool | `true` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".providers[0].folder` | string | `""` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".providers[0].name` | string | `"default"` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".providers[0].options.path` | string | `"/var/lib/grafana/dashboards/default"` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".providers[0].orgId` | int | `1` |  |
| `grafana.dashboardProviders."dashboardproviders.yaml".providers[0].type` | string | `"file"` |  |
| `grafana.dashboardsConfigMaps.default` | string | `"monitoring-dashboards"` |  |
| `grafana.datasources."datasources.yaml".apiVersion` | int | `1` |  |
| `grafana.datasources."datasources.yaml".datasources[0].access` | string | `"proxy"` |  |
| `grafana.datasources."datasources.yaml".datasources[0].isDefault` | bool | `true` |  |
| `grafana.datasources."datasources.yaml".datasources[0].name` | string | `"Prometheus"` |  |
| `grafana.datasources."datasources.yaml".datasources[0].type` | string | `"prometheus"` |  |
| `grafana.datasources."datasources.yaml".datasources[0].url` | string | `"http://monitoring-prometheus-server"` |  |
| `grafana.datasources."datasources.yaml".datasources[1].access` | string | `"proxy"` |  |
| `grafana.datasources."datasources.yaml".datasources[1].isDefault` | bool | `false` |  |
| `grafana.datasources."datasources.yaml".datasources[1].name` | string | `"Loki"` |  |
| `grafana.datasources."datasources.yaml".datasources[1].type` | string | `"loki"` |  |
| `grafana.datasources."datasources.yaml".datasources[1].url` | string | `"http://monitoring-loki:3100"` |  |
| `grafana.image.registry` | string | `"mtr.devops.telekom.de"` |  |
| `grafana.image.repository` | string | `"kubeprometheusstack/grafana"` |  |
| `grafana.ingress.annotations."cert-manager.io/cluster-issuer"` | string | `"letsencrypt"` |  |
| `grafana.ingress.annotations."kubernetes.io/ingress.class"` | string | `"traefik"` |  |
| `grafana.ingress.annotations."traefik.backend.loadbalancer.sticky"` | string | `"true"` |  |
| `grafana.ingress.annotations."traefik.frontend.passHostHeader"` | string | `"true"` |  |
| `grafana.ingress.annotations."traefik.ingress.kubernetes.io/frontend-entry-points"` | string | `"http,https"` |  |
| `grafana.ingress.annotations."traefik.ingress.kubernetes.io/redirect-entry-point"` | string | `"https"` |  |
| `grafana.ingress.enabled` | bool | `true` |  |
| `grafana.ingress.hosts[0]` | string | `"monitoring.gnuu.de"` |  |
| `grafana.ingress.tls[0].hosts[0]` | string | `"monitoring.gnuu.de"` |  |
| `grafana.ingress.tls[0].secretName` | string | `"monitoring-gnuu-de"` |  |
| `grafana.initChownData.enabled` | bool | `false` |  |
| `grafana.persistence.inMemory.enabled` | bool | `true` |  |
| `grafana.rbac.namespaced` | bool | `true` |  |
| `grafana.testFramework.enabled` | bool | `false` |  |

### Credits

Frank Kloeker eumel@admin.gnuu.de

