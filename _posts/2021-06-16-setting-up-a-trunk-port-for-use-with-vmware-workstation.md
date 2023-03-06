---
layout: post
title: Setting up a Trunk Port for use with VMware Workstation
date: 2021-06-16 15:02
---
<!-- wp:heading -->
<h2>Introduction</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"dropCap":true} -->
<p class="has-drop-cap">I could not do my job without a lab environment. I do my utmost to ensure that whatever changes I make will not have unintended consequences. Do they happen anyways? Yes. Do they happen less frequently? Also yes. This post will show how to create a trunk port, how to setup the NIC in Windows 10, and how to bridge the virtual adapters to virtual networks within VMware Workstation.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Prerequisites</h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li><a rel="noreferrer noopener" href="https://www.vmware.com/products/workstation-pro.html" target="_blank">VMware Workstation</a><ul><li><font size="3px">Can <em>technically</em> use <a rel="noreferrer noopener" href="https://www.vmware.com/products/workstation-player.html" target="_blank">VMware Player</a>, but requires <strong>vmnetcfg.exe</strong> from Workstation to configure networking. I will <strong><em>not </em></strong>be explaining how to set this up, as I am unsure of the legality.</font> </li></ul></li><li>Networking equipment with <a rel="noreferrer noopener" href="https://en.wikipedia.org/wiki/IEEE_802.1Q" target="_blank">802.1q</a> and trunking support.</li><li>Ethernet NIC with 802.1q and trunking support.</li></ul>
<!-- /wp:list -->

<!-- wp:heading -->
<h2>Equipment &amp; Software Used</h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li><a rel="noreferrer noopener" href="https://www.juniper.net/us/en/products-services/switching/ex-series/ex4300/" target="_blank">Juniper EX4300</a></li><li>VMware Workstation 16.1.2</li><li>Intel I211 Gigabit NIC <em><font size="2px">(built into motherboard, ASUS ROG STRIX B450-F)</font></em></li><li>Windows 10 Pro</li></ul>
<!-- /wp:list -->

<!-- wp:heading -->
<h2>Initial Setup</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The network environment should be setup prior (VLANs, networks, and services like DHCP, DNS, etc). We will only be changing the configuration for a single interface port that connects to your PC.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The advanced NIC drivers should be installed on your workstation first. In my case, Intel offers advanced NIC configuration via the <a rel="noreferrer noopener" href="https://downloadcenter.intel.com/download/22283/Intel-Ethernet-Adapter-Complete-Driver-Pack" target="_blank">Intel® Ethernet Adapter Complete Driver Pack</a> and using the <strong>Intel(R) PROSet Adapter Configuration Utility</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":270,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="/images/2021-06-16_i1.png?w=1024" alt="" class="wp-image-270" /></figure>
<!-- /wp:image -->

<!-- wp:heading -->
<h2>Trunk Port Configuration</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Now that the driver is installed, hop into your switch. You'll see my access port configuration below. We're going to convert the <strong>ge-0/0/13</strong> interface from an access port to a trunk port.</p>
<!-- /wp:paragraph -->

<!-- wp:code {"style":{"typography":{"fontSize":"14px"}}} -->
<pre class="wp-block-code" style="font-size:14px;"><code><p class="has-background-color has-text-color has-background" style="background-color:#000000;"><strong>show interfaces ge-0/0/13 | display set</strong>
set interfaces ge-0/0/13 description "Access Port"
set interfaces ge-0/0/13 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/13 unit 0 family ethernet-switching vlan members WIRED</p></code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>This port is configured as a member of the WIRED VLAN, but I want to test other networks from within VMware Workstation. For instance, I have a VM that is setup as a Student Windows 10 device. When the VM is a part of the STUDENT VLAN, I can test ACLs, QoS, and emulate how a device will act in its entirety. I also add other VLANs to allow for testing from staff devices and the guest network. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I set the WIRED VLAN (1122) as the native VLAN so that any wired device can still connect. I've had issues with NIC drivers crashing. This has left me disconnected due to VLAN trunking being disabled when the driver recovers. By using the native VLAN, I always fall back to a working state. <em>Note: A newer driver version has fixed the crashing issue (26.3).</em></p>
<!-- /wp:paragraph -->

<!-- wp:code {"style":{"typography":{"fontSize":"14px"}}} -->
<pre class="wp-block-code" style="font-size:14px;"><code><p class="has-background-color has-text-color has-background" style="background-color:#000000;"><strong>show interfaces ge-0/0/13 | display set</strong>
set interfaces ge-0/0/13 description "VM Trunk"
set interfaces ge-0/0/13 native-vlan-id 1122
set interfaces ge-0/0/13 unit 0 family ethernet-switching interface-mode trunk
set interfaces ge-0/0/13 unit 0 family ethernet-switching vlan members WIRED
set interfaces ge-0/0/13 unit 0 family ethernet-switching vlan members STUDENT
set interfaces ge-0/0/13 unit 0 family ethernet-switching vlan members INSTRUCTION
set interfaces ge-0/0/13 unit 0 family ethernet-switching vlan members GUEST</p></code></pre>
<!-- /wp:code -->

