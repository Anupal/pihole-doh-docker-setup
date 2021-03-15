# PiHole DoH setup Setup

I used following files to setup a pihole and cloudflared docker containers on my Ubuntu 20.04 LTS VM. The PiHole forwards DNS requests to Cloudflared server over a TLS tunnel.
These files are based on excellent tutorial posted by [Bipul](https://github.com/bipulkkuri) on this [link](https://github.com/bipulkkuri/pihole/tree/master/DOH).

You'll need following prerequisites before moving forward:
- docker
- docker-compose

## Setup
- Clone the repo and execute `docker-compose up -d`
- This brings up PiHole and Cloudflared containers as well as a 10.0.0.0/29 network between them.
- The PiHole admin console is accessible at http://127.0.0.1/admin (password: admin)
- Modify local machine settings to switch to use PiHole as DNS server.
  - Change the `/etc/resolv.conf` symlink from `../run/systemd/resolve/stub-resolv.conf` to `../run/systemd/resolve/resolv.conf`. 
  ```
  cd /etc/
  ln -sfn ../run/systemd/resolve/resolv.conf
  ```
  - Edit the netplan config (mine was at `/etc/netplan/00-installer-config.yaml`) and add 127.0.0.1 before other nameservers.
  ```
  # This is the network config written by 'subiquity'
  network:
    ethernets:
      enp9s0:
        dhcp4: true
        nameservers:
          addresses: [127.0.0.1,8.8.8.8]
    version: 2
  ```
  - Apply the updated network config using `sudo netplan apply`.

## Know issues and fixes

### TCP/UDP ports reserved by systemd-resolved
```
ERROR: for pihole_pihole_1 Cannot start service pihole: driver failed programming external connectivity on endpoint pihole_pihole_1 (f1742d41f7119f673122910b19b1b68eea7f4d329a8f3d037ef026df69a2bdd2): Error starting userland proxy: listen tcp4 0.0.0.0:53: bind: address already in use
```

**Stop and disable the service.** 
```
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

