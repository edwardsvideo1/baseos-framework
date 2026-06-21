# BaseOS Provisioning Template - Flatcar Ignition
Flatcar relies on [ignition](https://www.flatcar.org/docs/latest/provisioning/ignition/) which processes from the initramfs.

After iPXE has given instructions to boot the flatcar kernel and passed in the kernel params from the `discovery_ipxe.cfg` file, the kernel attempt to boot and will cycle through the interfaces.  As long as the upstream router or switch is configured for SLAAC and sending Router Advertisements (RAs), Flatcar will eventually be able to accept its SLAAC IPv6 address and download the `discovery_ignition.json` (ignition) template.  Example below:
```
{
  "ignition": {
    "version": "3.4.0"
  },
  "systemd": {
    "units": [
      {
        "name": "mtu-lshw-lldp-discovery.service",
        "enabled": true,
        "contents": "[Unit]\nDescription=Hardware discovery inventory (first boot only)\nConditionFirstBoot=yes\n\n[Service]\nType=oneshot\nRemainAfterExit=yes\nExecStart=/usr/bin/bash -c 'sudo ip link set dev enp10s0f0np0 mtu 9100; sudo ip link set dev enp138s0f1np1 mtu 9100; sudo lshw -json > /tmp/lshw.json ; sudo sleep 60 ; sudo networkctl lldp > /tmp/lldp.sdv'\n\n[Install]\nWantedBy=multi-user.target\n"
      }
    ]
  }
}
```
This is a simple ignition template with a systemd that configures MTU, runs lshw and lldp tools (generating output).  NOTE: The MTU had to be set to 9100 in order to meet unique platform requirements.  When configured for 1500, the server failed to retrieve files much larger than this template.  Unfortunately, there's no way I've found to add MTU successfully in the kernel parameters passed in via iPXE, so I had to use ignition.
