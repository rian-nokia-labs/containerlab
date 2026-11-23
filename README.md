# containerlab

Get a Nokia network lab running in a few minutes.

[containerlab](https://containerlab.dev/) turns one YAML file into a running lab of real network OS containers. This repo is the short path: install the tool, open the lab in the VS Code extension, log in, and tear it down.

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

## Use the VS Code extension

The [Containerlab extension](https://marketplace.visualstudio.com/items?itemName=srl-labs.vscode-containerlab) is the best way to run this lab. It finds `*.clab.yml` files in the folder you have open, draws the topology, and deploys, destroys, and logs in from the editor.

1. In VS Code, open Extensions and search for `Containerlab`. The publisher is `srl-labs`. The extension id is `srl-labs.vscode-containerlab`.
2. Clone this repo and open the folder in VS Code.
3. Click the Containerlab icon in the activity bar. `first-lab.clab.yml` is listed there.
4. Right-click the lab and choose **Deploy**. When you are done, choose **Destroy**.

From an open topology file, the same actions are on the editor title bar and in the command palette:

| Keys | Action |
| --- | --- |
| Ctrl+Alt+D | Deploy |
| Ctrl+Alt+R | Redeploy |
| Ctrl+Alt+K | Destroy |
| Ctrl+Alt+G | Open the topology in TopoViewer |

On a Mac, use Cmd in place of Ctrl.

TopoViewer shows the lab as a diagram next to the YAML. A right-click on a node opens an SSH session or the SR Linux CLI in a VS Code terminal. A right-click on a link starts a packet capture in Wireshark or Edgeshark.

The extension runs containerlab on the machine where that VS Code window is running. On Linux or WSL, that is the machine in front of you. From Windows or a Mac, connect to the lab host with the [Remote SSH extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) and use Containerlab in the remote window.

Your user needs to be in the `docker` and `clab_admins` groups so the extension can run labs without `sudo`. The extension offers to add you to `clab_admins`. Log out and back in after the group change.

Extension guide: [Containerlab VS Code extension](https://containerlab.dev/manual/gui/vsc-extension/).

## Run the first lab

The extension deploys this lab from the right-click menu. The same lab from a shell:

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
