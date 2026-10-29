# Home Assistant on RISC-V

A Docker-based Home Assistant deployment for RISC-V systems, using a dedicated Tailscale sidecar for remote access.

This project provides a starting point for running Home Assistant on RISC-V hardware. Depending on your board, operating system, and hardware integrations, additional configuration may be required.

## Features

- Local Docker image build for RISC-V
- Persistent Home Assistant configuration
- Tailscale sidecar for remote access
- Optional USB device passthrough for integrations such as Zigbee
- Customizable build dependencies

## Requirements

- A supported RISC-V 64-bit Linux system
- Docker Engine
- Docker Compose v2
- Internet connectivity for image building and initial setup
- Sufficient storage and memory for building Home Assistant and its dependencies

## Project structure

```text
.
├── compose.yaml
├── homeassistant/
│   ├── Dockerfile
│   └── config/
│       └── configuration.yaml
└── homeassistant-network/
    ├── Dockerfile
    ├── start.sh
    └── tailscale/
```

The exact directory structure may vary depending on your local setup.

## Configuration

### 1. Home Assistant

The Home Assistant container is built locally using the provided Dockerfile.

The configuration directory is mounted at `/config`, allowing configuration files to persist across container recreation.

The default configuration can be kept minimal:

```yaml
default_config:

mobile_app:
```

### 2. Tailscale

The `homeassistant-network` container runs `tailscaled`. The Home Assistant container shares its network namespace through:

```yaml
network_mode: "service:homeassistant-network"
```

Tailscale state is persisted using the following volume:

```yaml
volumes:
  - ./homeassistant-network/tailscale:/var/lib/tailscale
```

The network container also requires access to `/dev/net/tun` and the network capabilities configured in `compose.yaml`.

**Important:** Starting the container does not necessarily connect it to your tailnet. You must complete the Tailscale authentication and connection setup.

### 3. Optional device configuration

If you use hardware such as a Zigbee coordinator, add a `devices` entry to the `homeassistant` service in `compose.yaml`.

Check the device path on the host:

```bash
ls -l /dev/serial/by-id/
```

For example:

```yaml
devices:
  - /dev/ttyUSB0:/dev/ttyUSB0
```

Replace `/dev/ttyUSB0` with the actual device path on your system. Device passthrough is optional and may require additional permissions depending on your hardware.

### 4. Port configuration

The default configuration maps port `8123` on the host to port `8123` in the container:

```yaml
ports:
  - "8123:8123"
```

Home Assistant can then be accessed through:

```text
http://<HOST_IP>:8123
```

Replace `<HOST_IP>` with the host's IP address.

## Build and start

Run the following commands from the project root directory.

### 1. Build the images

```bash
docker compose build
```

### 2. Start the containers

```bash
docker compose up -d
```

Check the container status:

```bash
docker compose ps
```

Check the logs if necessary:

```bash
docker compose logs -f homeassistant-network
docker compose logs -f homeassistant
```

### 3. Connect Tailscale

Starting the containers does not replace the Tailscale connection setup.

Open a shell in the network container:

```bash
docker compose exec homeassistant-network bash
```

Then run:

```bash
tailscale up
```

Follow the displayed instructions to authenticate the device if prompted.

If the container reports that it is already connected, verify its status:

```bash
tailscale status
```

If authentication or connection setup is incomplete, follow the instructions printed by Tailscale.

**Run `tailscale up` when the device needs initial authentication or connection setup.** The persistent state directory is intended to retain Tailscale state across container recreation, so reauthentication should not normally be necessary every time the containers restart.

### 4. Access Home Assistant

Once Home Assistant is running and Tailscale is connected, access Home Assistant using the Tailscale IP address or MagicDNS hostname of the network container:

```text
http://<TAILSCALE_IP>:8123
```

Replace `<TAILSCALE_IP>` with the Tailscale IP address of the network container. Ensure that your tailnet access rules permit the connection.

## Troubleshooting

### Home Assistant does not start

Inspect the logs:

```bash
docker compose logs --tail=200 homeassistant
```

### Tailscale is not connected

Check its status:

```bash
docker compose exec homeassistant-network tailscale status
```

If initial setup has not been completed, enter the container and run:

```bash
docker compose exec homeassistant-network tailscale up
```

### A hardware device is unavailable

Check the host device path and confirm that the correct mapping is configured in `compose.yaml`.

### The build fails

RISC-V package availability and native dependency support may differ from x86-64 systems. Depending on the error, you may need to modify the Dockerfile, install additional dependencies, or adjust package versions.

## Updating

To rebuild the images after modifying build-related files:

```bash
docker compose build
docker compose up -d
```

To force a complete rebuild without using the Docker build cache:

```bash
docker compose build --no-cache
docker compose up -d
```

The Home Assistant configuration is stored on the host through the `/config` volume mapping. Back up important configuration data before making significant changes.

## Security notes

- Keep the Tailscale state directory private.
- Do not publish authentication credentials, private keys, or operational state files.
- Expose only the hardware devices required by your integrations.
- Review your Tailscale access controls and device permissions.
- Avoid exposing Home Assistant directly to untrusted networks unless you have an appropriate security configuration.

## Limitations

Successful image compilation does not guarantee that every Home Assistant integration or hardware device will work correctly on RISC-V.

Compatibility depends on the target board, operating system, Python packages, native libraries, and individual integration requirements.

This project is provided as a starting point. Users may need to adapt the configuration to their own environment.

## License

This project is licensed under the MIT License.

See [LICENSE](LICENSE) for details.
