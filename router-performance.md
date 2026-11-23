# Router performance

Hardware throughput for the Nokia routers a containerlab node can stand in for. The numbers are the system throughput printed on the Nokia datasheets for each platform.

A containerlab node does not forward at these rates. The lab datapath, the `type` values, and which nodes need a license are in [SR Linux and SR OS](nokia-routers.md). For a license, email [rian.pein@nokia.com](mailto:rian.pein@nokia.com).

## At a glance

| Platform | Hardware throughput | OS |
| --- | --- | --- |
| 7215 IXS-A1 and 7220 IXR-D1 | 88 Gb/s FD | SR Linux |
| 7220 IXR-D2L, the default lab type | 2.0 Tb/s FD | SR Linux |
| 7220 IXR-H6-64, H6-64 LC, H6-128 | 102.4 Tb/s FD | SR Linux |
| 7250 IXR-6e / 10e / 18e | Up to 115.2 / 230.4 / 460.8 Tb/s FD with latest-generation cards | SR Linux |
| 7730 SXR-1-32D and SXR-1d-32D | 4.4 Tb/s FD | SR Linux |
| 7730 SXR-1x-44S | 5.6 Tb/s FD | SR Linux |
| 7750 SR-14s | 216 Tb/s FD | SR OS |

FD is full duplex. On an SR OS 7250, the datasheet measures that rate on an IMIX mix, and the tables say so. On a 7750, intelligent aggregation (IA) is a second, higher figure when the sheet prints one. A chassis line that says "with latest-generation cards" is the equipped maximum.

`type` is the name containerlab boots. The labs in this repo use `ixr-d2l`.

## SR Linux

7220 IXR types boot with no license file. The unlicensed lab datapath is capped at 1000 packets per second. 7250 IXR and 7730 SXR types need a license or the node does not deploy.

### 7220 IXR-D

D1, D2L, and D3L are Broadcom Trident 3. D4 and D5 are Broadcom Trident 4.

| Model | `type` | Throughput | Ports |
| --- | --- | --- | --- |
| 7220 IXR-D1 | `ixr-d1` | 88 Gb/s FD | 48×1GE RJ45, 4×10G SFP+ |
| 7220 IXR-D2L | `ixr-d2l` | 2.0 Tb/s FD | 48×25G SFP28, 8×100G QSFP28, 2×10G SFP+ |
| 7220 IXR-D3L | `ixr-d3l` | 3.2 Tb/s FD | 32×100G QSFP28, 2×10G SFP+ |
| 7220 IXR-D4 | `ixr-d4` | 6.0 Tb/s FD | 28×100G QSFP28, 8×400G QSFP-DD |
| 7220 IXR-D5 | `ixr-d5` | 12.8 Tb/s FD | 32×400G QSFP-DD, 2×10G SFP+ |

Containerlab also accepts `ixr-d2` and `ixr-d3`. This datasheet set has D1, D2L, D3L, D4, and D5.

### 7220 IXR-H

| Model | `type` | Throughput | Silicon | Ports |
| --- | --- | --- | --- | --- |
| 7220 IXR-H2 | `ixr-h2` | 12.8 Tb/s FD | Broadcom Tomahawk 3 | 128×100G QSFP28, 4RU |
| 7220 IXR-H3 | `ixr-h3` | 12.8 Tb/s FD | Broadcom Tomahawk 3 | 32×400G QSFP-DD, 2×10G SFP+ |
| 7220 IXR-H4-32D | `ixr-h4-32d` | 12.8 Tb/s FD | Broadcom Tomahawk 4 | 32×400G QSFP-DD, 1×10G SFP+ |
| 7220 IXR-H4 | `ixr-h4` | 25.6 Tb/s FD | Broadcom Tomahawk 4 | 64×400G QSFP-DD, 2×10G SFP+, 2RU |
| 7220 IXR-H5-32D | `ixr-h5-32d` | 25.6 Tb/s FD | Broadcom Tomahawk 5 | 32×800G QSFP-DD, 2×10G SFP+ |
| 7220 IXR-H5-64D | `ixr-h5-64d` | 51.2 Tb/s FD | Broadcom Tomahawk 5 | 64×800G QSFP-DD, 2×10G SFP+ |
| 7220 IXR-H5-64O | `ixr-h5-64o` | 51.2 Tb/s FD | Broadcom Tomahawk 5 | 64×800G OSFP112, 2×10G SFP+ |
| 7220 IXR-H6-64 | `ixr-h6` | 102.4 Tb/s FD | not printed | 64×1.6T OSFP224, 2×25G SFP28, 3RU |
| 7220 IXR-H6-64 LC | — | 102.4 Tb/s FD | not printed | 64×1.6T OSFP224, 1×100G QSFP28, liquid-cooled 2OU |
| 7220 IXR-H6-128 | — | 102.4 Tb/s FD | not printed | 128×800G OSFP112, 1×100G QSFP28, 4RU |

