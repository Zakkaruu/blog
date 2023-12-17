---
layout: post
title: Segmenting Site-Wide Wireless VLANs
date: 2021-05-10 13:05
---

## Introduction

When I first arrived to the district, I was told that network performance takes a huge hit during testing. The theory was there were simply too many users connected at the same time. Many discussions with staff advised them to stagger testing periods to alleviate the issue. This never happened.

## Key Facts

Each site has a 500-1000Gbps<sup>1</sup> circuit. Testing is primarily text-based. Every classroom has at least one AP<sup>2</sup>. Class sizes are typically less than 30. WAN<sup>3</sup> links were not congested. Students could get disconnected from the AP and reconnect to another.

## Discovery

Months in, I was auditing network configurations. Going through all of the interfaces, VLANs<sup>4</sup>, LLDP<sup>5</sup>, and more. I wanted to become more familiar with the network and gain a better understanding. Shortly after, I realized what the problem was: site-wide broadcast traffic<sup>6</sup>. 

Wireless VLANs were served from our distribution switch to an entire site. Using Wireshark, I viewed traffic during a normal, non-test day and it was excessive. Most staff used wireless for laptops and phones, with very few using a hard wired connection. Interestingly enough, all other VLANs were broken down by building(wired, VoIP<sup>7</sup>, etc)--leaving only the wireless VLANs spanning the entire site.

Each site had three VLANs that handled wireless traffic spanning the entire site, one VLAN per SSID<sup>8</sup>/subnet<sup>9</sup>. Broadcast traffic would flood the entire site from any client at any building. During testing periods, there could be over 100 students on their laptops. This is in addition to printers, casting devices, and more that already use the wireless.

![](/images/2021-05-20_i1.png)

This image displays a simplified network diagram for the network.The smaller colored circles within each of the areas describes the broadcast domains at each site prior to any changes. Each area is a separate site.

![](/images/2021-05-20_i2.png)

## Solution

There were a couple options that I considered. Implement some kind of broadcast reduction using the wireless controller, or network segmentation<sup>10</sup>. Using segmentation, I could setup more control and customization over the network moving forward (ACLs/QoS/etc).

This project quickly became more than I expected. It was my first major project, and I intended to get it right the first time. By breaking up the site-wide VLANs, I now needed 3 VLANs and DHCP<sup>11</sup> scopes per building. This more than doubled the amount of DHCP scopes already in use. Luckily, PowerShell<sup>12</sup> made this process easy using a script.

`Sample Powershell Script`

```powershell
Add-DhcpServerv4Scope -StartRange 10.1.128.2 -EndRange 10.1.129.254 -SubnetMask 255.255.254.0 -Name "SITE_WIRELESS_SSID1" -LeaseDuration 0:08:00

Set-DhcpServerv4OptionValue -ScopeId 10.1.128.0 -DnsServer 10.0.8.5,10.0.8.6 -DnsDomain "contoso.com" -Router 10.1.128.1 -Force</code></pre>
```

I decided to use /23 networks instead of the previous /21s. "But Zach, I thought you wanted to make the broadcast domains<sup>13</sup> smaller?" I did, but I am also lazy and wanted to leave room for pre-planned growth. By choosing a /23 for each new network, I can pair them down as needed. Since this address space has never been in use, I took the easy route. 👍

<!-- wp:image {"id":62,"sizeSlug":"large","linkDestination":"none"} -->
![](/images/2021-05-20_i3.png) After segmentation, you can see each of the individual L3 switches are in their own broadcast domains. The exception is Area 5, which did not house any wireless traffic and was significantly smaller.

## Deployment

During the week of deployment, I preloaded all new configurations and verified connectivity during the day. Configuration involved addressing<sup>14</sup>, helper addresses<sup>15</sup>, OSPF<sup>16</sup> inclusion, etc. A co-worker and I planned to deploy this within 2-3 hours at the end of a Friday.

`Sample Juniper Configuration`

```text
show vlans | display set
set vlans SSID1 vlan-id 5
set vlans SSID1 l3-interface irb.5
set vlans SSID2 vlan-id 6
set vlans SSID2 l3-interface irb.6
set vlans SSID3 vlan-id 7
set vlans SSID3 l3-interface irb.7

show interfaces irb | display set
set interfaces irb unit 5 family inet address 10.1.128.1/23
set interfaces irb unit 6 family inet address 10.1.164.1/23
set interfaces irb unit 7 family inet address 10.1.200.1/23

show forwarding-options dhcp-relay | display set
set forwarding-options dhcp-relay server-group SITE_DC 10.0.8.5
set forwarding-options dhcp-relay group SITE_DC active-server-group SITE_DC
set forwarding-options dhcp-relay group SITE_DC interface irb.5
set forwarding-options dhcp-relay group SITE_DC interface irb.6
set forwarding-options dhcp-relay group SITE_DC interface irb.7

show protocols ospf area 1 | display set
set protocols ospf area 0.0.0.1 interface irb.5 passive
set protocols ospf area 0.0.0.1 interface irb.6 passive
set protocols ospf area 0.0.0.1 interface irb.7 passive

Note: IP addresses and VLANs have been changed and are not reflective of the actual configurations.
```

