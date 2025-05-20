+++
title = 'From Domain to Destination'
date = 2025-05-21T00:08:30+05:30
draft = false
summary= 'How Network Requests Find Their Way'
+++
Have you ever wondered what happens when you type a website address into your browser? You probably know that DNS (Domain Name System) converts website names into IP addresses. But I've always been curious - once it gets the IP address, how does the request actually find its way to that specific server?

Turns out, all the magic happens in the routing tables at each machine/router along the way. In this post, I'll explain how these help guide your request to its destination.

## 1. DNS Resolution: Getting the IP Address
When you type a website name like "abhilashmallikarjuna.in" in your browser, your computer first needs to find its IP address. Your system's DNS resolver handles this automatically, but you can also see this process using the `dig` command-line tool:

```bash
> dig +short abhilashmallikarjuna.in
216.24.57.1
```
## 2. Your Computer's Routing Table: The First Step
Once your computer has the IP address, it needs to figure out how to send data to that address. For this, it checks its own routing table. You can view your computer's routing table with the following command (the exact command varies by operating system):

```bash
❯ ip route show
default via 192.168.0.1 dev en0
127.0.0.0/8 via 127.0.0.1 dev lo0
127.0.0.1/32 via 127.0.0.1 dev lo0
169.254.0.0/16 dev en0 scope link
192.168.0.0/24 dev en0 scope link
192.168.0.1/32 dev en0 scope link
192.168.0.5/32 dev en0 scope link
224.0.0.0/4 dev en0 scope link
255.255.255.255/32 dev en0 scope link
```
Let's understand what some of these entries mean:

`**192.168.0.0/24 dev en0 scope link**`
→ This tells your computer that any IP address in the range 192.168.0.0 to 192.168.0.255 is on your local network. Your machine can send data directly to these addresses through the en0 network interface.

`**default via 192.168.0.1 dev en0**`
→ It tells your computer that for any IP address not matching other rules (like our website at 216.24.57.1), send the data to 192.168.0.1 first. This address is your "default gateway" - typically your home router or office firewall.
→ Our website IP falls in this category, so the request journey continues to our home router.

## 3. Hop-by-Hop Forwarding: The Internet Journey
Once your packet reaches your home router (192.168.0.1), the real journey begins. The router checks its own routing table to determine where to send your packet next.

1. **Your home router** looks at its routing table and forwards your packet to your **ISP's edge router**
2. **Your ISP's edge router** uses a protocol called **BGP** (Border Gateway Protocol) to communicate with other ISPs and determine the best path to reach your destination. BGP is like the postal service of the internet - it knows which networks connect to which other networks.
3. The ISP router might have an entry like:This means "to reach 216.24.57.1, send the packet to 198.51.100.1 next"216.24.57.1/24 → via next-hop 198.51.100.1 (announced by AS 13335)
4. **Each subsequent router** along the path does the same thing: looks up the destination IP in its routing table and forwards the packet to the next hop.
It's like a relay race, with your data packet being passed from router to router until it finally reaches the destination!
## Seeing the Path in Action: Traceroute
You can actually see the entire path your data takes using the `traceroute` command. This tool shows each "hop" (router) that your packet passes through:

```bash
❯ traceroute -v abhilashmallikarjuna.in
Using interface: en0
traceroute to abhilashmallikarjuna.in (216.24.57.1), 64 hops max, 40 byte packets
 1  192.168.0.1 (192.168.0.1) 48 bytes to 192.168.0.5  6.256 ms  2.989 ms  2.659 ms
 2  10.232.0.1 (10.232.0.1) 36 bytes to 192.168.0.5  5.246 ms  6.003 ms  4.304 ms
 3  * * *
 4  * * *
 5  121.242.109.165.static-bangalore.vsnl.net.in (121.242.109.165) 76 bytes to 192.168.0.5  8.172 ms  5.643 ms  6.069 ms
 6  172.25.138.158 (172.25.138.158) 148 bytes to 192.168.0.5  13.687 ms *  11.435 ms
 7  * * *
 8  if-bundle-34-2.qcore2.esin4-singapore.as6453.net (180.87.36.41) 148 bytes to 192.168.0.5  46.695 ms  45.609 ms  45.008 ms
 9  if-be-22-2.ecore2.svq-singapore.as6453.net (63.243.180.161) 148 bytes to 192.168.0.5  43.217 ms  42.579 ms
    if-bundle-2-2.qcore1.esin4-singapore.as6453.net (63.243.180.3) 148 bytes to 192.168.0.5  44.157 ms
10  162.158.160.149 (162.158.160.149) 36 bytes to 192.168.0.5  45.883 ms
    162.158.160.161 (162.158.160.161) 36 bytes to 192.168.0.5  44.279 ms
    if-ae-46-2.thar1.svq-singapore.as6453.net (120.29.214.10) 148 bytes to 192.168.0.5  53.483 ms
11  216.24.57.1 (216.24.57.1) 48 bytes to 192.168.0.5  43.161 ms  45.509 ms  42.983 ms
```
In this example, you can see:

- Hop 1: Your home router (192.168.0.1)
- Hop 2: Probably your Firewall
- Hops 3-4: Some routers that didn't respond to the traceroute
- Hop 5: A router in Bangalore
- Hops 8-9: Routers in Singapore
- Hop 11: Finally, the destination server (216.24.57.1)
The asterisks (*) indicate routers that didn't respond to the traceroute request, which is common as some routers are configured not to respond for security reasons.

## Want to Learn More?
If you're interested in learning more about how networking works, here are some related topics to explore:

1. **ARP (Address Resolution Protocol)**: How IP addresses are translated to physical MAC addresses on a local network
2. **BGP (Border Gateway Protocol)**: The routing protocol that powers the core of the internet
3. **OSPF (Open Shortest Path First)**: Another routing protocol commonly used within ISP networks
Understanding these concepts will give you a deeper appreciation of the amazing infrastructure that makes the internet work!


