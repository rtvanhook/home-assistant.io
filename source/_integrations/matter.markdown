---
title: Matter (BETA)
description: Instructions on how to integrate Matter with Home Assistant.
ha_category:
  - Binary Sensor
  - Climate
  - Cover
  - Light
  - Lock
  - Sensor
  - Switch
ha_release: '2022.12'
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@home-assistant/matter'
ha_domain: matter
ha_platforms:
  - binary_sensor
  - climate
  - cover
  - diagnostics
  - event
  - light
  - lock
  - sensor
  - switch
ha_integration_type: integration
---

The Matter integration allows you to control Matter devices on your local Wi-Fi or {% term Thread %} network.

<div class='note warning'>
The integration is marked BETA: Both the Matter standard itself and its implementation within Home Assistant are in a early stage. You may run into compatibility issues and/or other bugs.
</div>

# What is Matter?

Matter is [the new standard for home automation](<https://en.wikipedia.org/wiki/Matter_(standard)>), which has been released recently. It is in the process of being adopted by the tech industry. Matter is a local protocol. Device control is done without the need of any cloud. From a technical perspective, you can use a Matter-compatible device with Home Assistant without connecting to a vendor-specific cloud. However, some vendors may require you to set up an account before you can enable Matter.

Unlike the radio based protocols we're already familiar with in the IoT landscape, like Zigbee and Z-Wave, Matter uses standard IP-based communication. Matter is not a radio protocol, but a control protocol that runs on top of the existing network infrastructure, using your existing Wi-Fi/Ethernet routers and Thread.

Home Assistant is a so-called _controller_, it can control Matter based devices. Other examples of Matter controllers are the Google Nest speakers, Apple HomePods, and a SmartThings Station.

## Bridge devices

One of the great things about Matter is that you can have both Wi-Fi and Thread based devices on the same controller.
Next to actual devices (like actors or sensors), you will also see bridges. The bridge connects the network over Ethernet or Wi-Fi and bridges multiple devices into a Matter network. A great example is the Philips Hue V2 bridge, which is a Zigbee hub and a Matter bridge. This bridge exposes all Zigbee devices already connected to the bridge as Matter devices on the network. Also, Aqara, SwitchBot, and IKEA have launched such Hub devices.

<div class='note'>
Home Assistant, as a Matter controller, only supports **control** of Matter devices. Home Assistant is not a bridge itself and it cannot turn existing devices within Home Assistant into Matter compatible devices.
</div>

## Thread

Matter goes hand-in-hand with (but is not the same as) [Thread](<https://en.wikipedia.org/wiki/Thread_(network_protocol)>), which is a low power Radio mesh networking techology. Much like Zigbee, but with the key difference that it is _IP-addressable_, making it the perfect companion transport for Matter.

Thread devices become directly addressable by Matter controllers (such as Home Assistant) thanks to the use of so-called Thread Border Routers, which are in fact just devices that are both within your network and have a Thread chip builtin and thus act as a "router" between the Thread radio signal and your local network. These border routers (you will probably end up having multiple of them in your house) make sure that your Thread-based devices are reachable on your regular network and thus can be controlled with Matter. Examples of Thread Borders routers are the Apple TV 4K, HomePod (gen 2 or Mini), and the Google Nest Hub V2, so devices that you may already own. Besides that, all kind of other border routers are available, built-in to hardware appliances or software solutions based on OpenThread Border Router, such as the add-on we provide to use with the built-in Zigbee/Thread chip of the [Home Assistant Yellow](/yellow/) or the [Home Assistant SkyConnect](/skyconnect/) dongle.

To use any Thread-based devices on a Matter controller, you need to have at least one Thread Border router device within range of the device.
More info about Thread and diagnosing Thread networks and Border routers, see the [Thread](/integrations/thread/) integration.

<div class='note'>
Many devices that (will) hit the market will use Thread for radio communication and Matter as a control protocol, but this is not guaranteed. For example, Thread-based devices are available that only support Apple HomeKit or some vendor-specific communication protocol. There are also a few cases where you need to apply for a (beta) firmware update on the device to enable Matter as a communication protocol. Therefore, do not assume Matter support when you see a Thread logo when looking for devices. Please be sure to look for the *Matter* logo itself (on either Wi-Fi/Ethernet-based devices or Thread) or any other confirmation by the manufacturer that the device supports Matter.
</div>

## Bluetooth

Most (if not all) Matter-compliant devices will also have a Bluetooth chip onboard, this is to ease commissioning (a somewhat technical term for adding a device to your controller). Bluetooth will not be used to control a device but only to pair it after unboxing or factory resetting. The Home Assistant controller uses the Home Assistant Companion app to do commissioning, so you can bring your phone close to the device you want to commission. The controller will then send your network credentials to your device over Bluetooth in the commissioning process. If that succeeds, the device will communicate over its native interface, meaning Wi-Fi, Ethernet, or Thread.

<div class='note'>
Although your Home Assistant server might have a Bluetooth adapter on board that the controller can use to commission devices, we choose not to utilize that adapter. Mainly to prevent issues with the built-in Bluetooth integration but also because it makes more sense to bring your mobile devices close to the Matter device you'd like to commission.
</div>

## Multi fabric: join to multiple controllers

One of the great features of Matter is the so-called _Multi Fabric_ feature: you can join the same device to multiple controllers. For example, simultaneously add it to Google Home, Apple Home, and Home Assistant. The standard describes that each device should be able to at least support 5 different fabrics simultaneously.

For devices where Home Assistant provides a native integration (with local API), Matter may not be the best option. Matter, being a universal standard, might not have the nitty-gritty features that come with a product-specific protocol. A good example is Philips Hue: the communication over Matter only provides the basic controls over lights, while the official [Hue integration](/integrations/hue) brings all Hue unique features like (dynamic) scenes, entertainment mode, etc.

![image](/images/integrations/matter/matter_thread_infographic.webp)

Image taken from [this excellent article by The Verge](https://www.theverge.com/23165855/thread-smart-home-protocol-matter-apple-google-interview) about Matter that shows the landcape of Matter, Thread, Border routers and bridges in a nice visualized way.

{% include integrations/config_flow.md %}

For communicating with Matter devices, the Home Assistant integration runs its own "Matter controller" in a separate process which will be launched as an add-on. This add-on runs the controller software and connects your Matter network (called Fabric in technical terms) and Home Assistant. The Home Assistant Matter integration connects to this server via a WebSocket connection.

### Supported installation types

It is recommended to run the Matter add-on on Home Assistant OS. This is currently the best-supported option. 

If you run Home Assistant in a container, you can run a Docker image of the [Matter server](https://github.com/home-assistant-libs/python-matter-server). The requirements and instructions for your host setup are described on that GitHub page.

Running Matter on a Home Assistant Core installation is not supported.

## Adding Matter devices to Home Assistant

Each Matter network is called a fabric. Each home automation controller that controls Matter devices has its own "fabric". You can add devices directly to the fabric of your Home Assistant instance, or share them from another fabric (ie Google, Apple) to Home Assistant's fabric. We're going to explore all these options below.

### Add a device using the iOS Companion app

This will use the Bluetooth connection of your phone to add the device.

1. Open The Home Assistant app on your phone.
2. Go to {% my integrations title="**Settings** > **Devices & Services**" %}.
3. On the **Devices** tab, press the **Add device** button.
4. Choose **Add Matter device** at the top of the list.
5. Scan the QR-code of the Matter device with your phone camera or press **More options...** to manually enter the Commission code.
6. Select the **Add to Home Assistant** button which will start the commissioning process which may take up to a few minutes.
7. If you're adding a test board or beta device, you might get a prompt about an "Uncertified Accessory". In this dialog, select **Add Anyway**.
8. Once prompted, you can enter a custom **Accessory Name**, this is just an internal reference and not visible in Home Assistant. You can type whatever you like here.
9. Once the process is complete and you pressed the **Done** button, you are redirected to the device within Home Assistant. It is ready for use.

<lite-youtube videoid="8y79Kq3QfCQ" videotitle="Add Matter device via iOS app in Home Assistant"></lite-youtube>

### Add a device using the Android Companion app

This will use the Bluetooth connection of your phone to add the device.

1. Open The Home Assistant app on your phone.
2. Go to {% my integrations title="**Settings** > **Devices & Services**" %}.
3. On the **Devices** tab, press the **Add device** button.
4. Choose **Add Matter device** as the top of the list.
5. Scan the QR-code of the Matter device with your phones camera or select the **Setup without QR-code** button to manually enter the commission code.
6. The process will start adding the device which takes up to a few minutes.
7. If you're adding a test board (e.g. ESP32 running the example apps) and commissioning fails, you might need to take some actions in the Google Developer console, have a look at any instructions for your test device.
8. Once the process is complete and you pressed the **Done** button, you are redirected to the device within Home Assistant. It is ready for use.

<lite-youtube videoid="Fk0n0r0eKcE" videotitle="Add Matter device via Android app in Home Assistant"></lite-youtube>

### Share a device from Apple Home

This method will allow you to select a Matter device from Apple Home and share it to Home Assistant. The result is that the device can be controlled from both Apple Home and Home Assistant at the same time.

1.  Find your device in Apple Home and press the jogwheel to edit it. In the page with detailed descriptions and settings for the device, scroll all the way down and press the button **Turn On Pairing Mode**.
2.  You are now given a Setup code, copy this to the clipboard.
3.  Follow the [Add a device using the iOS Companion app](#add-a-device-using-the-ios-companion-app) directions above to add the device to Home Assistant where you paste the code you just received from Apple Home.

<lite-youtube videoid="nyGyZv90jnQ" videotitle="Share Matter device from Apple Home to Home Assistant"></lite-youtube>

### Share a device from Google Home

This method will allow you to share a device that was added to Google Home to Home Assistant. The result is that the device can be controlled from both Google Home and Home Assistant at the same time.

1.  Open the device in Google Home and press the settings button (jog wheel) in the top right.
2.  Click **Linked Matter apps and services**.
3.  Press the button **Link apps and services** to link the device to Home Assistant.
4.  Choose Home Assistant from the list, you are redirected to the Home Assistant Companion app now. Press **Add device**.
5.  Your device will now be added to Home Assistant. When the process finishes, you're redirected to the device page in Home Assistant.

<lite-youtube videoid="-B4WWevd2JI" videotitle="Share Matter device from Google Home to Home Assistant"></lite-youtube>


## Experiment with Matter using a ESP32 dev board

You do not yet have any Matter-compatible hardware but you do like to try it out or maybe create your own DIY Matter device? We have [prepared a page for you](https://nabucasa.github.io/matter-example-apps/) where you can easily flash Matter firmware to a supported ESP32 development board. We recommend the M5 Stamp C3 device running the Lighting app.

NOTE for Android users: You need to follow the instructions at the bottom of the page to add the test device to the Google developer console, otherwise commissioning will fail. iOS users will not have this issue but they will get a prompt during commissioning asking if you trust the development device.

1. Make sure you use Google Chrome or Microsoft Edge browser.
2. Open https://nabucasa.github.io/matter-example-apps/
3. Attach the ESP32 device using a USB cable.
4. Select the radio button next to the example you like to set up, in case of an M5 Stamp, click **Lighting app for M5STAMP C3**.
5. Select **Connect**.
6. In the popup dialog that appears, choose the correct serial device. This will usually be something like "cu-usbserial" or alike.
7. Click **Install Matter Lighting app example** and let it install the firmware on the device. This will take a few minutes.
8. Once the device is flashed with the Matter firmware, connect to the device again but this time choose **Logs & console**.
9. You are presented with a console interface where you see live logging of events. This is an interactive shell where you can type commands. For a list of all commands, type **matter help** and press enter.
10. To add the device, we need the QR code. In the console, type in `matter onboardingcodes ble` and copy/paste the URL into your browser.
11. Use the QR code to add the device using one of the above instructions on your phone, e.g. using the Home Assistant Companion app.

## Known working devices

Because the availability of actual Matter devices (and their documentation) is sparse, we provide a list of all known working devices that are tested by the Home Assistant Matter developers and/or the community to help you find the device you need. Note: The list below is ordered alphabetically, and some links contain affiliate links.

Did you test a device that is not listed below? It would be greatly appreciated if you share your experience either on the Matter discord channel or contribute a PR (or suggestion) to this documentation page so you can help others, thanks in advance!

### Aqara M2 Hub

- Bridges Aqara (Zigbee) devices connected to the hub to Matter.
- You need to enable Matter support/firmware in the Aqara app.
- You will need an Aqara (cloud) account and the app before you can use Matter.
- See [this Aqara landingpage](https://www.aqara.com/en/article-1583275073188196352.html) for more information, including what devices are bridged.
- Device events, for example for the wall rockers and Cube, are not supported.

### Eve Energy (power plug), Eve Door & Window (contact sensor), Eve Motion (motion sensor)

- If you see a Matter logo on the box, the device runs Matter already and you can add it to HA immediately.
- If there is a Thread logo, you need to install the Matter firmware using the Eve app.
- No Matter logo and no Thread logo on the box? The device is not Matter compatible.
- The energy metering feature of the Eve plug will not work in Home Assistant (Apple only feature).
- Eve has just released official Matter support so ignore any documentation about the beta program.

[Eve Energy on Amazon](https://amzn.to/3YuO62P)
[Eve Door & Window on Amazon](https://amzn.to/3RIU6ml)
[Eve Motion on Amazon](https://amzn.to/3jDujiP)

### Leviton Decora Smart Switches (D215S)

- You will need a My Leviton account and app, and must first connect the switch to the app through the cloud.
- [Upgrade to Matter-supported firmware]([https://nanoleaf.me/en-EU/integration/matter/](https://decorasmartsupport.leviton.com/hc/en-us/articles/11638969410331)
- If you want to break Leviton's connection to the switch (and the hole in your firewall), factory reset the swtich before pairing with Matter.
- Ensure IPv6 is enabled on your HA host, and your phone and HA host are on the same network/VLAN as the switch.
- The Matter pairing code for the switch is found in the My Leviton app.  There's also a QR code, but it's worthless with a single phone.
- Place the switch in pairing mode by holding the top for about 7 seconds until the LED turns amber.
- Doesn't always pair on the first attempt, but eventually works.

### Nanoleaf (Essentials) Matter bulbs and Lightstrips

- Although the products work great once commissioned, multiple users have reported that commissioning them can be a bit difficult and requires some patience and multiple resets or optimizations to your home network.
- Check the [Nanoleaf Matter infopage](https://nanoleaf.me/en-EU/integration/matter/) for all supported products and instructions.

### Philips Hue (V2) Bridge

The Philips Hue V2 bridge supports Matter. You can enable Matter support in the Hue app. You can then commission it to Home Assistant and other fabrics.

- Binding the Hue bridge to Home Assistant does not make sense because you will lose functionality over the default Hue integration in Home Assistant, such as button press events and (dynamic) scenes.
- You will need a Hue/Signify (cloud) account and the app before you can use Matter.
- Device events for example for dimmer remotes are not supported.
- Only basic control of lights is supported, no scenes, events, effects etc.

### SwitchBot Hub 2

SwitchBot has released a (beta) firmware update to enable Matter support on their Hub 2.  The SwitchBot Hub 2 is a Matter bridge device. It is bridging some of the devices, such as curtain motors, into Matter.

- To use Matter, in the SwitchBot app, enable Matter bridge support. Then, copy the code and use that to commission the Hub to Home Assistant. Another option is to use a second device to scan the QR code.
- Device support is limited. You bridge specific devices to Matter by adding them as **Secondary device** in the app. Note that not all SwitchBot devices can be bridged.
- Enabling Matter support does not convert the actual SwitchBot devices into matter devices. Those still need to be within the Bluetooth range of the hub.
- Bridged SwitchBot devices appear with a rather technical name in Home Assistant. This is a known issue.


### Tasmota

Tasmota supports Matter over IP on all ESP32 based devices (in experimental phase). Follow the [instructions](https://tasmota.github.io/docs/Matter/).

### TP-Link Tapo P125M (power plug)

- Look for the _M_ addition in the model name. A device without the M (regular P125) is not Matter compliant.
- This device is available in the US only.

[TP-Link Tapo P125M on Amazon](https://amzn.to/3RILJah)

## Troubleshooting

### General recommendations

- Using Thread-based Matter devices in Home Assistant requires Home Assistant OS version 10 and above. Not using Home Assistant OS is at your own risk. We do provide some [documentation](https://github.com/home-assistant-libs/python-matter-server/blob/main/README.md) on how to run the Matter Server as a Docker container. The documentation includes a description of the host and networking requirements. 

- To use Thread devices you will need a Thread Network with at least one Thread Border Router in your network nearby the Thread device(s). Apple users need for example the Apple TV 4K or the HomePod Mini, while Google users need a Nest Hub V2. Use the Thread integration in Home Assistant to diagnose your Thread network(s).

- Start simple and work from there, keep your network simple and add for example an ESP32 test device. Once that works, move on to the next step or more devices.

- Realize that you are an early adopter, both on the hardware side and on the software (controller) side so you may run into compatibility issues or features that are still missing. Report any issues you may find and help out others if you find a workaround or tested a device.

- Make sure IPv6 (multicast) traffic travels freely from your network to the Home Assistant host. There is no requirement to have an IPv6-enabled internet connection or DHCPv6 server. However, IPv6 support has to be enabled on Home Assistant. Go to **{% my network title="Settings > System > Network" %}**, and make sure **IPv6** is set to **Automatic** or **static**, depending on your network setup. If you're unsure, use **Automatic**.

- For more detailed information on network configuration, refer to the [README of the Matter server repository](https://github.com/home-assistant-libs/python-matter-server/blob/main/README.md).


### I do not see the button "Commission using the Companion app"

This button will only be visible within the Home Assistant Companion App (so not in the browser) and your device meets all requirements for Matter support.

- For iOS, minimum version is iOS 16 (minimal 16.3 is preferred) and the most recent version of the HA companion app.
- For Android, minimum version is 8.1 and the most recent version of the (full) HA Companion app, downloaded from the Play Store.

### When I'm trying to commission using the Android app, I get an error stating "Matter is currently unavailable"

See above, make sure your device meets all requirements to support Matter. Update Android to the latest version and the Home Assistant Companion app. To quickly verify if your device meets all requirements to support Matter, on your Android device, go to **Settings** > **Google** > **Devices & Sharing**. There should be an entry there for **Matter devices**.

Some users have reported that uninstalling and reinstalling the Google Home app fixed this issue for them.
Also see this [extended troubleshooting guide](https://developers.home.google.com/matter/verify-services) from Google.

### Unable to commission devices, it keeps giving errors or stops working randomly

The Matter protocol relies on (local) IPv6 and mDNS (multicast traffic) which should be able to travel freely in your network. Matter devices that use Wi-Fi (including Thread Border routers) must be on the same LAN/VLAN as Home Assistant. Matter devices that only use Thread must be joined to Thread networks for which there is at least one border router connected to the Home Assistant LAN.

If you experience any issues with discovering devices (for example, if the initial commission keeps failing or if devices become unavailable randomly), investigate your network topology. For instance, a setting on your router or Wi-Fi access point to "optimize" multicast traffic can harm the (discovery) traffic from Matter devices. Keep this in mind when you experience issues trying to add or control Matter devices. Protocols like Matter are designed for regular residential network setups and may not integrate well with enterprise networking solutions like VLANs, Multicast filtering, and (malfunctioning) IGMP snooping. To avoid issues, try to keep your network topology as simple and flat as possible.
