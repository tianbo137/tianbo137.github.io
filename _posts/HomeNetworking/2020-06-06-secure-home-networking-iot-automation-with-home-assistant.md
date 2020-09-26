---
layout: post
title: 'Secure Home Networking: IoT Automation with Home Assistant'
date: '2020-06-06 11:13'
tags:
  - Networking
---

This post introduces [Home Assistant](https://www.home-assistant.io/), an open-source home IoT automation that puts local control and privacy first. it also helps address many IoT platform by providing a cross-vendor management interface.

I by no means have a large collection of home automation or IoT toys. But between Apple HomeKit, Amazon Alexa, and Google Assistant, it is sometimes annoying to find something works on one platform but not so much on the other. In addition, automation services such as IFTTT often have limitations imposed by vendors on what can be controlled by automation. Therefore, I start to use Home Assistant as the main IoT management and automation system.

# Install with Docker and Docker Compose
Since I have an Intel NUC running Ubuntu server which is also used in my [Pi-Hole]({{ site.baseurl }}{% link _posts/HomeNetworking/2019-09-14-secure-home-network-block-ad-with-pi-hole.md %}) project, I install Home Assistant using [Docker and Docker Compose](https://www.home-assistant.io/docs/installation/docker/#docker-compose):

```yaml
version: '3'
services:
  homeassistant:
    container_name: home-assistant
    image: homeassistant/home-assistant:stable
    volumes:
      - /home/ubuntu/home-assitant/config:/config
    environment:
      - TZ=America/Chicago
    restart: always
    network_mode: host
```

Then start the container with:

```bash
docker-compose up -d
```

To restart Home Assistant when you have changed configuration:

```bash
docker-compose restart
```

Note, to automatically start Docker on boot:
```bash
sudo systemctl enable docker
```

The Home Assistant container is configured to restart automatically when docker engine starts up, so there is no need to start up on boot.

# Check Configuration
Home Assistant UI is useful but sometimes limited. Sometimes its configuration file, located in `/home/ubuntu/home-assitant/config`, must be manually edited. The following command can verify the integrity of the configuration files after editing.

```bash
docker exec home-assistant python -m homeassistant --script check_config --config /config
```

Then restart Home Assistant to use the updated configuration.

# Upgrade Home Assistant
Use `docker-compose pull` to get the latest docker image, and restart after `pull` is complete.

```bash
docker-compose pull
docker-compose restart
```

# Configure IoT Device
Home Assistant is developed and maintained by community. While many devices can be configured through its web UI, many still requires configurations through its YAML files. For example, I have many [Yeelight](https://www.home-assistant.io/integrations/yeelight/) bulbs, but it must be configured through YAML. It requires configuring static IPs for light bulbs:

```yaml
yeelight:
  devices:
    192.168.20.5:
      name: Bedroom Color
      model: color2
    192.168.20.6:
      name: Floor Lamp
    192.168.20.7:
      name: Hallway
    192.168.20.8:
      name: Second Room
    192.168.20.9:
      name: Lamp
      model: lamp1
```

Always validate the configuration before restarting your Home Assistant!

Home Assistant is limited by what vendors allow for third-party integration. For example, Google no longer allows new third-party integration with [Nest](https://www.home-assistant.io/integrations/nest/) devices, and you cannot configure Home Assistant to control it if you don't have a developer account with Nest before the cut-off date.

# Further Reads
This is the post series. Other posts on the home network topics are:
1. [Device and Management Setup]({{ site.baseurl }}{% link _posts/HomeNetworking/2019-09-07-secure-home-network-device-and-management-setup.md %})
1. [Isolating Connected Devices with VLAN]({{ site.baseurl }}{% link _posts/HomeNetworking/2019-09-02-secure-home-network-isolating-connected-devices-with-vlan.md %})
1. [Using HomeKit Devices Across VLANs]({{ site.baseurl }}{% link _posts/HomeNetworking/2019-08-27-secure-home-network-using-homekit-devices-across-vlans.md %})
1. [Using AirPlay Across VLANs]({{ site.baseurl }}{% link _posts/HomeNetworking/2019-08-31-secure-home-network-using-airplay-across-vlans.md %})
1. [Extend WiFi Coverage with Multiple APs]({{ site.baseurl }}{% link _posts/HomeNetworking/2020-01-11-secure-home-network-extend-wifi-coverage-with-multiple-aps.md %})
1. [Backup Your Configurations]({{ site.baseurl }}{% link _posts/HomeNetworking/2019-11-23-secure-home-network-backup-your-configurations.md %})
1. [Block Ad and Tracking with Pi-Hole]({{ site.baseurl }}{% link _posts/HomeNetworking/2019-09-14-secure-home-network-block-ad-with-pi-hole.md %})
1. [Troubleshoot DHCP Problems]({{ site.baseurl }}{% link _posts/HomeNetworking/2020-01-11-secure-home-network-troubleshoot-dhcp-problems.md %})