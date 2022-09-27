# Homelab
## What's this ?

This is a setup guide + ansible script that sets up my homelab.

<details>
<summary>All services available after setup</summary>
URLs listed below expects the hostname to be `homelab.lan`, if you changed it those URLs will have to be updated too.

- DNS server (dnsmasq)
    - Be sure to setup your router to advertise it as its sole DNS server (both IPv4 and IPv6)
- Samba auto-mount
    - Will mount automatically some samba shares (the one for Jellyfin and the one for Paperless)
- [DOCKER] nginx
    - [https://homelab.lan/](https://homelab.lan/)
    - The web server that is used to reverse-proxy everything
- [DOCKER] Jellyfin
    - [https://play.homelab.lan/](https://play.homelab.lan/)
    - A home-hosted Netflix for your movie / series collection
- [DOCKER] Paperless-ngx
    - [https://paper.homelab.lan/](https://paper.homelab.lan/)
    - A document library for everything you receive by mail and other important document.
- [DOCKER] Gitea
    - [https://git.homelab.lan/](https://git.homelab.lan/)
    - A private git
- [DOCKER] Home Assistant
    - [https://domo.homelab.lan/](https://domo.homelab.lan/)
    - IoT for everything in the home
    - Stuff related to Home Assistant:
    - Companions:
        - [DOCKER] Node-red
            - [https://nr.homelab.lan](https://nr.homelab.lan)
            - Node-red lets you automate stuff using a node "language", mainly used as a Home Assistant companion 
        - [DOCKER] Mosquitto
            - [homelab.lan:1883](homelab.lan:1883)
            - Mosquitto is a MQTT server mainly used in Home Assistant, it's exposed for you if you have the need for it for other things than HASS
        - [DOCKER] Zigbee2MQTT
            - [https://z2m.homelab.lan](https://z2m.homelab.lan)
            - Zigbee2MQTT is a gateway server that takes all your Zigbee devices and bridge them to Home Assistant / Node-red through the MQTT server
</details>

## Usage

1. Install Debian 11 on your computer (@TODO: Create an unattended script?)
  Setup infos:
  - 1. Language / keymap does not matter, use what you want
  - 2. Machine name: homelab (or the same as you have in inventory.yaml)
  - 3. No domain
  - 4. Root password does not matter, use what you want
  - 5. User: oxodao (or the same as you have in inventory.yaml)
  - 6. Use full drive
  - 7. Softwares:
    - Debian desktop environment
    - ... GNOME
    - SSH server
    - Usual system tools
  - 8. Grub on main SSD

2. Reboot once the setup is over, find however you want the IP of your homelab server and on your main computer copy your SSH public key to the authorized hosts:
```sh
$ ssh-copy-id [username]@192.168.1.159
```

3. Add the missing files

The first required missing files are the CA key, for this take a look at the [guide to generate yours](docs/be_your_own_ca.md) until you've trusted it on your computer, for this guide there is no need to generate the certificates manually as this will be handled by ansible. Once it's created, backup it somewhere safe !

The files I've called `oxo-ca.pem` and `oxo-ca.key` should be put in the `files/ssl/` folder.

You can edit the `inventories.yaml` file to setup your machines IP but also to edit the variables you want to use.

Finally, you need to copy `variables.yaml.dist` to `variables.yaml` and fill them accordingly.

`inventories.yaml` and `variables.yaml` are the only files that should be edited !

4. Running the initial ansible

The initial script will ensure that the port is on the new one and that the `oxodao` user has root permissions passwordless.

This is a separate playbook as it's not idempotent and has to be run only once, and will not work later. Plus it requires the original root password.

This could be resolved once we build a custom ISO.

```sh
$ ansible-playbook -i inventories.yaml --limit homelab-vm initial_setup.yaml -b --ask-become-pass
```

4. Running the ansible !

This is the step that does the magic.

```sh
$ ansible-galaxy install -r requirements.yaml
$ ansible-playbook -i inventories.yaml --limit homelab-vm setup.yaml
```

5. Reboot and have fun !

Reboot is optional as everything should be handled correctly in the ansible setup, but you're pretty much done, your homelab is ready.

6. (Optional) Restore from backup

Let's say you are using this because you just reinstalled your homelab, let's see how to restore from the restic backups that are done regularly.

@TODO
