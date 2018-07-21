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

### Install babeld

Tinc, which we will install just after this, is only used to connect all devices
in a switch mode. To route the traffic a dynamic routing protocol is required.
To be compatible with earlier setups of *LibreNet6* this setup sticks to
`babeld`.

Install the routing protocol via the following command.

    apt install babeld

Now add the following line in `/etc/babeld.conf` to let the server announce that
it can route *IPv6* traffic.

    redistribute ip 0::0/0 le 0  allow

Restart the `babeld` daemon to be sure it loaded the new configuration.

    /etc/init.d/babeld restart
    # or via systemctl
    systemctl restart babeld

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


Once installed we have to setup the `librenet6`, to automatically created all
required files run the following command. The current host will be known to the
network based on the current hostname, you mach change that manually.

    tinc init -n librenet6 "server_$HOSTNAME"

To tell `tinc` to automatically start the `librenet6` network, we have to add it
to the `/etc/tinc/nets.boot` file, the easiest way to do so is via the following
command.

    echo librenet6 >> /etc/tinc/nets.boot

Now `tinc` needs some information on how to handle handle the `librenet6`
network interface. As we let `babeld` all the magic, we don't need to set a
specific *IP* address. Please add the following content to
`/etc/tinc/librenet6/tinc-up`

    ip -6 link set $INTERFACE up txqueuelen 1000
    babeld -D $INTERFACE

That's all! Start the VPN like this.

    tinc -n librenet6 start

### Inviting a gateway

So once the combination of `babeld` and `tinc` is running, it is possible to
add other routers, here called *IPv6 gateways*. To add new *gateways* they are
invited, as that is by far the easiest step for configuration!

To invite a node called one simply run the following command:

    tinc -n librenet6 invite gateway_<hostname>

To easily distinguish between *gateways* and *servers* both should be prefixed
with this identifying string. The *hostname* you choose will later be known to
all other clients of the network.

    209.97.142.116/-Qn8CtuuJaS4P419RB2AMTSioSyc24aP_xwJr6BwfOQ7Yp6T

This invitation contains three parts:

1. Starting with the public address of the *server*, this can be both an *IP* or
   a domain.
2. A hash of the *servers* public key, to allow the *gateway* the verification
   that it is talking to the correct *server*.
3. A shared secret, called cookie, which allows the client to authenticate
   itself as being previously invited.

All is done in a secret manner, so no extra HTTPS or anything similar is
required to be setup on the *server* instance. To read more on this checkout the
online documentation covering
[invitations](http://tinc-vpn.org/documentation-1.1/How-invitations-work.html#How-invitations-work).

The created invitation has to be given to the *gateway* owner in a secret
manner, as anyone with this link can connect to the server, without further
authentication. Also it is important to know that an invitation works only once!
