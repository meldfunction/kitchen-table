# Mesh Network — Architecture and Config Push

## What the mesh needs to do

1. Connect all nodes privately (encrypted, no public IPs required)
2. Allow services on one node to be reachable from any other node
3. Allow config updates to be pushed to all nodes simultaneously
4. Work across different ISPs, NAT types, and home routers without port forwarding

---

## Option 1: Tailscale (recommended for most groups)

Tailscale uses WireGuard under the hood with a key management server (Tailscale's
own infrastructure). This means you trust Tailscale for key management but not for
your traffic — all data is encrypted end-to-end between your nodes.

**Free tier:** 100 devices, unlimited bandwidth, 3 users (add more with a free account
per person — each person controls their own devices).

### Network topology for a friend group

Each person's Pi joins the Tailscale network. Tailscale assigns each Pi a stable IP
(in the 100.x.x.x range) and a DNS hostname. Services (NextCloud, Navidrome, OpenClaw)
are accessed via these hostnames.

```
alice-node.tail12345.ts.net  →  100.64.0.1  (Alice's Pi, gateway host)
bob-node.tail12345.ts.net    →  100.64.0.2  (Bob's Pi)
charlie-node.tail12345.ts.net →  100.64.0.3  (Charlie's Pi)
casaos-host.tail12345.ts.net  →  100.64.0.4  (shared services host)
```

### Access controls (Tailscale ACLs)

Restrict which nodes can reach which services. Example: only nodes in the group
can reach the OpenClaw gateway; Navidrome and NextCloud are reachable from mobile
devices too.

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["group:friends"],
      "dst": ["tag:openclaw-gateway:18789"]
    },
    {
      "action": "accept",
      "src": ["group:friends"],
      "dst": ["tag:services:4533,8080,4533"]
    }
  ],
  "groups": {
    "group:friends": [
      "alice@example.com",
      "bob@example.com",
      "charlie@example.com"
    ]
  }
}
```

### Subnet routing (expose local LAN through mesh)

If one person's Pi should act as an exit node or expose local LAN services:

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24 --accept-routes
# Then approve the route in Tailscale admin panel
```

---

## Option 2: Yggdrasil (fully decentralized, no central authority)

Yggdrasil is a mesh networking protocol that doesn't depend on any central
infrastructure. No Tailscale servers involved. Your nodes find each other directly.

**When to use Yggdrasil instead of Tailscale:**
- Your group doesn't want to trust any external key management
- You want the mesh to survive even if Tailscale (the company) disappears
- You're building something meant to be fully decentralized

**Tradeoff:** More configuration, less polished tooling, harder to debug.

```bash
# Install on each Pi
sudo apt install yggdrasil

# Generate config
yggdrasil -genconf | sudo tee /etc/yggdrasil/yggdrasil.conf

# Edit config to add known peers (other nodes' public keys or public Yggdrasil peers)
sudo nano /etc/yggdrasil/yggdrasil.conf

# Start
sudo systemctl enable yggdrasil
sudo systemctl start yggdrasil

# Get your IPv6 address (Yggdrasil uses IPv6)
sudo yggdrasilctl getSelf
```

Each node gets a stable globally-routable IPv6 address derived from its public key.
No IP assignment service needed — the address is cryptographically determined.

---

## Config push with Ansible

Once your group has 3+ nodes, managing them individually becomes painful. Ansible
lets you push changes to all nodes simultaneously with one command.

### Setup (on any one machine — not the Pis)

```bash
pip install ansible

# SSH key setup (do this once)
ssh-keygen -t ed25519 -f ~/.ssh/openclaw-net-key
# Copy public key to each Pi:
ssh-copy-id -i ~/.ssh/openclaw-net-key pi@alice-node.tail12345.ts.net
ssh-copy-id -i ~/.ssh/openclaw-net-key pi@bob-node.tail12345.ts.net
```

### Inventory file

```ini
# inventory.ini
[nodes]
alice-node.tail12345.ts.net
bob-node.tail12345.ts.net
charlie-node.tail12345.ts.net

[services]
casaos-host.tail12345.ts.net

[all:vars]
ansible_user=pi
ansible_ssh_private_key_file=~/.ssh/openclaw-net-key
```

### Common operations

```bash
# Push OpenClaw config update to all nodes
ansible nodes -i inventory.ini -m copy \
  -a "src=config/openclaw-config.json dest=~/.openclaw/config.json"

# Restart OpenClaw on all nodes
ansible nodes -i inventory.ini -m shell -a "openclaw restart"

# Pull new model on all nodes simultaneously
ansible nodes -i inventory.ini -m shell \
  -a "ollama pull phi3.5:3.8b-mini-instruct-q4_K_M"

# Run openclaw doctor on all nodes, collect results
ansible nodes -i inventory.ini -m shell -a "openclaw doctor 2>&1" \
  --one-line

# Update all Pis simultaneously
ansible nodes -i inventory.ini -m shell \
  -a "sudo apt update && sudo apt upgrade -y"

# Check which Ollama models are loaded on each node
ansible nodes -i inventory.ini -m shell -a "ollama list"
```

### Playbook for new member onboarding

When a new person joins the group, run one playbook that sets up their Pi completely:

```yaml
# onboard-new-member.yml
---
- name: Onboard new group member
  hosts: "{{ target_node }}"
  become: yes
  tasks:
    - name: Install core dependencies
      apt:
        name: [git, curl, build-essential, cmake, python3, python3-pip, nodejs, docker.io]
        state: present
        update_cache: yes

    - name: Install Ollama
      shell: curl -fsSL https://ollama.com/install.sh | sh

    - name: Pull assistant model
      shell: ollama pull phi3.5:3.8b-mini-instruct-q4_K_M
      become_user: pi

    - name: Pull background model
      shell: ollama pull llama3.2:1b-instruct-q8_0
      become_user: pi

    - name: Install OpenClaw
      shell: npm install -g openclaw
      become_user: pi

    - name: Install Tailscale
      shell: curl -fsSL https://tailscale.com/install.sh | sh

    - name: Copy OpenClaw base config
      copy:
        src: config/openclaw-base-config.json
        dest: /home/pi/.openclaw/config.json
        owner: pi

    - name: Run openclaw doctor
      shell: openclaw doctor
      become_user: pi
      register: doctor_output

    - name: Show doctor results
      debug:
        msg: "{{ doctor_output.stdout }}"
```

Run with:
```bash
ansible-playbook -i inventory.ini onboard-new-member.yml \
  --extra-vars "target_node=new-member-node.tail12345.ts.net"
```

---

## Hardware mesh profiles

The group can maintain a shared repository of hardware configuration profiles
that Ansible pushes to nodes. This is how you ensure every Pi in the group has
consistent settings.

```
group-config/
├── inventory.ini
├── playbooks/
│   ├── onboard.yml
│   ├── update-all.yml
│   └── push-openclaw-config.yml
├── config/
│   ├── openclaw-base-config.json
│   ├── ollama-config.json
│   └── tailscale-acls.json
└── templates/
    ├── soul-md-template.md
    └── openclaw-config.json.j2
```

Store this in a private Gitea repository. When you make a config change, commit it
and run the relevant playbook. Every node in the group stays synchronized.

This is the "mesh network profiles to push to hardware" approach — infrastructure
as code for a friend group's distributed compute cluster.
