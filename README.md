# Bind Exporter

Export BIND(named/dns) v9+ service metrics to Prometheus. This is a fork of the Digital Ocean repo, and modified to run only one exporter that can export BIND metrics from multiple remote BIND endpoints. The target is exporter from the `target` GET parameter of the URL.

## Getting started

```bash
go get github.com/digitalocean/bind_exporter
cd $GOPATH/src/github.com/digitalocean/bind_exporter
make
./bind_exporter [flags]
```

Grafana Dashboard: https://grafana.com/dashboards/1666

## Example Prometheus configuration

This is an example for Prometheus configuration, asuming that BIND exporter listens on localhost:9119 and there are three remote BIND servers with IPs 10.1.1.4, 10.1.1.5, 10.1.1.6, all listening on 8053 as statistics port.

```
scrape_configs:
  - job_name: 'bind'
    static_configs:
      - targets:
        - 10.1.1.4  # remote BIND server 1
        - 10.1.1.5  # remote BIND server 3
        - 10.1.1.6  # remote BIND server 2
    metrics_path: /bind
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9119  # The BIND exporter's real hostname:port.
```

## Troubleshooting

Make sure BIND was built with libxml2 support. You can check with the following
command: `named -V | grep libxml2`.

Configure BIND to open a statistics channel. 

```
statistics-channels {
  inet 0.0.0.0 port 8053 allow { <BIND Exporter IP>; };
};
```

---

