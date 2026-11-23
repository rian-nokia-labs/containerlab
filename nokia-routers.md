# Nokia SR Linux and SR OS

Which Nokia router to put in a lab, what it costs in CPU and RAM, and how to write it in a containerlab file.

Containerlab has three Nokia router kinds.

| | SR Linux | SR OS SR-SIM | SR OS vSIM |
| --- | --- | --- | --- |
| Kind | `nokia_srlinux` | `nokia_srsim` | `nokia_sros` |
| Packaging | One container per node | One container per node, or one container per card on a chassis | QEMU virtual machine inside a container |
| Image | Public. `ghcr.io/nokia/srlinux:26.7.2` | Download from the Nokia Support Portal, then `docker load` | Build with [vrnetlab](https://containerlab.dev/manual/vrnetlab/) from an SR OS disk image |
| License | Optional for 7220 IXR. Required for 7250 IXR and 7730 SXR | Required. The lab does not deploy without one | Required. The lab does not deploy without one |
| Login | `admin` / `NokiaSrl1!` | `admin` / `NokiaSros1!` | `admin` / `admin` |

SR Linux is the one to start with. The image is public, a node is one container, and containerlab asks for 2 vCPU and 2 GB of RAM per node. The labs in this repo are SR Linux.

SR-SIM is the SR OS to use when the lab needs SR OS. It is the same simulator as vSIM, packaged as containers. A pizza-box such as an SR-1 is one container. A chassis is one container per CPM and one container per IOM, so the host schedules the cards separately. A published two-node lab of one SR Linux and one SR-SIM is budgeted at 2 vCPU and 5 GB.

vSIM is the older form. Containerlab still runs it, and the SR OS docs point new labs at SR-SIM. A vSIM node pays for a full virtual machine on top of the simulator, so it uses more CPU and RAM and takes longer to reach a prompt.

## Datapath

An unlicensed SR Linux node runs the 7220 IXR types only. The emulated datapath is capped at 1000 packets per second, and the `sr_linux` process restarts once a week. A license file removes both limits. It does not turn the container into the forwarding rate of the hardware. The datapath is still the XDP emulation, and the host CPU has to support SSSE3.

SR-SIM and vSIM have no unlicensed mode. Without a license file, containerlab refuses to deploy the node.

## How to write them

Set `kind`, `image`, and, when you care which chassis boots, `type`. Put a shared image and license on `topology.kinds` so each node does not repeat them.

### SR Linux, no license

`ixr-d2l` is the default. It is a 7220 IXR-D2L and it boots with no license file. Other 7220 types (`ixr-d1`, `ixr-d2`, `ixr-d3`, `ixr-h2`, `ixr-h3`, and the rest of that family) are the same: no license, same 1000 packet-per-second cap.

```yaml
name: srl

topology:
  defaults:
    kind: nokia_srlinux
    image: ghcr.io/nokia/srlinux:26.7.2
  nodes:
    spine1:
      type: ixr-d3l
    leaf1:
      type: ixr-d2l
  links:
    - endpoints: ["leaf1:ethernet-1/1", "spine1:ethernet-1/1"]
```

Interface names are `ethernet-1/1`, `ethernet-1/2`, and so on. `e1-1` is the same port.

### SR Linux with a license

7250 IXR types (`ixr-6e`, `ixr-10e`, `ixr-18e`, `ixr-x1b`, `ixr-x3b`, `ixr-x4`) and 7730 SXR types (`sxr-1x-44s`, `sxr-1d-32d`, `sxr-1-32d`) do not boot without a license. The same file, placed on a 7220 node, is what removes the 1000 packet-per-second cap and the weekly restart.

```yaml
topology:
  kinds:
    nokia_srlinux:
      image: ghcr.io/nokia/srlinux:26.7.2
      license: ./license-srl.txt
  nodes:
    spine1:
      kind: nokia_srlinux
      type: ixr-6e
    leaf1:
      kind: nokia_srlinux
      type: ixr-d2l
  links:
    - endpoints: ["leaf1:ethernet-1/1", "spine1:ethernet-1/1"]
```

`license` is a path on the lab host. Containerlab copies that file into the node directory.

### SR OS SR-SIM

The image is not on a public registry. Download the SR-SIM tar from the Nokia Support Portal and load it:

```bash
docker load -i srsim.tar.xz
```

The local name looks like `nokia_srsim:26.7.R1`. The tag is the version you loaded. An SR-1 is the default type and is a single container. The first data port on that default card is `1/1/c1/1`.

```yaml
name: srl-sros

topology:
  nodes:
    srl1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:26.7.2
    sros1:
      kind: nokia_srsim
      image: nokia_srsim:26.7.R1
      type: sr-1
      license: ./license-sros.txt
  links:
    - endpoints: ["srl1:ethernet-1/1", "sros1:1/1/c1/1"]
```

A chassis does not fit in one container. List each CPM and IOM under `components`. This SR-7 is four containers: CPM A, CPM B, and two IOMs.

```yaml
topology:
  kinds:
    nokia_srsim:
      image: nokia_srsim:26.7.R1
      license: ./license-sros.txt
  nodes:
    sros1:
      kind: nokia_srsim
      type: sr-7
      components:
        - slot: A
        - slot: B
        - slot: 1
        - slot: 2
```

Connect a link to the line card that owns the port, using the SR OS name (`1/1/c1/1`) or the Linux form (`e1-1-c1-1`).

Integrated types containerlab knows include `sr-1`, `sr-1s`, `ixr-r6`, `ixr-e2`, `ixr-e2c`, `ixr-e2n`, `ixr-e2n-s`, `ixr-e2sc`, `ixr-e3c`, `ixr-e3x`, and `ixr-ec`. Anything chassis-based (`sr-7`, `sr-14s`, and the other SR-s chassis) uses the distributed form above.

### SR OS vSIM

Use this only when you already have a vrnetlab image. Do not put the license in the image build. Pass it in the topology, the same way as SR-SIM.

```yaml
topology:
  nodes:
    sros1:
      kind: nokia_sros
      image: vrnetlab/nokia_sros:25.10.R1
      type: sr-1s
      license: ./license-sros.txt
```

## Getting a license

Email [rian.pein@nokia.com](mailto:rian.pein@nokia.com) and say which product you need, SR Linux, SR-SIM, or both, plus the version and how many nodes. SR Linux 7220 labs do not need one. SR Linux 7250 and 7730, every SR-SIM node, and every vSIM node do.

Kind reference: [SR Linux](https://containerlab.dev/manual/kinds/srl/), [SR-SIM](https://containerlab.dev/manual/kinds/sros/), [vSIM](https://containerlab.dev/manual/kinds/vr-sros/).
