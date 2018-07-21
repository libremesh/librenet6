



## Server setup

This manual requires a Debian machine to run, minimal system requirements are to
have either a public IPv6 or IPv4 addresses.

To use the new features of Tinc 1.1, you have to enable the `experimental`
package sources. To do so add the following line to `/etc/apt/sources.list`

    deb http://deb.debian.org/debian experimental main

Once added update the repository to allow the installation of the *experimental*
tools.

    apt update


### With public IPv6

Make sure the connection actually works by pinging a website only reachable over
*IPv6*

    ping ipv6.ident.me
    # or
    ping ipv6.google.com

### With public IPv4 only

With a public IPv4 it is possible to setup a *6to4 tunnel*. To simplify the
setup Debian offers a package to automatically setup the tunnel called
`auto6to4`. Install it via the following command.

    apt install -t experimental auto6to4

Once installed the configuration is done automatically and reboot safe. Test the
connection by pinging a website which is only reachable over *IPv6*

    ping ipv6.ident.me
    # or
    ping ipv6.google.com

To know your new *IPv6* address run the following command, requiring `curl` or
`wget` installed on the host system.

    curl ipv6.ident.me
    # or
    wget -qO- ipv6.ident.me

### Install Tinc

To connect the *IPv6 server*, the one we currently work on, with the *IPv6
gateways*, the routers which normally would only have *IPv4* addresses, the VPN
tool *tinc* is used. It offers a simple way to connect multiple devices to a
*virtual* network. This allows to add *IPv6 servers* to provide real *IPv6*
uplink, but also to connect within the *virtual* network directly between *IPv6
gateways* with the same *IPv6* addresses.

First install the experimental version of `tinc` which offers new convenience
features like `join` and `invite`, more on this in a second.

    apt install -t experimental tinc

Once installed we have to setup a new network, first create a folder to store
the `tinc` setup with the following command.

    mkdir -p /etc/tinc/librenet6

All further `tinc` commands will contain the `-n librenet6` argument so the tool
know what network we want to work on.

### Install babeld 

Tinc is only used to connect all devices in a switch mode. To route the traffic
a dynamic routing protocol is required. To be compatible with earlier setups of
*LibreNet6* this setup sticks to `babeld`.

Install the routing protocol via the following command.

    apt install babeld

Now add the following line in `/etc/babeld.conf` to let the server announce that
it can route *IPv6* traffic.

    redistribute ip 0::0/0 le 0  allow

Restart the `babeld` daemon to be sure it loaded the new configuration.

    /etc/init.d/babeld restart
    # or via systemctl
    systemctl restart babeld

