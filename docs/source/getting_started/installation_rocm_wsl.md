# AMD ROCm on WSL

The ROCm setup is separate from the existing NVIDIA Docker configuration. It targets an AMD GPU exposed to Ubuntu through WSL and uses Docker Engine running inside that Ubuntu distribution.

## Prerequisites

- A GPU supported by AMD ROCm on WSL.
- ROCm installed in Ubuntu WSL so that `rocminfo` lists the GPU.
- Docker Engine installed and running inside Ubuntu WSL.
- `/dev/dxg`, `/usr/lib/wsl/lib/libdxcore.so`, and `/opt/rocm/lib/libhsa-runtime64.so.1` available in Ubuntu.

## Verify the host

```bash
rocminfo | grep -E 'Name:|Marketing Name:'
docker context show
```

The Docker context must point at the Docker Engine inside Ubuntu, not Docker Desktop's Windows engine.

## Build and verify PyTorch

From the repository in Ubuntu WSL:

```bash
docker compose -f docker-compose.rocm.yaml build
docker compose -f docker-compose.rocm.yaml run --rm demo \
  python -c "import torch; print(torch.__version__); print(torch.version.hip); print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
```

PyTorch ROCm deliberately uses the `torch.cuda` API names, so `torch.cuda.is_available()` should return `True` on the AMD GPU.

## Start Kimodo

WSL may stop the distribution after the last Windows-side WSL session exits, even
when containers were started in detached mode. Keep one hidden WSL session open:

```powershell
Start-Process -FilePath 'C:\Windows\System32\wsl.exe' `
  -ArgumentList @('-d', 'Ubuntu', '--exec', '/usr/bin/sleep', 'infinity') `
  -WindowStyle Hidden
```

Then start Kimodo from Ubuntu WSL:

```bash
sudo systemctl start docker
docker compose -f docker-compose.rocm.yaml up -d
```

Set `KIMODO_HF_CACHE` when the Hugging Face cache is somewhere other than `/mnt/d/kimodo-hf-cache` as seen from Ubuntu WSL.

The first start can take several minutes while the CPU text encoder loads and
the Viser client is built. Open <http://localhost:7860> after both containers
are running. Use `docker compose -f docker-compose.rocm.yaml ps` to check them.
