# ansible-roles

Custom ansible roles tailored to Fedora workstation and Ubuntu server

## Description

This repo is mainly used to deploy Fedora workstation based laptops. I am using it with following distros:

- Fedora
- Fedora silverblue

Additionally there are also some roles that are for ubuntu server I am using.

## Usage

Add this repo as a submodule in repo where you want to use these roles:

```shell
git submodule add git@github.com:xtruder/ansible-roles.git roles
```

Example ansible playbook for fedora workstation:

```yaml
---
- hosts: workstations
  become_user: "root"
  gather_facts: yes
  vars:
    pkg_upgrade: False
    enable_secure_boot: True

    firefox_addons:
    - darkreader
    - window-titler

    firefox_profiles:
      main:
        name: Firefox
        desktop: org.mozilla.Firefox.desktop
        addons:
        - adblock-plus
        - decentraleyes
        - privacy-badger17
        - vertical-tabs-reloaded
        options:
          hide_tabs: true
  
  roles:
    - role: alzadude.firefox-addon
    - role: setup
      tags: ["always"]
    - role: fedora
      tags: ["fedora"]
      when: ansible_facts['distribution'] == "Fedora"
    - role: jason_riddle.tailscale
      tags: ["tailscale"]
      become: yes
    - role: systemd-resolved
      tags: ["networking", "systemd-resolved"]
    - role: gnome
      tags: ["gnome"]
    - role: flatpak
      tags: ["flatpak"]
    - role: chromium
      tags: ["chromium"]
    - role: firefox
      tags: ["firefox"]

  tasks:
    - name: Install packages
      package:
        name:
          - powertop
      become: true
 
    - name: Install flatpak packages
      community.general.flatpak:
        remote: flathub
        name: 
          # audio/video
          - org.gnome.Cheese
          - org.videolan.VLC

          # tools
          - com.github.tchx84.Flatseal
          - org.gnome.Extensions
          - org.gnome.PowerStats


- hosts: work
  become_user: "root"
  gather_facts: no
  tags: ["work"]
  vars:
    incus_profile_user: offlinehq
  roles:
    - role: printing
      tags: ["printing"]
    - role: usbguard
      tags: ["usbguard"]
      vars:
        usbguard_gnome_integration: True
    - role: docker
      tags: ["docker"]
    - role: qemu
      tags: ["qemu"]
    - role: incus
      tags: ["incus"]
    - role: libvirt
      tags: ["libvirt"]
    - role: 1password
      tags: ["1password"]
    - role: vscode
      tags: ["vscode"]
    - role: gpg
      tags: ["gpg"]
  tasks:
  - name: Install work packages
    package:
      name:
        # yubikey support
        - pam_yubico
        - libykneomgr
        - pcsc-lite
        - pcsc-tools

        # filesystem
        - davfs2 # nextcloud mounts

        # encryption
        - fscrypt
        - sirikali
        - qtpass
        - pass
        - pass-otp

        # containers
        - podman-compose
        #- podman-docker

        # iso
        - livecd-tools
        - mediawriter

        # android
        - android-tools # needed for adb
        - waydroid

        # debugging
        - bcc
      state: present
    become: yes

  - name: Install work flatpak packages
    flatpak:
      remote: flathub
      name:
        # networking
        - org.wireshark.Wireshark

        # social/communication
        - com.slack.Slack
        - us.zoom.Zoom
        - com.microsoft.Teams

        # office
        - org.onlyoffice.desktopeditors
        - com.github.jeromerobert.pdfarranger
        - com.github.xournalpp.xournalpp

        # knowledge managment
        - com.logseq.Logseq
        - md.obsidian.Obsidian

        # remote desktop
        - org.remmina.Remmina

        # graphics
        - com.github.maoschanz.drawing
        - org.inkscape.Inkscape
        - org.gimp.GIMP

        # cloud
        - com.nextcloud.desktopclient.nextcloud

        # crypto
        - org.gnome.seahorse.Application

        # download
        - com.transmissionbt.Transmission
      state: present
      method: user
```
