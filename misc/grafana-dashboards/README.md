# Grafana Dashboards Reference

Using the following script, retrieve and modify the Grafana dashboards. The script will generate Kubernetes ConfigMaps containing the dashboard JSON data.

```bash
GRAFANA_DASHBOARD_URLS=(
"https://grafana.com/api/dashboards/13277/revisions/latest/download"
"https://grafana.com/api/dashboards/7645/revisions/latest/download"
"https://grafana.com/api/dashboards/7639/revisions/latest/download"
"https://grafana.com/api/dashboards/11829/revisions/latest/download"
"https://grafana.com/api/dashboards/7636/revisions/latest/download"
"https://grafana.com/api/dashboards/7630/revisions/latest/download"
"https://grafana.com/api/dashboards/15859/revisions/latest/download"
"https://grafana.com/api/dashboards/10890/revisions/latest/download"
"https://grafana.com/api/dashboards/10642/revisions/latest/download"
"https://grafana.com/api/dashboards/13396/revisions/latest/download"
)

for url in ${GRAFANA_DASHBOARD_URLS[@]}; do
  echo "###################################"
  echo "Working on URL $url..."
  
  DASHBOARD_TITLE=$(curl -s "$url" | jq -r '.title' | tr '[:upper:]' '[:lower:]' | tr ' ' '-')
  echo "Dashboard title: $DASHBOARD_TITLE"
  
  DASHBOARD_JSON=$(curl -s "$url" | 
  jq -c 'del(.__inputs)' | 
  sed 's|${DS_THANOS-MASTER}|Prometheus|g' | 
  sed 's|${DS_PROMETHEUS}|Prometheus|g')

  cat <<EOF > "$DASHBOARD_TITLE.yaml"
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-$DASHBOARD_TITLE
  namespace: kube-prometheus-stack
  labels:
    grafana_dashboard: "1"
  annotations:
    grafana_folder: service-mesh
data:
  $DASHBOARD_TITLE.json: |
    $DASHBOARD_JSON
EOF

done
```

Apply the ConfigMaps to the Kubernetes cluster to import the dashboards into Grafana.

```bash
kubectl apply -f .
```

Finally, ensure the dashboards are now available in Grafana under the `Service Mesh` folder.