# portainer-tailscaled

A Docker Compose setup that combines Portainer CE with Tailscale for secure, remote container management. Access your Portainer dashboard from anywhere through your Tailscale network.

## Features

- 🐳 Full Portainer CE container management
- 🌐 Secure access via Tailscale network
- 🔒 No need to expose ports to the public internet
- 🔄 Persistent Tailscale state and Portainer data
- 🤖 Includes Portainer Agent for enhanced management
- 🚀 Easy deployment with Docker Compose

## Prerequisites

- Docker and Docker Compose installed
- A Tailscale account and auth key ([Get one here](https://login.tailscale.com/admin/settings/keys))
- Docker socket access on the host

## Quick Start

1. **Clone/navigate to the repository:**

   ```bash
   cd portainer-tailscaled
   ```

2. **Copy the example configuration:**

   ```bash
   cp docker-compose.example.yml docker-compose.yml
   ```

3. **Edit `docker-compose.yml` with your settings:**

   ```bash
   nano docker-compose.yml
   ```

   Update the following values:
   - `TS_AUTHKEY`: Your Tailscale auth key (get one from [Tailscale admin](https://login.tailscale.com/admin/settings/keys))
   - `hostname`: Optional - customize the Tailscale machine name
   - For rootless Docker: Update the agent volumes path (see [Customization](#rootless-docker))

4. **Start the services:**

   ```bash
   docker compose up -d
   ```

5. **Check logs:**

   ```bash
   docker compose logs -f
   ```

6. **Access Portainer:**
   - Find your Tailscale IP: `tailscale ip` or check the [Tailscale admin console](https://login.tailscale.com/admin/machines)
   - Open `https://<tailscale-ip>:9443` or `http://<tailscale-ip>:9000`

## Configuration

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `TS_AUTHKEY` | Yes | Tailscale authentication key | `tskey-auth-...` |
| `TS_STATE_DIR` | Yes | Directory for Tailscale state | `/var/lib/tailscale` |
| `TS_USERSPACE` | No | Run in userspace mode (recommended for containers) | `true` |
| `TS_EXTRA_ARGS` | No | Additional Tailscale arguments | `--advertise-tags=tag:container` |

### Ports

| Port | Service | Description |
|------|---------|-------------|
| `9443` | Portainer | HTTPS web interface |
| `9000` | Portainer | HTTP web interface |
| `8000` | Portainer | Edge Agent server (optional) |
| `9001` | Agent | Portainer Agent communication |

### Volume Mounts

| Path | Description |
|------|-------------|
| `./tailscale/state` | Persistent Tailscale state |
| `./portainer_data` | Portainer configuration and data |
| `/var/run/docker.sock` | Docker socket for container management |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Tailscale Network                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  tailscale-portainer container                              │
│  ├── Tailscale daemon (userspace mode)                      │
│  └── Exposes ports: 9443, 9000, 8000                        │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────────┐
│  portainer container    │     │  portainer_agent container  │
│  (network_mode: service)│     │  (port 9001)                │
│  Web UI & API           │     │  Enhanced management        │
└─────────────────────────┘     └─────────────────────────────┘
```

## How It Works

1. The Tailscale container connects to your tailnet using the provided auth key
2. Portainer runs in the same network namespace as Tailscale (`network_mode: service`)
3. All Portainer traffic is routed through Tailscale
4. You can access Portainer securely from any device on your Tailscale network
5. The Agent provides additional capabilities like volume browsing and host management

## Customization

### Rootless Docker

If using rootless Docker, adjust the volumes path for the agent:

```yaml
agent:
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ~/.local/share/docker/volumes:/var/lib/docker/volumes
    - /:/host
```

### Without Edge Agents

If you don't plan to use Edge Agents, you can remove port 8000:

```yaml
tailscale-portainer:
  ports:
    - 9443:9443
    - 9000:9000
    # Remove port 8000
```

### Using Tags

To tag the machine in Tailscale ACLs:

```yaml
environment:
  - TS_EXTRA_ARGS=--advertise-tags=tag:container,tag:portainer
```

## Troubleshooting

### Cannot access Portainer web UI

- Verify Tailscale is connected: `docker compose logs tailscale-portainer`
- Check the Tailscale IP in the admin console
- Ensure you're connected to the same Tailscale network

### Tailscale authentication fails

- Verify your `TS_AUTHKEY` is valid and not expired
- Generate a new key from the [Tailscale admin console](https://login.tailscale.com/admin/settings/keys)
- Use a reusable key for persistent deployments

### Portainer shows no containers

- Ensure Docker socket is properly mounted
- Check permissions on `/var/run/docker.sock`
- Verify the portainer container can access the socket: `docker compose logs portainer`

### Agent connection issues

- Ensure the agent is running: `docker compose ps`
- Check agent logs: `docker compose logs agent`
- Verify network connectivity between Portainer and Agent

## Security Considerations

- 🔐 **Tailscale Auth Keys**: Use short-lived or reusable keys with limited capabilities
- 🔒 **Docker Socket**: Access to Docker socket grants root-equivalent permissions
- 🌐 **Network Access**: Portainer is only accessible via Tailscale, not the public internet
- 🏷️ **ACL Tags**: Use Tailscale ACLs to restrict which devices can access Portainer
- 🔄 **Updates**: Regularly update Portainer and Tailscale images for security patches

## Related Resources

- [Portainer Documentation](https://docs.portainer.io/)
- [Tailscale Documentation](https://tailscale.com/kb/)
- [Tailscale in Docker](https://tailscale.com/kb/1282/docker)

## License

This project is provided as-is without warranty. Use at your own risk.

## Support

For issues and questions:

- Open an issue on GitHub
- Check existing issues for similar problems
- Review container logs for error messages
