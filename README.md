# containerlab

Get a Nokia network lab running in a few minutes.

[containerlab](https://containerlab.dev/) turns one YAML file into a running lab of real network OS containers. This repo is the short path: install the tool, deploy two Nokia SR Linux nodes, log in, and tear the lab down.

## What you need

- A Linux host. The installer below is tested on Ubuntu 22.04 and 24.04, Debian 11 and 12, and RHEL 9 and similar. On Windows, use WSL2. On a Mac, use a Linux VM.
- `sudo` on that host.
- About 2 GB of RAM per SR Linux node. Two nodes are comfortable on an 8 GB host.
- A CPU with SSSE3. Physical machines have it. On a VM, set the CPU type to `host`. Containerlab refuses to start SR Linux without it.

## Install

This installs Docker, containerlab, and the GitHub CLI.

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
containerlab version
```

Other install options are in the [containerlab install guide](https://containerlab.dev/install/).

## Run the first lab

```bash
git clone https://github.com/rian-nokia-labs/containerlab.git
cd containerlab
sudo containerlab deploy -t first-lab.clab.yml
```

The first deploy pulls `ghcr.io/nokia/srlinux:24.10`. That image is public. The pull is a few gigabytes, and it is the slow part. Later deploys reuse the local image.

The lab file is two SR Linux nodes and one link:

```yaml
name: first-lab

topology:
  nodes:
    srl1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:24.10
    srl2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:24.10
  links:
    - endpoints: ["srl1:ethernet-1/1", "srl2:ethernet-1/1"]
```

Containerlab names the containers `clab-first-lab-srl1` and `clab-first-lab-srl2`, wires `ethernet-1/1` between them, and brings those interfaces up.

## Log in

The default login is `admin` / `NokiaSrl1!`.

```bash
ssh admin@clab-first-lab-srl1
```

From the host, without SSH:

```bash
docker exec -it clab-first-lab-srl1 sr_cli
```

List the nodes, their state, and their management addresses:

```bash
sudo containerlab inspect -t first-lab.clab.yml
```

## Tear it down

```bash
sudo containerlab destroy -t first-lab.clab.yml
```

Destroy removes the containers and the lab network. The SR Linux image stays on the host.

## Good to know

- The default hardware type is a 7220 IXR-D2L. That type runs without a license. Without a license the datapath is limited to 1000 packets per second, and the SR Linux process restarts once a week.
- Save config you want to keep with `tools system configuration save` inside SR Linux, or with `sudo containerlab save -t first-lab.clab.yml`.
- SR Linux kind details, credentials, and management APIs: [Nokia SR Linux in containerlab](https://containerlab.dev/manual/kinds/srl/).
- The upstream quickstart and command reference: [containerlab quickstart](https://containerlab.dev/quickstart/).