Containerlab publishes one H6 type, `ixr-h6`, listed on the H6-64 row. H6-64 LC and H6-128 are the other faceplates at the same 102.4 Tb/s, on the April 2026 preliminary sheet. H5-32D, H5-64D, and H5-64O support LPO. H6-64 LC and H6-128 support LPO and LRO.

### 7215 IXS-A1

| Model | `type` | Throughput | Ports |
| --- | --- | --- | --- |
| 7215 IXS-A1 | `ixs-a1` | 88 Gb/s FD | 48×1GE RJ45, 4×1/10G SFP+, 1RU |

SR Linux only. Out-of-band management interconnect. The datasheet does not name the silicon.

### 7250 IXR on SR Linux

These types need a license.

| Model | `type` | Throughput | Silicon | Ports |
| --- | --- | --- | --- | --- |
| 7250 IXR-X1b | `ixr-x1b` | 7.2 Tb/s FD | Broadcom Jericho 2C+ | 12×400G QSFP-DD, 24×100G QSFP28, 1RU |
| 7250 IXR-X3b | `ixr-x3b` | 14.4 Tb/s FD | Broadcom Jericho 2C+ | 36×400G QSFP-DD, 1RU |
| 7250 IXR-X4 | `ixr-x4` | 25.6 Tb/s FD | not printed | 32×800G QSFP-DD, or 32×800G OSFP, 1RU |
| 7250 IXR-6e | `ixr-6e` | Up to 115.2 Tb/s FD with latest-generation cards | J2C+ / J3 | 4 IMM slots, 10RU |
| 7250 IXR-10e | `ixr-10e` | Up to 230.4 Tb/s FD with latest-generation cards | J2C+ / J3 | 8 IMM slots, 16RU |
| 7250 IXR-18e | `ixr-18e` | Up to 460.8 Tb/s FD with latest-generation cards | J2C+ / J3 | 16 IMM slots, 35RU |

IXR-X4 is on the Release 26 preliminary SR Linux sheet. IXR-6, IXR-10, and IXR-s in the next section are the SR OS platforms.

### 7730 SXR

SR Linux, Nokia FPcx. These types need a license.

| Model | `type` | Throughput | Ports |
| --- | --- | --- | --- |
| 7730 SXR-1-32D | `sxr-1-32d` | 4.4 Tb/s FD | 4×400G QSFP-DD, 28×100G QSFP28, 1RU |
| 7730 SXR-1d-32D | `sxr-1d-32d` | 4.4 Tb/s FD | 4×400G QSFP-DD, 28×100G QSFP28, 2RU |
| 7730 SXR-1x-44S | `sxr-1x-44s` | 5.6 Tb/s FD | 4×400G QSFP-DD, 40×100G SFP-DD, 1RU |

## SR OS

Every SR OS node needs a license. The kind is `nokia_srsim` for the container, or `nokia_sros` for the VM. A fixed system is one container. A chassis is one container per CPM and one container per IOM. Writing the node is in [SR Linux and SR OS](nokia-routers.md).

