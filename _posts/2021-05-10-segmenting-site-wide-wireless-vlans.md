---
layout: post
title: Segmenting Site-Wide Wireless VLANs
date: 2021-05-10 13:05
author: zakkaru
comments: true
categories: [Juniper, Networking, PowerShell, Professional, Projects, VLAN, Wireless]
---
<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Introduction</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"dropCap":true} -->
<p class="has-drop-cap">When I first arrived to the district, I was told that <strong>network performance takes a huge hit during testing</strong>. The theory was there were simply <strong><em>too</em> </strong>many users connected at the same time. Many discussions with staff advised them to stagger testing periods to alleviate the issue. This never happened.</p>
<!-- /wp:paragraph --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Key Facts</h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li><span style="text-decoration:underline;"></span>Each site has a 500-1000Gbps<sup>1</sup> circuit.</li><li>Testing is primarily text-based.</li><li>Every classroom has <em>at least</em> one AP<sup>2</sup>.</li><li>Class sizes are typically less than 30.</li><li>WAN<sup>3</sup> links were not congested.</li><li>Students could get disconnected from the AP and reconnect to another.</li></ul>
<!-- /wp:list --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Discovery</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Months in, I was auditing network configurations. Going through all of the interfaces, VLANs<sup>4</sup>, LLDP<sup>5</sup>, and more. I wanted to become more familiar with the network and gain a better understanding. Shortly after, I realized what the problem was: <strong>site-wide broadcast</strong> <strong>traffic</strong><sup>6</sup><strong>.</strong> </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Wireless VLANs were served from our distribution switch to an entire site. Using Wireshark, I viewed traffic during a normal, non-test day and it was excessive. Most staff used wireless for laptops and phones, with very few using a hard wired connection. Interestingly enough, all other VLANs were broken down by building(wired, VoIP<sup>7</sup>, etc)--leaving only the wireless VLANs spanning the entire site.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Each site had <strong>three VLANs</strong> that handled wireless traffic spanning the entire site, one VLAN per SSID<sup>8</sup>/subnet<sup>9</sup>. Broadcast traffic would flood the entire site from any client at any building. During testing periods, there could be <strong>over 100 students on their laptops</strong>. This is in addition to printers, casting devices, and more that already use the wireless.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":61,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="/images/2021-05-20_i1.png?w=929" alt="" class="wp-image-61" /><figcaption>This image displays a simplified network diagram for the network.<br>The smaller colored circles within each of the areas describes the broadcast domains at each site prior to any changes.<br>Each area is a separate site.</figcaption></figure>
<!-- /wp:image -->

