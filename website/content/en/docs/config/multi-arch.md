---
title: Intel-on-ARM and ARM-on-Intel
weight: 20
---

Lima supports two modes for running Intel-on-ARM and ARM-on-Intel:
- [Slow mode](#slow-mode)
- [Fast mode](#fast-mode)
- [Fast mode 2](#fast-mode-2)

## [Slow mode: Intel VM on ARM Host / ARM VM on Intel Host](#slow-mode)

Lima can run a VM with a foreign architecture, just by specifying `arch` in the YAML.

```yaml
arch: "x86_64"
# arch: "aarch64"

images:
  - location: "https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img"
    arch: "x86_64"
  - location: "https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-arm64.img"
    arch: "aarch64"

# Disable mounts and containerd, otherwise booting up may time out if the host is slow
mounts: []
containerd:
  system: false
  user: false
```

Running a VM with a foreign architecture is extremely slow.
Consider using [Fast mode](#fast-mode) or [Fast mode 2](#fast-mode-2) whenever possible.

## [Fast mode: Intel containers on ARM VM on ARM Host / ARM containers on Intel VM on Intel Host](#fast-mode)

This mode uses QEMU User Mode Emulation.
QEMU User Mode Emulation is significantly faster than QEMU System Mode Emulation, but it often sacrifices compatibility.

Set up:
```bash
lima sudo systemctl start containerd
lima sudo nerdctl run --privileged --rm tonistiigi/binfmt:qemu-v7.0.0-28@sha256:66e11bea77a5ea9d6f0fe79b57cd2b189b5d15b93a2bdb925be22949232e4e55 --install all
```

Run containers:
```console
$ lima nerdctl run --platform=amd64 --rm alpine uname -m
x86_64

$ lima nerdctl run --platform=arm64 --rm alpine uname -m
aarch64
```

Build and push container images:
```console
$ lima nerdctl build --platform=amd64,arm64 -t example.com/foo:latest .
$ lima nerdctl push --all-platforms example.com/foo:latest
```

See also https://github.com/containerd/nerdctl/blob/main/docs/multi-platform.md

## [Fast mode 2 (Rosetta): Intel containers on ARM VM on ARM Host](#fast-mode-2)

| ⚡ Requirement | Lima >= 0.14, macOS >= 13.0, ARM |
|-------------------|----------------------------------|

[Rosetta](https://developer.apple.com/documentation/virtualization/running_intel_binaries_in_linux_vms_with_rosetta) is known to be much faster than QEMU User Mode Emulation.
Rosetta is available for [VZ](../vmtype/#vz) instances on ARM hosts.

{{< tabpane text=true >}}
{{% tab header="CLI" %}}
```bash
limactl start --vm-type=vz --rosetta
```
{{% /tab %}}
{{% tab header="YAML" %}}
```yaml
vmType: "vz"
rosetta:
  # Enable Rosetta for Linux.
  # Hint: try `softwareupdate --install-rosetta` if Lima gets stuck at `Installing rosetta...`
  enabled: true
  # Register rosetta to /proc/sys/fs/binfmt_misc
  binfmt: true
```
{{% /tab %}}
{{< /tabpane >}}

### [Enable Rosetta AOT Caching with CDI spec](#rosetta-aot-caching)
| ⚡ Requirement | Lima >= 2.0, macOS >= 14.0, ARM |
|-------------------|----------------------------------|

Rosetta AOT Caching speeds up containers by saving translated binaries, so they don't need to be translated again.  
Learn more: [WWDC2023 video](https://developer.apple.com/videos/play/wwdc2023/10007/?time=721)

**How to use Rosetta AOT Caching:**

- **Run a container:**  
  Add `--device=lima-vm.io/rosetta=cached` to your `docker run` command:
  ```bash
  docker run --platform=linux/amd64 --device=lima-vm.io/rosetta=cached ...
  ```

- **Build an image:**  
  Add `# syntax=docker/dockerfile:1-labs` at the top of your Dockerfile to enable the `--device` option.  
  Use `--device=lima-vm.io/rosetta=cached` in your `RUN` command:
  ```Dockerfile
  # syntax=docker/dockerfile:1-labs
  FROM ...
  ...
  RUN --device=lima-vm.io/rosetta=cached <your amd64 command>
  ```

- **Check if caching works:**  
  Look for cache files in the VM:
  ```bash
  limactl shell {{.Name}} ls -la /var/cache/rosettad
  docker run --platform linux/amd64 --device=lima-vm.io/rosetta=cached ubuntu echo hello
  limactl shell {{.Name}} ls -la /var/cache/rosettad
  # You should see *.aotcache files here
  ```

- **Check if Docker recognizes the CDI device:**  
  Look for CDI info in the output of `docker info`:
  ```console
  docker info
  ...
  CDI spec directories:
    /etc/cdi
    /var/run/cdi
  Discovered Devices:
    cdi: lima-vm.io/rosetta=cached
  ```

- **Learn more about CDI:**  
  [CDI spec documentation](https://github.com/cncf-tags/container-device-interface/blob/main/SPEC.md)
