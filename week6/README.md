# Week 6 lab: Virtualisation, overlays and software forwarding

## 1. List the Linux building blocks behind one Mininet host and one Mininet cable.

A Mininet host is a bash process in its own network namespace. `dump` showed h1 as pid 26 and h2 as pid 28, and `lsns` showed the same PIDs with their own namespaces.

A Mininet cable is a veth pair which is two virtual interfaces joined together, one end in the host (`h1-eth0`) and the other on the switch (`s1-eth1`). The switch is a Linux bridge.

## 2. Show your MTU and throughput measurements for underlay and overlay, and explain the 50-byte difference header by header. Why does the same tunnel cost 70 bytes over IPv6?

Underlay (`h1-eth0`)
- MTU : 1500
- Largest ping with `-M do` : -
- iperf3 receiver : 25.7 Gbits/s

Overlay (`vxlan100`)
- MTU : 1450
- Largest ping with `-M do` : 1422 works, 1423 fails
- iperf3 receiver : 20.1 Gbits/s

The overlay was about 22% slower because every packet has to be wrapped and unwrapped and carries 50 fewer bytes of data.

The 50 bytes are VXLAN's extra headers: inner Ethernet (14) + VXLAN (8) + UDP (8) + outer IPv4 (20) = 50. So the overlay MTU is 1500 − 50 = 1450.

Over IPv6, the outer header is 40 bytes instead of 20, so the tunnel costs 70 bytes and its MTU is 1430. That's why 1402 worked while 1403 failed.

## 3. Why did small pings keep working during the MTU mismatch while TCP almost stopped?

After h2's underlay MTU was set to 1400, small pings still fit, so they got through. Large packets were too big and were dropped at h2 without any error. `tx_dropped` on s1-eth2 and `rx_dropped` on h2-eth0 both went up from 13 to 16 after 3 large pings.

TCP sends large packets and almost all of them were dropped. No error came back to tell h1 to send smaller ones, so TCP kept retrying and fell to 2.30 Mbit/s.

## 4. What does a Docker container have that a Mininet host doesn't?

A Docker container has its own filesystem (`/etc/os-release` showed Alpine Linux) and its own processes (`ps` showed only `sleep 600` and `ps`). A Mininet host only has its own network namespace. It shares the filesystem and processes with the lab container.

## 5. Compare this VXLAN tunnel with your Week 5 P4 tunnel: how many bytes each adds, which devices on the path must understand it, and what stops a packet looping forever.

VXLAN
- Bytes added : 50 (70 for IPv6)
- Devices that must understand it : Only the two ends (h1 and h2)
- What stops a loop : The outer IP header's TTL

P4 tunnel
- Bytes added : 4 (5 with my hop limit)
- Devices that must understand it : Every switch on the path
- What stops a loop : nothing until I added the hop limit.
