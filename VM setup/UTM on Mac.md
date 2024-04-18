# UTM on Mac

https://mac.getutm.app/

Also available in the mac app store (paid) or via a homebrew cask.

It can run virtualized (same architecture as host) or emulated (allows e.g. to run AMD64 on M1/M2 ARM macs).

## Installation

We need at least one virtual machine. If you want to test out agent nodes - then a machine for each of them.

On ARM mac - I found it was easiest to get debian working. On intel - either debian or ubuntu.

So grab an image - install it - use defaults as much as possible in UTM.

When it boots - install - for debian I chose XFCE desktop at tasksel - to keep the load as light as possible. Enable ssh too.

Pick meaningful hostnames (k3s, k3s-agent-1 etc) and pick some random test domain that is the same across all of them (workshop.itera.com for example.)

Once booted - some tidy up:

- Give your "non-root" user sudo (su - root, visudo)
- Install curl if missing (apt install curl)
- If you want to edit the yaml in visual code _inside_ the VM - install it from the MS download site - I will be copying in files over ssh
- Install the spice-vdagent apt package

### Networking

UTM sets up DHCP ion the 192.168.64.\* net. This is not good for testing agents - since the IP addresses change.

In XFCE - Settings > Advanced network > Fixed IPv4.

Set the control plane node (the main node) to 192.168.64.2, netmask /24, gateway 192.168.64.1
Any agents - just go with .3, .4 etc.

Add them to each hosts /etc/hosts file for ease of reference.

### Resizing

XFCE on debian doesn't properly detect resizes. Fix is to run `xrandr --output Virtual-1 --auto` - I created an alias for that.