There were some hurdles the night of deployment. We wanted to standardize the server IP design across sites, in addition to the wireless changes. That caused some other issues. Implementing the new VLANs was a matter of moving APs into groups based on closet within the wireless controller<sup>17</sup>. Logic in the controller was then used to put each SSID into the proper VLAN.

Due to the other unplanned issues with server changes, it took about 3x longer to get everything running smoothly. We were still able to get it done in a single night thankfully.

## Day One

On the following day, there were a few bumps. The RADIUS<sup>18</sup> servers were not properly setup in the controller, leaving staff unable to connect to the Instruction network--the primary SSID in use. We were hung up on this the previous night, so there was some familiarity. Long story short, we didn't point each site to their specific RADIUS server. We were able to resolve the issue prior to the start of classes. Following that, everything ran smooth.

## Motivation

A large driving factor for this project was students returning to the classroom from distance learning. I figured with their return, they would be on their devices more than ever--and that has definitely been the case.

The next round of testing arrived at each of the sites, and all of us in the tech department almost forgot. We didn't hear any complaints about the network being slow, being dropped, or anything really. The project was a quiet success, but it felt great knowing that technology would not get in the way of testing any longer.

An added bonus was that we could now see and track which building someone was in. IPs now give insights into locations and tracking down devices could be done by searching the scopes and coordinating it with the MAC<sup>19</sup> tables. (Search for the MAC at the distribution switch<sup>20</sup> down to the access switch<sup>21</sup>.

## Lessons Learned

Do not do too much at one time.Planning pays huge dividends (I couldn't imagine trying to do all of this the same night). Documentation could have saved a lot of headache.Powershell knowledge is important.Double check OSPF configuration.

## Key Terms

1. Gbps: Gigabits per second. Used as a measurement of network bandwidth.
2. AP: Access Point. A device that provides wireless connectivity.
3. WAN: Wide Area Network. Connects geographically separate networks together.
4. VLAN: Virtual Local Area Network. A logical construct used to segment layer 2 networks. Defined in IEEE 802.1q.
5. LLDP: Link Layer Discovery Protocol. A layer 2 protocol used to advertise device information such as capabilities and connectivity information.
6. Broadcast Traffic: Data that is sent to all hosts within a broadcast domain.
7. VoIP: Voice over IP. Protocols used to transmit voice over network packets.
8. SSID: Service Set Identifier. Identifies a network used by wireless devices.
9. Subnet: A network that is a subset of a larger network.
10. Network Segmentation: The act of breaking up a network into different components based on some other aspect, such as by department, function, location, level of security, etc.
11. DHCP: Dynamic Host Configuration Protocol. Used to automatically configure network information on host devices.
12. PowerShell: A scripting language typically used for configuration tasks; created by Microsoft.
13. Broadcast Domain: A logical scope of devices that will hear broadcast traffic.
14. IP Addressing: Layer 3 information used to route packets. This includes IP addresses and subnet masks.
15. Helper Address: In this blog post, a helper address is used to forward DHCP traffic to the DHCP server, that would otherwise not reach it. DHCP broadcasts can be sent to other networks, allowing hosts to receive addressing information from a centralized server.
16. OSPF: Open Shortest Path First. A link-state dynamic routing protocol. It floods various link state advertisement messages containing routing information to all participating neighbors. An area concept is used to logically separate and control routing data. Each router uses its link state database to calculate the best paths to destinations using <a href="https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm">Dijkstra's algorithm</a>.
17. Wireless Controller: A device (or cloud solution) that controls and configures all APs in a centralized location. Typically, uses <a href="https://en.wikipedia.org/wiki/CAPWAP">CAPWAP </a>tunnels from APs to the controller.
18. RADIUS: Remote Access Dial-In User Service. Provides a means for authentication, authorization, and accounting for users on a network.
19. MAC: Media Access Control. In this blog, it refers to a MAC address, which is the physical address of a network interface card.
20. Distribution Switch: A switch that aggregates all access switches.Access Switch: A switch that provides hosts access to the network.

## To Readers

This was my first actual tech blog. There is probably a lot to be desired, so I invite anyone reading to provide feedback. One thing I think could add a lot would be more images, but I wasn't quite sure what exactly. I'm open to go back, add more, and edit; so let me know. 😃

--Zach

## Changelog

- Initial Release
  - 2021-05-10
- Add PowerShell/Network configuration examples.
  - 2021-05-20
- Add and explain key terms.
  - 2021-05-20
- Add Icon Key/Map to previous images.
  - 2021-05-20
