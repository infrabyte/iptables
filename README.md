# iptables
IPtables howto

iptables almost always comes pre-installed on any Linux distribution. To update/install it, just retrieve the iptables package:
sudo apt-get install iptables
Types of Chains

iptables uses three different chains: input, forward, and output.

Input – This chain is used to control the behavior for incoming connections. For example, if a user attempts to SSH into your PC/server, iptables will attempt to match the IP address and port to a rule in the input chain.

Forward – This chain is used for incoming connections that aren’t actually being delivered locally. Think of a router – data is always being sent to it but rarely actually destined for the router itself; the data is just forwarded to its target. Unless you’re doing some kind of routing, NATing, or something else on your system that requires forwarding, you won’t even use this chain.

There’s one sure-fire way to check whether or not your system uses/needs the forward chain.
iptables -L -v

Output – This chain is used for outgoing connections. For example, if you try to ping howtogeek.com, iptables will check its output chain to see what the rules are regarding ping and howtogeek.com before making a decision to allow or deny the connection attempt.

The caveat

Even though pinging an external host seems like something that would only need to traverse the output chain, keep in mind that to return the data, the input chain will be used as well. When using iptables to lock down your system, remember that a lot of protocols will require two-way communication, so both the input and output chains will need to be configured properly. SSH is a common protocol that people forget to allow on both chains.

More times than not, you’ll want your system to accept connections by default. Unless you’ve changed the policy chain rules previously, this setting should already be configured. Either way, here’s the command to accept connections by default:
    iptables --policy INPUT ACCEPT
    iptables --policy OUTPUT ACCEPT
    iptables --policy FORWARD ACCEPT

If you would rather deny all connections and manually specify which ones you want to allow to connect, you should change the default policy of your chains to drop. Doing this would probably only be useful for servers that contain sensitive information and only ever have the same IP addresses connect to them.
    iptables --policy INPUT DROP
    iptables --policy OUTPUT DROP
    iptables --policy FORWARD DROP

Connections from a single IP address

This example shows how to block all connections from the IP address 10.10.10.10.
iptables -A INPUT -s 10.10.10.10 -j DROP

This example shows how to block all of the IP addresses in the 10.10.10.0/24 network range. You can use a netmask or standard slash notation to specify the range of IP addresses.
iptables -A INPUT -s 10.10.10.0/24 -j DROP

This example shows how to block SSH connections from 10.10.10.10.
iptables -A INPUT -p tcp --dport ssh -s 10.10.10.10 -j DROP

This example shows how to block SSH connections from any IP address.
iptables -A INPUT -p tcp --dport ssh -j DROP

what if you only want SSH coming into your system to be allowed? Won’t adding a rule to the output chain also allow outgoing SSH attempts?

That’s where connection states come in, which give you the capability you’d need to allow two way communication but only allow one way connections to be established. Take a look at this example, where SSH connections FROM 10.10.10.10 are permitted, but SSH connections TO 10.10.10.10 are not. However, the system is permitted to send back information over SSH as long as the session has already been established, which makes SSH communication possible between these two hosts.
    iptables -A INPUT -p tcp --dport ssh -s 10.10.10.10 -m state --state NEW,ESTABLISHED -j ACCEPT

    iptables -A OUTPUT -p tcp --sport 22 -d 10.10.10.10 -m state --state ESTABLISHED -j ACCEPT

Saving Changes

The changes that you make to your iptables rules will be scrapped the next time that the iptables service gets restarted unless you execute a command to save the changes.  This command can differ depending on your distribution:

Ubuntu:

    sudo /sbin/iptables-save

Red Hat / CentOS:

    /sbin/service iptables save

Or

    /etc/init.d/iptables save
