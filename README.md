<!-- markdownlint-disable MD024 -->

# 🐋 turingpi-homelab

This repository contains Docker Compose files for deploying services on a Turing Pi cluster. Some services use Docker Swarm, while others (like Home Assistant) use standard Docker Compose.

- Swarm
  - [Portainer](https://www.portainer.io) — web-based container management
  - [Pi-hole](https://pi-hole.net) — network-wide ad blocking

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

``` bash
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

    Accessible at http://\<node-ip\>/admin

- **Home Assistant**

    Deploy:

    ```bash
    sudo docker compose -f homeassistant-compose.yaml up -d
    ```

    Accessible at http://\<node-ip\>:8123

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
