# atl-cjdns-pi

<h1><a href="https://www.atlmesh.com">www.AtlMesh.com</a></h1>

The following instructions will help you set up an encrypted mesh network on Raspberry Pi's. It takes about 15 minutes to set up a node with the Pi 3. Obviously, to have a mesh you will need more than one node. 

Many models of Orange Pi hardware running [Armbian](https://www.armbian.com/) are also supported. The same installation steps can be followed, except you would flash the SD card with Armbian instead of Raspbian. See [Hardware Table](#hardware-table) for the full list of supported hardware.

## Set Up

1. Make sure you have the following items:

    * Raspberry Pi Zero, 1, 2, 3 (Pi 3 recommended), or for advanced users other [compatible hardware](#hardware-table)
    * An SD card that works with the Pi
    * **Optional:** A USB WiFi adapter with support, such as the [TP-LINK TL-WN722N]

1. Flash the SD card with [Raspbian Stretch Lite](https://www.raspberrypi.org/downloads/raspbian/).

1. Create an empty file named **ssh** to enable SSH when the Pi boots:

    ```
    $ touch /path/to/sd/boot/ssh
    ```

1. Plug the SD card and USB WiFi adapter into the Pi.

1. Plug the Pi into your router, so it has connectivity to the Internet. SSH into the Pi with `ssh pi@raspberrypi.local` and password **raspberry**.

    **Optional:** There are other ways to connect, such as connecting the Pi to your computer and sharing Internet to it. Or if you have multiple Pi's connected to your router, find its IP with `nmap -sn 192.168.X.0/24` (where 192.168.X is your subnet) and SSH to the local IP assigned to the Pi you want to address `ssh pi@192.168.X.Y`.

1. In your SSH session, run `passwd` and change your login password. It is very important to choose a strong password so others cannot remotely access your Pi.

1. Run the following, then let the installation complete. After about 5 minutes the Pi will reboot:

    ```
    $ sudo wget https://raw.githubusercontent.com/AtlMesh/atl-cjdns-pi/master/setup && sudo chmod +x setup && sudo ./setup
    ```
    For help with installation or other inquires, you can find us on our chat room at <a href="https://www.atlmesh.com/chat">www.AtlMesh.com/chat</a>
    
    The installation script can also install many optional features such as distributed applications and network analysis tools that are useful but non-essential to run a node.


## Optional Features

| Feature Flag                    | HTTP Service Port                              | Description |
| :------------------------------ | :--------------------------------------------- | :---------- |
| `WITH_MESH_POINT`               | None                                           | Set to `true` if you have a suitable USB WiFi adapter and want to configure it as a 802.11s Mesh Point interface. |
| `WITH_WIFI_AP`                  | None                                           | Set to `true` if you have a Raspberry Pi 3 and want to configure the on-board WiFi as an Access Point. The default configuration routes all traffic to the Ethernet port `eth0`. |
| `WITH_IPFS`                     | **80**: HTTP-to-IPFS gateway at `/ipfs/HASH` | Set to `true` if you want to install [IPFS](https://ipfs.io). |
| `WITH_PROMETHEUS_NODE_EXPORTER` | **9100**: Node Exporter UI                     | Set to `true` if you want to install [Prometheus Node Exporter](https://github.com/prometheus/node_exporter) to report network metrics. |
| `WITH_PROMETHEUS_SERVER`        | **9090**: Prometheus Server UI                 | Set to `true` if you want to install [Prometheus Server](https://github.com/prometheus/prometheus) to collect network metrics. *Requires Prometheus Node Exporter.* |
| `WITH_GRAFANA`                  | **3000**: Grafana UI (login: admin/admin)      | Set to `true` if you want to install [Grafana](https://grafana.com) to display network metrics. *Requires Prometheus Server.* |
| `WITH_H_DNS`                    | None                                           | Set to `true` if you want to use Hyperboria-compatible DNS servers: `fc4d:c8e5:9efe:9ac2:8e72:fcf7:6ce8:39dc` and `fc6e:691e:dfaa:b992:a10a:7b49:5a1a:5e09` |
| `WITH_H_NTP`                    | None                                           | Set to `true` if you want to use a Hyperboria-compatible NTP server: `fc4d:c8e5:9efe:9ac2:8e72:fcf7:6ce8:39dc` |
| `WITH_FAKE_HWCLOCK`             | None                                           | Set to `true` if you want to force hwclock to store its time every 5 minutes. |
| `WITH_EXTRA_TOOLS`              | None                                           | Set to `true` if you want to install non-essential tools useful for network analysis: vim socat oping bmon iperf3 |

If you are connected to the WiFi Access Point, all HTTP services are available via `http://10.0.0.1:PORT` as well as the cjdns IPv6. To connect with the cjdns address, first note your node's fc00::/8 address from `status`, then navigate to `http://[fcaa:bbbb:cccc:dddd:eeee:0000:1111:2222]:PORT` from your browser.

## Check Status

1. Give the Pi about 15 seconds to reboot and SSH back into it. You should find the status of your mesh node automatically printed. You can also print this anytime by running `status`.

1. Verify that **cjdns Service** is active, and **Mesh Interface** (if applicable). The **NODE** section should display a single IPv6 address, that's the identity of your Pi in the cjdns mesh. The **PEERS** section should indicate a list of IPv6 addresses that are active peers to your node. This list will be empty, until you have another nearby node with the same set up.

## Network Benchmark

You can benchmark the network throughput with more than one node. Let's name our two Pi's **Hillary** and **Friend**.

1. SSH to Friend and note its IPv6.

1. Run `iperf3 -s` to start listening. Do not end the SSH session.

1. In another Terminal session, SSH to Hillary and run `iperf3 -c FRIEND_IPV6`. You should start seeing Hillary sending encrypted packets to her Friend. See [phillymesh/cjdns-optimizations](https://github.com/phillymesh/cjdns-optimizations) for expected throughput.

## Update & Uninstall

To uninstall the services, run `./atl-cjdns-pi/scripts/uninstall`.

If you are updating, run the same uninstall script, but keep all configuration files and data directories when prompted, remove the **atl-cjdns-pi** directory along with the **install** script, then repeat the last installation step.

## Experimental Support for Orange Pi

We are adding support for [Orange Pi](http://www.orangepi.org/) boards and have tested with the [Orange Pi Zero (Armbian nightly)](https://dl.armbian.com/orangepizero/nightly/), [Orange Pi One (Armbian nightly)](https://dl.armbian.com/orangepione/nightly/), and [Orange Pi Lite  (Armbian nightly)](https://dl.armbian.com/orangepilite/nightly/). Instead of flashing Raspbian, start with the Armbian nightly images linked above, then follow the same installation steps as the Raspberry Pi.

## Hardware Table

List of tested hardware:

| Hardware                  | Base OS         | [CJDNS Benchmark](https://github.com/phillymesh/cjdns-optimizations) (salsa20/poly1305, switching) | USB | Ethernet | Notes    |
| :-------------------------|:----------------|:---------------------------------------------------------------------------------------------------|:----|:---------|:---------|
| Raspberry Pi 3            | [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)         | 350k, 100k | 2       | 10/100 |                                       |
| Raspberry Pi 2            | [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)         | 150k,  50k | 2       | 10/100 |                                       |
| Raspberry Pi 1 A+         | [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)         |  35k,   -  | 1       | None   |                                       |
| Raspberry Pi 1 B+         | [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)         |  35k,   -  | 2       | 10/100 |                                       |
| Raspberry Pi Zero         | [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)         |  68k,  30k | 1*      | None   | *Need OTG Cable                       |
| Orange Pi Lite            | [Armbian Nightly](https://dl.armbian.com/orangepilite/nightly/)          | 198k,  76k | 2       | None   |                                       |
| Orange Pi One             | [Armbian Nightly](https://dl.armbian.com/orangepione/nightly/)           | 198k,  76k | 1       | 10/100 |                                       |
| Orange Pi Zero            | [Armbian Nightly](https://dl.armbian.com/orangepizero/nightly/)          | 148k,  56k | 1 (+2*) | 10/100 | *Additional USB available via headers |
| Orange Pi Zero Plus 2 H5  | [Armbian Nightly](https://dl.armbian.com/orangepizeroplus2-h5/nightly/)  | 142k,  92K | 0 (+2*) | None   | *USB available via headers            |

## Development

To add a new module, use **scripts/ipfs/** as an example to:

* Create a `WITH_NEW_MODULE` tag
* Create **scripts/new-module/install** and **scripts/new-module/uninstall**
* Make corresponding references in the main **install**, **install2**, **status**, **uninstall** files

## Notes

* Your computer can be a node too! It will mesh with the Pi's over your router. See the [cjdns repository](https://github.com/cjdelisle/cjdns) on how to set this up.
