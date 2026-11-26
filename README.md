# unsloth-llama-docker

Drop-in replacements for the official `ghcr.io/ggml-org/llama.cpp` `llama-server` images, rebuilt automatically every 6 hours from [`unslothai/llama.cpp`](https://github.com/unslothai/llama.cpp) releases (upstream `b####` + Unsloth's `pr-set.json` patches).

## Available Images

Every image is tagged with both its rolling variant name and pinned release tag (`:<variant>-<release>`, e.g. `:server-cuda13-sm89-b12676-mix-c32dcc8`).

| Variant | Target Hardware | Pull String |
| :--- | :--- | :--- |
| **CPU** | x86_64 CPU | `ghcr.io/kityr/unsloth-llama-docker:server` |
| **CUDA 13 (General)** | All supported NVIDIA GPUs | `ghcr.io/kityr/unsloth-llama-docker:server-cuda13` |
| **CUDA 13 (`sm_89`)** | NVIDIA Ada Lovelace (RTX 40-series, RTX 2000/4000/6000 Ada, L4/L40) | `ghcr.io/kityr/unsloth-llama-docker:server-cuda13-sm89` |
| **ROCm** | AMD GPUs (HIP) | `ghcr.io/kityr/unsloth-llama-docker:server-rocm` |
| **Vulkan** | Cross-vendor GPUs (AMD, Intel, NVIDIA) | `ghcr.io/kityr/unsloth-llama-docker:server-vulkan` |

### Quick Pull

```bash
# CPU
podman pull ghcr.io/kityr/unsloth-llama-docker:server

# CUDA 13 (all architectures)
podman pull ghcr.io/kityr/unsloth-llama-docker:server-cuda13

# CUDA 13 (Ada Lovelace / sm_89 only — smaller image)
podman pull ghcr.io/kityr/unsloth-llama-docker:server-cuda13-sm89

# ROCm
podman pull ghcr.io/kityr/unsloth-llama-docker:server-rocm

# Vulkan
podman pull ghcr.io/kityr/unsloth-llama-docker:server-vulkan
```

## Podman Quadlet Example

Save as `/etc/containers/systemd/llama-server.container` (rootful) or `~/.config/containers/systemd/llama-server.container` (rootless):

```ini
# llama-server.container
[Unit]
Description=Unsloth llama-server via Podman Quadlet
After=network-online.target

[Container]
# Swap tag to :server, :server-cuda13, :server-cuda13-sm89, :server-rocm, or :server-vulkan
Image=ghcr.io/kityr/unsloth-llama-docker:server-cuda13
ContainerName=llama-server

# Expose port 8080 (use 127.0.0.1:8080:8080 to bind localhost only)
PublishPort=8080:8080

# Mount your GGUF directory into /models (use :z or :Z on SELinux systems)
Volume=/path/to/models:/models:z

# NVIDIA GPU access via CDI (requires nvidia-ctk cdi generate)
AddDevice=nvidia.com/gpu=all
# For AMD GPUs (ROCm / Vulkan), use instead:
# AddDevice=/dev/kfd
# AddDevice=/dev/dri

# Uncomment to let podman-auto-update pull new builds automatically
# AutoUpdate=registry

# Single-model mode (--host 0.0.0.0 is required inside the container for PublishPort):
Exec=-m /models/your-model.gguf -ngl 99 -c 32768 --host 0.0.0.0 --port 8080
# Or router mode with a presets file:
# Exec=--models-preset /models/models.ini --host 0.0.0.0 --port 8080

SecurityLabelDisable=true

[Service]
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now llama-server
journalctl -fu llama-server # watch the service logs as it pulls the image and starts up
```
