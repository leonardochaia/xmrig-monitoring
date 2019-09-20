# XMRig Mining Monitoring with Grafana and Prometheus

Credits to Dockprom https://github.com/stefanprodan/dockprom

## How this works

The basic idea is that, Prometheus scrapes metrics via HTTP in a certain format. For exposing the metrics in the Prometheus format we use what we call "exporters". These exporters read system/software values and export them via HTTP for Prometheus to scrap.

For obtaining system metrics like CPU, RAM, Temperature we use [node-exporter](https://github.com/prometheus/node_exporter)
For XMRig metrics we use [xmrig-exporter](https://github.com/sbrudenell/xmrig_exporter)

### XMRig setup

#### Already existing XMRIG

If you are already mining using XMRig, make sure you expose the HTTP API, i.e

```bash
xmrig --http-enabled --http-host 0.0.0.0 --http-port 8080 --api-worker-id your-unique-worker-id # other params here
```

You'll then need to run `node-exporter` on your mining rig. This needs to run on your mining rig so it can obtain system stats.

`xmrig-exporter` does not really need to run on your miner, although it's most common to run your exporter next to your apps.

For runinng those, you can use the provided [compose file](./mining/docker-compose.yml) or you can run them directly on the host OS, that's up to you.

#### XMRig on Docker

If you want to run the miner itself on Docker you can simply run the provided [compose file](./mining/docker-compose.yml) on your mining rig.
First edit the relevant parameters (i.e pool, algorithm) and just `docker-compose up -d`
This will spin up `xmrig`, `node-exporter` and `xmrig-exporter`

Note that I'm using an `xmrig` image that I built my self, I recommend you don't trust me and build it yourself instead. You can get the Dockerfile from [here](https://github.com/leonardochaia/docker-xmrig) (you might alsa wanna build the GCC base image too)

### Monitoring setup

To run the monitoring stack, we'll use another host. You probably don't want to run Prometheus and Grafana on your mining rig.
You can use an rpi, or a VM or whatever.

To run Prometheus and Grafana, first edit the `/monitoring/.env` file with a new user/password for Grafana.

The compose file located on `/monitoring` includes Prometheus, Grafana, alert-manager and nginx-proxy.
You can setup alert-manager to send you hashrate alarms or temperature (rules are not currently setup, but the software is)
nginx-proxy is used to access the URLs via fqdn instead of ip/port addresses. It uses .localhost as an example but you could also use your custom FQDN with your DNS server.

You'll then need to edit the `/monitoring/prometheus/prometheus.yml` to provide the FQDN or IP address of your mining rigs for scraping.
Change the job name and the targets.

```yml
scrape_configs:
  - job_name: "my-miner-job"
    static_configs:
      - targets: ["<ip or miner fqdn>:9100"] # node-exporter
      - targets: ["<ip or miner fqdn>:9189"] # xmrig-exporter
```

If you `docker-compose up` on `/monitoring` you should be able to access `grafana.localhost` and login with your creds.
It should create the Prometheus data source and dashboards by itself so it should be ready to get started.

## Testing

For testing the setup, you can run the miner and monitoring on the same host on your workstation.
In the `scrape_configs` add your workstation IP

## Troubleshooting

You can access `prometheus.localhost` and view the targets to see if prometheus is able to reach your miners.

Also `docker logs prometheus` `docker logs grafana`

Feel free to create new issues and I'll try to help you.
