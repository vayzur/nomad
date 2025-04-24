# Nomad

Nomad is a lightweight automation tool focused solely on **P2P Tunnel Setup**. It simplifies and automates the process of establishing peer-to-peer VPN tunnels, ensuring a seamless and low-latency connection between remote endpoints.

---

### Supported Methods:
- **Tunneling between 2 servers:**
  - EasyTier
  - SIT (6to4)
- **Xray Core**

---

## Tunneling

### EasyTier

**Heads up!** Tunneling needs two servers—one in the internal network and one in the external network. Let’s set up EasyTier with the reverse tunneling method, which we recommend.

#### Step 1: Create the Inventory File

Tell Ansible where to work by creating the `hosts.yml` file:

```yml
all:
  vars:
    ansible_port: 3022
    ansible_user: root

  hosts:
    wormhole:
      ansible_host: <your_internal_server_ip>
    stargate:
      ansible_host: <your_external_server_ip>
```

#### Step 3: Update the Config Files

Edit the following files to match your requirements:

```bash
<your-favorite-editor> inventory/group_vars/all/*.yml
```

#### Step 4: Enable EasyTier

Set `enable_easytier` to `true` in `all.yml`:

```yml
## Set to true to enable the EasyTier service
enable_easytier: true
```

By default tunneling method is reversed in `easytier.yml`
```yml
easytier_reverse: true
```

Then configure EasyTier in `easytier.yml`. For example:

```yml
easytier_multithread: true
```

For optimal performance, keep multithreading enabled (it reduces ping by spreading tasks across multiple threads).

#### Step 5: Set a Network Secret

Generate a new secret using:

```bash
openssl rand -hex 6
```

Replace the default secret with your custom one in `easytier.yml`.

#### Step 6: Run the Playbook

You’re ready! Execute this command:

```bash
ansible-playbook -i inventory/hosts.yml tunnel.yml
```

Verify the setup:

```bash
systemctl status easytier
```

```bash
ip addr
```
Expected output:

```bash
3: easytier: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1380 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.44.44.1/24 scope global easytier
       valid_lft forever preferred_lft forever
```

---

## Xray Core

> [!WARNING]
> Xray is supported only on x86\_64 architectures.&#x20;

For installing Xray Core:

1. Set `enable_xray` to `true` in `all.yml`:

```yml
enable_xray: true
```

2. Use the default config or replace it with your custom Xray config:

> [!WARNING]
> Config file for internal server must be: `wormhole.json` for external server: `stargate.json`
```bash
cp /path/internal.json roles/xray/files/wormhole.json
cp /path/external.json roles/xray/files/stargate.json
```

Run the playbook:

```bash
ansible-playbook -i inventory/hosts.yml xray.yml
```

Verify:

```bash
systemctl status xray
```

```bash
ss -tulpn
```

---

# Credits

- [EasyTier](https://github.com/EasyTier/EasyTier)
- [Xray](https://github.com/XTLS/Xray-core)
