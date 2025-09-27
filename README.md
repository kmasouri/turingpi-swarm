<!-- markdownlint-disable MD024 -->

# 🐋 turingpi-homelab

This repository contains Docker Compose files for deploying services on a Turing Pi cluster. Some services use Docker Swarm, while others (like Home Assistant) use standard Docker Compose.

- Swarm

  - [Portainer](https://www.portainer.io) — web-based container management
  - [Pi-hole](https://pi-hole.net) — network-wide ad blocking

    - Monitoring (Prometheus, Grafana, Node Exporter, Blackbox Exporter)

  - [Portal](https://github.com/kmasouri/homelab-portal) — portal for your homelab apps and bookmarks

- Compose
  - [Home Assistant](https://www.home-assistant.io) — home automation platform

## 📦 Prerequisites

- Docker Engine installed (no Docker Desktop required)
- Docker Compose installed
- Docker Swarm initialized:

  ```bash
  sudo docker swarm init --advertise-addr <your-node-ip>
  ```

- At least one manager node (for Swarm services)

## 🚀 Deploying a Service

### Swarm

Deploy a service stack using:

```bash
sudo docker stack deploy -c \<compose-file\>.yaml \<stack-name\>
```

Example:

```bash
sudo docker stack deploy -c portainer-compose.yaml portainer
```

To remove a stack:

```bash
sudo docker stack rm \<stack-name\>
```

Example:

```bash
sudo docker stack rm portainer
```

### Compose

Deploy a service using Docker Compose:

```bash
sudo docker compose -f \<compose-file\>.yaml up -d
```

Example:

```bash
sudo docker compose -f homeassistant-compose.yaml up -d
```

To stop a service:

```bash
sudo docker compose -f \<compose-file\>.yaml down
```

Example:

```bash
sudo docker compose -f homeassistant-compose.yaml down
```

## 🔗 Networks

Stacks use overlay networks for cross-node communication. Each stack defines its own network (e.g., portainer_network, pihole_network) to avoid conflicts.

Example block inside a Compose file:

```yaml
networks:
  network:
    driver: overlay
    attachable: true
```

## 💾 Volumes (Persistent Data)

These services use Docker volumes to store configuration and data so they survive restarts and upgrades:

- Portainer → `portainer_data`
- Pi-hole → `pihole_data`
- Home Assistant → `homeassistant_volume`
- Prometheus → `prometheus_volume`
- Grafana → `grafana_volume`

You don’t need to manage these manually for day-to-day use. Just keep in mind that if you ever rebuild your cluster from scratch, restoring these volumes will bring back your configs.

## Updating Services

### Swarm

To update a service to a new image version:

```bash
sudo docker service update --image \<new-image\> \<stack\>_\<service\>
```

Example:

```bash
sudo docker service update --image portainer/portainer-ce:latest portainer_portainer
```

### Compose

To update a service, pull the latest image and recreate the container:

```bash
# Pull the latest image
sudo docker compose -f \<compose-file\>.yaml pull

# Recreate and start the container
sudo docker compose -f \<compose-file\>.yaml up -d
```

Example:

```bash
sudo docker compose -f homeassistant-compose.yaml pull

sudo docker compose -f homeassistant-compose.yaml up -d
```

Alternatively, update the image: tag in your Compose file and redeploy the stack.

## 📜 Current Stacks

- **Portainer**

  Deploy:

  ```bash
  sudo docker stack deploy -c portainer-compose.yaml portainer
  ```

  Accessible at http://\<node-ip\>:9000

- **Pi-hole**

  Deploy:

  ```bash
  sudo docker stack deploy -c pihole-compose.yaml pihole
  ```

  Accessible at http://\<node-ip\>:8080/admin

- **Home Assistant**

  Deploy:

  ```bash
  sudo docker compose -f homeassistant-compose.yaml up -d
  ```

  Accessible at http://\<node-ip\>:8123

- **Monitoring (Prometheus + Grafana + Node Exporter + Blackbox Exporter)**

  Deploy:

  ```bash
  sudo docker stack deploy -c monitoring-compose.yaml monitoring
  ```

  Access:

  - Prometheus: http://`<node-ip>`:9090
  - Grafana: http://`<node-ip>`:3000
  - Blackbox Exporter probe endpoint: http://`<node-ip>`:9115
  - Node Exporter metrics per node (if port published): http://`<node-ip>`:9100/metrics

  Notes:

  - Node Exporter runs in global mode (one per node)
  - This setup uses explicit static targets in `prometheus.yaml` for clarity.
  - You can switch back to Swarm DNS discovery (e.g. `tasks.monitoring_prometheus-node-exporter`) if you prefer dynamic service-based discovery.

- **Homelab Portal**

  A lightweight static portal (single-page app) served by Nginx to provide quick links to the main homelab services.

  Deploy:

  ```bash
  sudo docker stack deploy -c homelab-portal-compose.yaml portal
  ```

  Accessible at http://`<node-ip>`/ (port 80)

  Customization is done via the mounted `homelab-portal-config.json` file. Update it and then force a service update to refresh cached content:

  ```bash
  sudo docker service update --force portal_homelab-portal
  ```

  [See full configuration reference.](https://github.com/kmasouri/homelab-portal?tab=readme-ov-file#%EF%B8%8F-configuration)

## 🧹 Maintenance

- List stacks

  ```bash
  docker stack ls
  ```

- List services in a stack

  ```bash
  docker stack services <stack>
  ```

- List services created by compose

  ```bash
  sudo docker compose -f \<compose-file\>.yaml ps
  ```

- List volumes

  ```bash
  docker volume ls
  ```

- List networks

  ```bash
  docker network ls
  ```

- Remove unused resources

  ```bash
  docker network prune
  docker volume prune
  ```