<!-- wp:heading -->
<h2>VLAN Virtual Adapters</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The next step is to add the VLAN virtual adapters within the NIC configuration. Click the <strong>Teaming/VLANs</strong> tab within <strong>PROSet </strong>(or equivalent tool) to get started. You'll click <strong>New</strong> at the bottom to create a new virtual adapter. Upon creating each virtual adapter, the tool may hang. Wait until it responds again.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":264,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-06-16_i2.png?w=405" alt="" class="wp-image-264" /><figcaption>New VLAN Dialog Box</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I name each VLAN the same as it is on the switch for easily identification. For the native VLAN, check the <strong>Untagged</strong> box. All other VLANs should be added using their assigned IDs and leaving the box unchecked.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":262,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-06-16_i3.png?w=1024" alt="" class="wp-image-262" /><figcaption>Finalized NIC configuration.</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>By adding all the VLANs, Windows will now display new adapters within the <strong>Network Connections tool</strong> (Control Panel). <em>Ignore the VMware Network Adapters for now.</em> You can now see that there are <strong>five </strong>I211 Adapters. One is the base physical adapter, with 4 virtuals for each VLAN.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":273,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-06-16_i4.png?w=1024" alt="" class="wp-image-273" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>My personal preference is to uncheck <strong>TCP/IPv4</strong> and <strong>TCP/IPv6</strong> on all VLANs that are going to be used for VMs only. I do not need additional IP addresses assigned to my PC, and by unchecking those features, they will not provide those services to the host. Right-click each adapter and go into their properties to change these settings.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Settings in my environment:<br>     <strong>WIRED</strong> (Untagged): TCP/IPv4 Enabled only<br>     <strong>STUDENT/INSTRUCTION/GUEST</strong> (Tagged): No TCP/IPv4 or TCP/IPv6</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":275,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-06-16_i5.png?w=726" alt="" class="wp-image-275" /><figcaption>Left: Base Physical Adapter, no label/tag.<br>Right: Individual Virtual Adapter tagged with VLAN : GUEST</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:heading -->
<h2>VMware Configuration</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Now that the virtual adapters have been created and configured, we will move on to bridging them with VMware Workstation. In the Workstation software, go to <strong>Edit -&gt; Virtual Network Editor</strong>. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":295,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="/images/2021-06-16_i6.png?w=1024" alt="" class="wp-image-295" /></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>When the window appears, click <strong>Change Settings</strong> in the bottom right.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":279,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-06-16_i7.png?w=603" alt="" class="wp-image-279" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>You'll recognize VMnet1 and VMnet8 from the previous Network Connections screen. We're going to leave those alone. VMnet1 is for communicating to our host machine only, and VMnet8 allows the VM to be NATed. We don't want to do this.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>We want to assign VMs to different VLANs directly, by bridging them to the virtual adapter. Click on <strong>Add Network...</strong> and select a VMnet you want to assign it to.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":285,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-06-16_i8.png?w=257" alt="" class="wp-image-285" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Select the new network in the list, then click the radial button next to <strong>Bridged</strong>. In the <strong>Bridged to:</strong> section, select the virtual adapter you want to use. They will be named exactly the same as they were in the Network Connections tool.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":286,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="/images/2021-06-16_i9.png?w=591" alt="" class="wp-image-286" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Lastly, you can click the <strong>Rename Network...</strong> button to give the connection a friendly name, instead of VMnet<em>x</em>. This will make it easier to identify when assigning connections in VM configuration. When you're done, press <strong>OK</strong>. Do this for all VLANs you want to use within VMware Workstation.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>VM Configuration</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The last thing to do is assign the connections you just created to VMs. In the <strong>Virtual Machine Settings</strong> for your VM, select <strong>Network Adapter</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":290,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="/images/2021-06-16_i10.png?w=750" alt="" class="wp-image-290" /></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Select the radial button for <strong>Custom:</strong> and then choose the connection you want to use. I renamed one of the VMnets to GUEST, so I can easily identify the VLAN/network. You're done! If the VM is currently running, you will need to renew the DHCP lease or statically assign a new IP from the network the VLAN resides in.</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2><span style="text-decoration:underline;">Changelog</span></h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>Initial Release<ul><li>2021/06/16</li></ul></li><li>Updated Software with Windows 10 Pro<ul><li>2021/07/10</li></ul></li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->
