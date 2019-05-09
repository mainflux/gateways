## Mainflux Gateways

Mainflux gateways are HW gateways that can be used as a far IoT edge gateways
to connect local devices and send data to Mainflux IoT platform in the cloud or on-prem.


Mainflux develops and maintains SW and FW for 2 types of IoT edge gateways:
- MFX-1, powerfull ARM-based edge gateway
- MFX-mini, modest MIPS-based edge gateway with OpenWrt system

Mainflux gateways are remotely managed by the Edgeflux application running in the cloud.

In order to do this management, Edgeflux demands that each edgd gateway runs locally
a network agent, called Edged.

Edged is a small MQTT client, which connects to Mainflux
and subscribes to channels dedicated for controlplane.
It receives commands via Mainflux from Edgeflux application in the cloud,
then executes these commands locally on the gateway and sends the responses
back to the Edgeflux in the cloud.


### MFX-1
#### HW
MFX-1 is ARM gateway based on [SolidRun's CBi board](https://www.solid-run.com/nxp-family/hummingboard-cbi/).

It is based on [i.MX6 ARM Cortex-A9 SoM](https://www.solid-run.com/nxp-family/imx6-som/) and has
a lot of very powerful industrial [features](https://www.solid-run.com/nxp-family/hummingboard-cbi/#specifications).

#### SW
MFX-1 is powered by [EdgeX Foundry](https://www.edgexfoundry.org/) system, specifically crafted for this board
by [Mainflux team](https://www.mainflux.com).

Mainflux is official Linux Foundation's EdgeX Foundry member and one of the biggest contributors.
SW components running on the MFX-1 gateway are curated, tested and maintained by Mainflux alongside with
top-notch tooling and documentation needed for edge application development.

### MFX-mini
MFX-mini is MIPS gateway based on [8Devices' Lima Board](https://www.8devices.com/products/lima).
Lima is a [QCA4531](https://www.qualcomm.com/products/qca4531) chipset based module with a 650 MHz CPU and 802.11n 2x2 (MIMO) radio.

MFX-mini is simple, low-cost, low-power edge node, powered by [OpenWrt](https://openwrt.org/) Linux - a powerful SDK for application development.

MFX-mini runs `mEdge` (mini-edge) Mainflux firmwre - a minimalistic Edged agent connected
to Mainflux cloud and Mosquitto MQTT broker locally - to enable simple,
highly secure embedded edge IoT platform.

## Documentation
Documentation is located in the [./docs](docs) directory.

## Community
- [Google group](https://groups.google.com/forum/#!forum/mainflux)
- [Gitter](https://gitter.im/mainflux/mainflux)
- [Twitter](https://twitter.com/mainflux)

## License
[Apache-2.0](LICENSE)