A dash in the type column means this page does not assign a containerlab type. The integrated types with generated defaults are on the [SR-SIM kind page](https://containerlab.dev/manual/kinds/sros/). Containerlab also accepts `ixr-e2n-s`. This datasheet set has no separate IXR-e2n-s throughput.

Rates on the 7250 SR OS tables are full-duplex IMIX.

### 7250 IXR-e, IXR-e2, and IXR-e3

| Model | `type` | Throughput | Silicon | Ports |
| --- | --- | --- | --- | --- |
| 7250 IXR-e 14SFP+ 4RJ45 | — | 120 Gb/s FD, IMIX | Broadcom Qumran | 14×10/1G SFP+, 4×RJ45 |
| 7250 IXR-e 2QSFP28 8SFP28 24SFP+ | — | 300 Gb/s FD, IMIX | Broadcom Qumran | 2×100G QSFP28, 8×25/10/1G, 24×10/1G |
| 7250 IXR-ec | `ixr-ec` | 64 Gb/s FD, IMIX | not printed | 6×10/1G SFP+, 20×1G SFP, 4×RJ45 |
| 7250 IXR-e2 | `ixr-e2` | 800 Gb/s FD, IMIX | Broadcom Qumran | 2×400G QSFP-DD, 2×100G QSFP28, 24×25/10/1G |
| 7250 IXR-e2c | `ixr-e2c` | 100 Gb/s FD, IMIX | not printed | 2×100G QSFP28, 12×25/10/1G |
| 7250 IXR-e2n | `ixr-e2n` | 44 Gb/s FD, IMIX | not printed | 4×10/1G SFP+, 4×1G SFP |
| 7250 IXR-e2sc | `ixr-e2sc` | 100 Gb/s FD, IMIX | not printed | 4×25/10/1G SFP28, 16×10/1G SFP+ |
| 7250 IXR-e3c | `ixr-e3c` | 400 Gb/s FD, IMIX | Broadcom Qumran | 4×100G QSFP28, 8×50G, 16×25G |
| 7250 IXR-e3x | `ixr-e3x` | 2.4 Tb/s FD, IMIX | Broadcom Qumran | 6×400G QSFP-DD, 16×100G SFP112, 15×50G |

### 7250 IXR-R

The rate holds with a single CPM or a redundant active/standby pair. Silicon is not named on the datasheet.

| Model | `type` | Throughput | Chassis |
| --- | --- | --- | --- |
| 7250 IXR-R4 | — | 300 Gb/s FD, IMIX | 2RU, 4 MDA slots, plus fixed IOM ports |
| 7250 IXR-R6 | `ixr-r6` | 800 Gb/s FD, IMIX | 3RU, 6 slots |
| 7250 IXR-R6d | — | 2.4 Tb/s FD, IMIX | 4RU, 6 compact slots |
| 7250 IXR-R6dl | — | 2.4 Tb/s FD, IMIX | 7RU, 6 slots |

### 7250 IXR-X on SR OS

| Model | `type` | Throughput | Silicon | Ports |
| --- | --- | --- | --- | --- |
| 7250 IXR-Xs | — | 4.8 Tb/s FD, IMIX | Broadcom Jericho 2 | 6×400G QSFP-DD, 48×50/25/10G SFP, 1RU |
| 7250 IXR-X1 | — | 4.8 Tb/s FD, IMIX | Broadcom Jericho 2 | 4×400G QSFP-DD, 32×100/40G QSFP28, 1RU |
| 7250 IXR-X3 | — | 14.4 Tb/s FD, IMIX | Broadcom Jericho 2C+ | 36×400G QSFP-DD, 1RU |

The SR Linux faceplates are IXR-X1b, IXR-X3b, and IXR-X4, in the section above.

### 7250 IXR-6, IXR-10, and IXR-s

SR OS. Silicon is not named on the datasheet. These are separate platforms from IXR-6e and IXR-10e.

| Model | Throughput | Chassis |
| --- | --- | --- |
| 7250 IXR-6 | 14.4 Tb/s FD, IMIX | 7RU, 4 card slots, 3.6 Tb/s FD per slot |
| 7250 IXR-10 | 28.8 Tb/s FD, IMIX | 13RU, 8 card slots, 3.6 Tb/s FD per slot |
| 7250 IXR-s | 800 Gb/s FD, IMIX | 1RU fixed, 6×100/40GE, 48×10/1GE |

### 7750 SR

SR OS, Nokia FP silicon. Where the sheet prints intelligent aggregation, that figure is in the throughput cell next to the base full-duplex rate.

| Model | `type` | Throughput | Silicon | Faceplate |
| --- | --- | --- | --- | --- |
| 7750 SR-1 | `sr-1` | 1.5 Tb/s FD (up to 4.0 Tb/s FD with intelligent aggregation) | FP4 / FP5 | Modular, single slot |
| 7750 SR-1-24D | — | 2.4-2.8 Tb/s FD (up to 9.6T IA) | FP4 / FP5 | 24×400G/800G QSFP-DD |
| 7750 SR-1-46S | — | 2.4-2.8 Tb/s FD (up to 9.6T IA) | FP4 / FP5 | 40×10/25G SFP-DD, 6×400G QSFP-DD |
| 7750 SR-1-48D | — | 2.8 Tb/s FD (up to 19.2 Tb/s with IA) | FP4 / FP5 | 48×400G QSFP-DD |
| 7750 SR-1-92S | — | 2.8 Tb/s FD (up to 12.8 Tb/s with IA) | FP4 / FP5 | 80×200G SFP-DD, 12×400G QSFP-DD |
| 7750 SR-1x-48D | — | 4.8-6.0 Tb/s FD (up to 19.2 Tb/s with IA) | FP5 | 48×400G/800G QSFP-DD, 2RU |
| 7750 SR-1x-92S | — | 6.0 Tb/s FD (up to 12.8 Tb/s FD with intelligent aggregation) | FP5 | 80×200G SFP-DD, 12×800G QSFP-DD, 2RU |
| 7750 SR-a4 | — | 100-200 Gb/s FD | FP3 | 5RU, MDA faceplate, 300 mm |
| 7750 SR-a8 | — | 200 Gb/s FD | FP3 | 7RU, MDA faceplate, 300 mm |
| 7750 SR-1e | — | Up to 200 Gb/s FD | FP4 / FP5 | 6RU, 4 MDA-e |
| 7750 SR-2e | — | Up to 400 Gb/s FD | FP4 / FP5 | 10RU, 8 MDA-e |
| 7750 SR-3e | — | Up to 600 Gb/s FD | FP4 / FP5 | 13RU, 12 MDA-e |
| 7750 SR-1s | `sr-1s` | Up to 4.8 Tb/s FD | FP4 | 3RU fixed |
| 7750 SR-1se | — | 19.2 Tb/s FD | FP5 | 3RU fixed |
| 7750 SR-2s | — | 9.6 Tb/s FD | FP4 | 5RU, 2 slots |
| 7750 SR-2se | — | 36 Tb/s FD | FP5 | 5RU, 2 slots, 18 Tb/s FD per slot |
| 7750 SR-7 | `sr-7` | Up to 4-6 Tb/s FD (with IA) | FP4 | 5 slots, 800 Gb/s FD per slot (up to 1.2 Tb/s with IA) |
| 7750 SR-12 | — | Up to 8-12 Tb/s FD (with IA) | FP4 | 10 slots, 800 Gb/s FD per slot (up to 1.2 Tb/s with IA) |
| 7750 SR-12e | — | Up to 13.5 Tb/s FD (with IA) | FP4 | 9 slots, up to 1.5 Tb/s FD per slot |
| 7750 SR-7s | — | 108 Tb/s FD | FP4 / FP5 | 6 slots |
| 7750 SR-14s | `sr-14s` | 216 Tb/s FD | FP4 / FP5 | 12 slots |

`sr-1` and `sr-1s` are integrated. `sr-7` and `sr-14s` are chassis types and boot distributed. The SR-a sheet names FP3 silicon.

## Source

Figures are the system throughput, silicon, and faceplate on the Nokia router datasheets for 7215 IXS, 7220 IXR-D, 7220 IXR-H (including the April 2026 preliminary H6 sheet), 7250 IXR, 7730 SXR, and 7750 SR. Where a sheet does not name the silicon, the table says so. H6-64 is the one H6 model on the July 2026 sheet, and that sheet does not name its silicon either.
