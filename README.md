# monitoring

### Install prometheus

```
wget https://github.com/prometheus/prometheus/releases/download/v2.39.0/prometheus-2.39.0.linux-amd64.tar.gz
```

```
sudo tar -xzf prometheus-2.39.0.linux-amd64.tar.gz \
    --transform 's/prometheus-2.39.0.linux-amd64/prometheus/' \
    -C /etc
```

```
sudo mv /etc/prometheus/prometheus /usr/local/bin/
```

Check
```
prometheus --version
```

Config
```
cat /etc/prometheus/prometheus.yml
```

Run
```
prometheus --config.file "/etc/prometheus/prometheus.yml" &
```

Test
```
curl localhost:9090/metrics
```

### Install Node Exporter

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar -xzf node_exporter-*
sudo cp node_exporter-*/node_exporter /usr/local/bin/
```

Check
```
node_exporter --version
```

Run
```
node_exporter &
```

Test
```
curl localhost:9100/metrics
curl localhost:9100/metrics | grep node_cpu_seconds_total
curl localhost:9100/metrics | grep node_memory_MemTotal_bytes
```

Add
```
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]
```

PromQuery
```
{job="prometheus"}
{job="node"}
node_cpu_seconds_total{job="node"}
100 - avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance) * 100
(node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_memory_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100
(node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_free_bytes{mountpoint="/"}) / node_filesystem_size_bytes{mountpoint="/"} * 100
predict_linear(node_filesystem_free_bytes{mountpoint="/"}[1h], 4*3600) < 0 # Dự đoán trong 4 giờ có full disk không
```

Preparing namespace
```
kubectl create ns monitoring
kubectl create ns metrics-server
```

Install Ingress Controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Get ingress log
```
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller
```

Workaround if metrics server got certificate error -> Push this line to metrics-server deployment
```
- --kubelet-insecure-tls
```
