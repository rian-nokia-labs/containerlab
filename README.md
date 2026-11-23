# containerlab

Get a Nokia network lab running in a few minutes.

[containerlab](https://containerlab.dev/) turns one YAML file into a running lab of real network OS containers. This repo is the short path: install the tool, open the lab in the VS Code extension, log in, and tear it down.

## Nokia routers

Two guides sit next to this quickstart.

| Guide | What it answers |
| --- | --- |
| [SR Linux and SR OS](nokia-routers.md) | Which router to put in a lab, what it costs in CPU and RAM, how to write it in the YAML, and which types need a license. |
| [Router performance](router-performance.md) | Hardware throughput, silicon, and faceplate for each platform, and the containerlab `type` that boots it. |

The labs in this repo are SR Linux. The default type is a 7220 IXR-D2L.

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
3. Click the Containerlab icon in the activity bar. `first-lab.clab.yml`, `clos3.clab.yml`, and `clos5.clab.yml` are listed there.
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

The first deploy pulls `ghcr.io/nokia/srlinux:26.7.2`. That image is public. The pull is a few gigabytes, and it is the slow part. Later deploys reuse the local image.

The lab file is two SR Linux nodes and one link:

```yaml
name: first-lab

topology:
  nodes:
    srl1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:26.7.2
    srl2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:26.7.2
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

Destroy removes the containers and the lab network. The SR Linux image stays on the host. Use the same `-t` file you deployed.

## Clos labs

Two larger SR Linux fabrics are in this repo. Deploy one lab at a time. Each node wants about 2 GB of RAM, and the labs share the host with anything else you already have running.

`clos3.clab.yml` is a 3-stage Clos: two spines and four leaves. Every leaf connects to both spines, so the path from one leaf to another is leaf, spine, leaf. Six nodes, about 12 GB of RAM.

```bash
sudo containerlab deploy -t clos3.clab.yml
```

`clos5.clab.yml` is a 5-stage Clos with two pods under two superspines. Each pod has two leaves and two spines. A leaf connects only to the spines in its own pod, and every spine connects to both superspines. The path from a leaf in one pod to a leaf in the other is leaf, spine, superspine, spine, leaf. Ten nodes, about 20 GB of RAM.

```bash
sudo containerlab deploy -t clos5.clab.yml
```

Each node has `group` set to `leaf`, `spine`, or `superspine`, so TopoViewer stacks the tiers. Fabric links use `ethernet-1/1` and up, and each link has an IPv4 /31. The first address in `ipv4` belongs to the first endpoint. Containerlab configures that address on subinterface 0. `configs/leaf.cli` and `configs/spine.cli` add those subinterfaces to the `default` network-instance, which is what makes the addresses reachable. Routing protocols are not configured.

```bash
sudo containerlab destroy -t clos3.clab.yml
sudo containerlab destroy -t clos5.clab.yml
```

## Good to know

- The default hardware type is a 7220 IXR-D2L. That type runs without a license. Without a license the datapath is limited to 1000 packets per second, and the SR Linux process restarts once a week. 7250 IXR, 7730 SXR, and every SR OS node need a license. See [SR Linux and SR OS](nokia-routers.md) and [Router performance](router-performance.md).
- Save config you want to keep with `tools system configuration save` inside SR Linux, or with `sudo containerlab save -t first-lab.clab.yml`.
- SR Linux kind details, credentials, and management APIs: [Nokia SR Linux in containerlab](https://containerlab.dev/manual/kinds/srl/).
- The upstream quickstart and command reference: [containerlab quickstart](https://containerlab.dev/quickstart/).

## Arista, Cisco, and Juniper images

SR Linux is the image you can pull with no account. Arista, Cisco, and Juniper images come from the vendor. Download the image, import or build it locally, then use that local name in the topology. The full catalog is the [containerlab kinds list](https://containerlab.dev/manual/kinds/).

### Arista

[cEOS](https://containerlab.dev/manual/kinds/ceos/) is the container. Create an account on arista.com, download the `cEOS64` tar for the release you want, and import it with `docker import`. The kind is `arista_ceos`, and the interfaces in a topology are `eth1`, `eth2`, and so on.

[vEOS](https://containerlab.dev/manual/kinds/vr-veos/) is the VM form. Build that image with [vrnetlab](https://containerlab.dev/manual/vrnetlab/) from the vEOS disk image.

### Cisco

[XRd](https://containerlab.dev/manual/kinds/xrd/) is the IOS XR control-plane container. Download it with an active Cisco service account, then load the image into Docker. The kind is `cisco_xrd`. Containerlab does not run the XRd vRouter form, which needs PCI interfaces. Cisco's own notes on where to get the image are in the [XRd images tutorial](https://xrdocs.io/virtual-routing/tutorials/2022-08-22-xrd-images-where-can-one-get-them/).

[IOL](https://containerlab.dev/manual/kinds/cisco_iol/) is IOS and IOS XE packaged as a Linux binary. Take the `iol` or `ioll2` image from the Cisco CML reference-platform ISO and build a container with vrnetlab. The kind is `cisco_iol`. It is much lighter than the VM images.

The other Cisco products are VMs. Build [XRv](https://containerlab.dev/manual/kinds/vr-xrv/), [XRv9k](https://containerlab.dev/manual/kinds/vr-xrv9k/), [CSR1000v](https://containerlab.dev/manual/kinds/vr-csr/), [Nexus 9000v](https://containerlab.dev/manual/kinds/vr-n9kv/), [Catalyst 9000v](https://containerlab.dev/manual/kinds/vr-cat9kv/), and the rest from the vendor disk image with [vrnetlab](https://containerlab.dev/manual/vrnetlab/).

### Juniper

[cRPD](https://containerlab.dev/manual/kinds/crpd/) is the routing-daemon container. Download the cRPD Docker tarball from Juniper and load it with `docker load`. The kind is `juniper_crpd`. A license file is what turns on the licensed features.

[cSRX](https://containerlab.dev/manual/kinds/csrx/) is the container firewall. Load that image the same way and use the kind `juniper_csrx`.

[vMX](https://containerlab.dev/manual/kinds/vr-vmx/), [vQFX](https://containerlab.dev/manual/kinds/vr-vqfx/), [vSRX](https://containerlab.dev/manual/kinds/vr-vsrx/), [vJunos-router](https://containerlab.dev/manual/kinds/vr-vjunosrouter/), [vJunos-switch](https://containerlab.dev/manual/kinds/vr-vjunosswitch/), and [vJunosEvolved](https://containerlab.dev/manual/kinds/vr-vjunosevolved/) are VMs. Build each one with [vrnetlab](https://containerlab.dev/manual/vrnetlab/) from the Juniper disk image.