<!-- wp:image {"align":"center","id":212,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-05-20_i2.png?w=367" alt="" class="wp-image-212" /></figure></div>
<!-- /wp:image --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Solution</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>There were a couple options that I considered. Implement some kind of <strong>broadcast reduction</strong> using the wireless controller, or <strong>network segmentation</strong><sup>10</sup>. Using segmentation, I could setup more control and customization over the network moving forward (<em>ACLs/QoS/etc</em>).</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>This project quickly became more than I expected</strong>. It was my <em>first </em>major project, and I intended to <em>get it right the first time</em>. By breaking up the site-wide VLANs, I now needed <strong>3 VLANs and DHCP</strong><sup>11</sup><strong> scopes per building</strong>. This <strong>more than doubled </strong>the amount of DHCP scopes already in use. Luckily, PowerShell<sup>12</sup> made this process easy using a script.</p>
<!-- /wp:paragraph -->

<!-- wp:code {"style":{"typography":{"fontSize":"14px"}}} -->
<pre class="wp-block-code" style="font-size:14px;"><code><p class="has-background-color has-text-color has-background" style="background-color:#012456;"><span style="text-decoration:underline;"><strong>Sample Powershell Script</strong></span>

Add-DhcpServerv4Scope -StartRange 10.1.128.2 -EndRange 10.1.129.254 -SubnetMask 255.255.254.0 -Name "SITE_WIRELESS_SSID1" -LeaseDuration 0:08:00

Set-DhcpServerv4OptionValue -ScopeId 10.1.128.0 -DnsServer 10.0.8.5,10.0.8.6 -DnsDomain "contoso.com" -Router 10.1.128.1 -Force</p></code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>I decided to use /23 networks instead of the previous /21s. <em>"But Zach, I thought you wanted to make the broadcast domains<sup>13</sup> smaller?"</em> I did, but I am also lazy and wanted to leave room for pre-planned growth. By choosing a /23 for each new network, I can pair them down as needed. Since this address space has never been in use, I took the easy route. 👍</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":62,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="/images/2021-05-20_i3.png?w=926" alt="" class="wp-image-62" /><figcaption>After segmentation, you can see each of the individual L3 switches are in their own broadcast domains. <br>The exception is Area 5, which did not house any wireless traffic and was significantly smaller.</figcaption></figure>
<!-- /wp:image --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Deployment</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>During the week of deployment, I preloaded all new configurations and verified connectivity during the day. Configuration involved addressing<sup>14</sup>, helper addresses<sup>15</sup>, OSPF<sup>16</sup> inclusion, etc. A co-worker and I planned to deploy this within <strong>2-3 hours</strong> at the end of a Friday.</p>
<!-- /wp:paragraph -->

<!-- wp:code {"style":{"typography":{"fontSize":"14px"}}} -->
<pre class="wp-block-code" style="font-size:14px;"><code><p class="has-background-color has-text-color has-background" style="background-color:#000000;"><strong><span style="text-decoration:underline;">Sample Juniper Configuration</span></strong>

<strong>show vlans | display set</strong>
set vlans SSID1 vlan-id 5
set vlans SSID1 l3-interface irb.5
set vlans SSID2 vlan-id 6
set vlans SSID2 l3-interface irb.6
set vlans SSID3 vlan-id 7
set vlans SSID3 l3-interface irb.7

<strong>show interfaces irb | display set</strong>
set interfaces irb unit 5 family inet address 10.1.128.1/23
set interfaces irb unit 6 family inet address 10.1.164.1/23
set interfaces irb unit 7 family inet address 10.1.200.1/23

<strong>show forwarding-options dhcp-relay | display set</strong>
set forwarding-options dhcp-relay server-group SITE_DC 10.0.8.5
set forwarding-options dhcp-relay group SITE_DC active-server-group SITE_DC
set forwarding-options dhcp-relay group SITE_DC interface irb.5
set forwarding-options dhcp-relay group SITE_DC interface irb.6
set forwarding-options dhcp-relay group SITE_DC interface irb.7

<strong>show protocols ospf area 1 | display set</strong>
set protocols ospf area 0.0.0.1 interface irb.5 passive
set protocols ospf area 0.0.0.1 interface irb.6 passive
set protocols ospf area 0.0.0.1 interface irb.7 passive

<em>Note: IP addresses and VLANs have been changed</em> <em>and are not reflective of the actual configurations.</em></p></code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>There were some hurdles the night of deployment. We wanted to standardize the server IP design across sites, in addition to the wireless changes. That caused some other issues. Implementing the new VLANs was a matter of moving APs into groups based on closet within the wireless controller<sup>17</sup>. Logic in the controller was then used to put each SSID into the proper VLAN.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Due to the other unplanned issues with server changes, it took about 3x longer to get everything running smoothly. <em>We were still able to get it done in a single night <strong>thankfully</strong></em>.</p>
<!-- /wp:paragraph --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Day One</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>On the following day, there were a few bumps. The RADIUS<sup>18</sup> servers were not properly setup in the controller, leaving staff unable to connect to the Instruction network--the primary SSID in use. We were hung up on this the previous night, so there was some familiarity. Long story short, we didn't point each site to their specific RADIUS server. We were able to resolve the issue prior to the start of classes. Following that, everything ran smooth.</p>
<!-- /wp:paragraph --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Motivation</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>A large driving factor for this project was students returning to the classroom from distance learning. I figured with their return, they would be on their devices more than ever--and that has definitely been the case.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The next round of testing arrived at each of the sites, and all of us in the tech department almost forgot. We didn't hear any complaints about the network being slow, being dropped, or anything really. The project was a <em>quiet </em>success, but it felt great knowing that technology would not get in the way of testing any longer.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>An added bonus was that we could now see and track which building someone was in. IPs now give insights into locations and tracking down devices could be done by searching the scopes and coordinating it with the MAC<sup>19</sup> tables. (Search for the MAC at the distribution switch<sup>20</sup> down to the access switch<sup>21</sup>).</p>
<!-- /wp:paragraph --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Lessons Learned</h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>Do not do too much at one time.</li><li>Planning pays huge dividends (I couldn't imagine trying to do all of this the same night).</li><li>Documentation could have saved a lot of headache.</li><li>Powershell knowledge is important.</li><li>Double check OSPF configuration.</li></ul>
<!-- /wp:list --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>Key Terms</h2>
<!-- /wp:heading -->

<!-- wp:list {"ordered":true} -->
<ol><li><strong>Gbps</strong>: Gigabits per second. Used as a measurement of network bandwidth.</li><li><strong>AP</strong>: Access Point. A device that provides wireless connectivity.</li><li><strong>WAN</strong>: Wide Area Network. Connects geographically separate networks together.</li><li><strong>VLAN</strong>: Virtual Local Area Network. A logical construct used to segment layer 2 networks. Defined in IEEE 802.1q.</li><li><strong>LLDP</strong>: Link Layer Discovery Protocol. A layer 2 protocol used to advertise device information such as capabilities and connectivity information.</li><li><strong>Broadcast Traffic</strong>: Data that is sent to all hosts within a broadcast domain.</li><li><strong>VoIP</strong>: Voice over IP. Protocols used to transmit voice over network packets.</li><li><strong>SSID</strong>: Service Set Identifier. Identifies a network used by wireless devices.</li><li><strong>Subnet</strong>: A network that is a subset of a larger network.</li><li><strong>Network Segmentation</strong>: The act of breaking up a network into different components based on some other aspect, such as by department, function, location, level of security, etc.</li><li><strong>DHCP</strong>: Dynamic Host Configuration Protocol. Used to automatically configure network information on host devices.</li><li><strong>PowerShell</strong>: A scripting language typically used for configuration tasks; created by Microsoft. </li><li><strong>Broadcast Domain</strong>: A logical scope of devices that will hear broadcast traffic.</li><li>[IP] <strong>Addressing</strong>: Layer 3 information used to route packets. This includes IP addresses and subnet masks.</li><li><strong>Helper Address</strong>: In this blog post, a helper address is used to forward DHCP traffic to the DHCP server, that would otherwise not reach it. DHCP broadcasts can be sent to other networks, allowing hosts to receive addressing information from a centralized server.</li><li><strong>OSPF</strong>: Open Shortest Path First. A link-state dynamic routing protocol. It floods various link state advertisement messages containing routing information to all participating neighbors. An area concept is used to logically separate and control routing data. Each router uses its link state database to calculate the best paths to destinations using <a href="https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm">Dijkstra's algorithm</a>.</li><li><strong>Wireless Controller</strong>: A device (or cloud solution) that controls and configures all APs in a centralized location. Typically, uses <a href="https://en.wikipedia.org/wiki/CAPWAP">CAPWAP </a>tunnels from APs to the controller.</li><li><strong>RADIUS</strong>: Remote Access Dial-In User Service. Provides a means for authentication, authorization, and accounting for users on a network.</li><li><strong>MAC</strong>: Media Access Control. In this blog, it refers to a MAC address, which is the physical address of a network interface card.</li><li><strong>Distribution Switch</strong>: A switch that aggregates all access switches.</li><li><strong>Access Switch</strong>: A switch that provides hosts access to the network.</li></ol>
<!-- /wp:list --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2>To Readers</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>This was my <em>first</em> <em>actual tech blog</em>. There is probably a lot to be desired, so I invite anyone reading to provide feedback. One thing I think could add a lot would be more images, but I wasn't quite sure what exactly. I'm open to go back, add more, and edit; so let me know. 😃</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>--Zach</p>
<!-- /wp:paragraph --></div>
<!-- /wp:group -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:heading -->
<h2><span style="text-decoration:underline;">Changelog</span></h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>Initial Release<ul><li>2021-05-10</li></ul></li><li>Add PowerShell/Network configuration examples.<ul><li>2021-05-20</li></ul></li><li>Add and explain key terms.<ul><li>2021-05-20</li></ul></li><li>Add Icon Key/Map to previous images.<ul><li>2021-05-20</li></ul></li><li><s>Add additional images.</s><ul><li><s><strong><em>TBD</em></strong> (Planned 2021-05-12)</s></li></ul></li></ul>
<!-- /wp:list --></div>
<!-- /wp:group -->
