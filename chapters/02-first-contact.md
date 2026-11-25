# Chapter 2: First Contact

Northwind's three servers are running, and you are sitting at the control node. Before Ansible can do anything useful, it needs three things: a way to log in to every server without a password, a list of the servers, and a first command to prove it all works. This chapter sets up all three and runs your first **ad-hoc commands**, one-off commands sent to many servers at once.

> **Try it:** DockerLab, *Ansible lab 1, First contact*. Topology: control `10.0.0.10`, web1 `10.0.0.11`, web2 `10.0.0.12`, db1 `10.0.0.21`. Work in control's console.

## Meet the control node

Ansible is not one program but a family of commands:

| Command | Use |
|---|---|
| `ansible` | run one module on many hosts: **ad-hoc commands** |
| `ansible-playbook` | run a playbook (Chapter 5) |
| `ansible-doc` | read a module's documentation, with examples |
| `ansible-inventory` | show an inventory as Ansible understands it |
| `ansible-config` | show the configuration and where each setting came from |
| `ansible-vault` | encrypt secrets (Chapter 11) |
| `ansible-galaxy` | create roles, install roles and collections (Chapter 10) |

```
ansible --version
ansible-doc -l | wc -l
```

`ansible --version` prints the version of ansible-core, the configuration file in use (none yet), and the Python it runs on. `ansible-doc -l` lists the **modules** available, several thousand with the collections included.

## Can control reach the servers?

Ansible needs only two things from the network: the managed node's name or address, and its SSH port, TCP 22.

```
for h in web1 web2 db1; do ping -c1 -W1 "$h" > /dev/null && echo "$h answers"; done
for h in web1 web2 db1; do timeout 2 bash -c "echo > /dev/tcp/$h/22" && echo "$h: SSH is listening"; done
```

DockerLab puts every container's name in `/etc/hosts`, so `web1` resolves to `10.0.0.11`. On a real network, names come from DNS, or the inventory gives each host's address (Chapter 4).

## Key-based login

Ansible logs in to every host many times per run, so typing passwords is out. It uses **SSH key pairs**: a private key that stays on control, and a public key copied to each server's `~/.ssh/authorized_keys`. A server lets in anyone who proves they hold the private key matching one of its authorized public keys.

<figure class="diagram">
<svg viewBox="0 0 720 316" style="width:100%;height:auto;max-width:720px" role="img" aria-label="SSH key-based login. On control, ssh-keygen makes a key pair: the private key id_ed25519 stays on control; the public key id_ed25519.pub is copied by ssh-copy-id into authorized_keys on web1, web2 and db1. When Ansible connects, the server checks that control holds the private key matching an authorized public key, and lets it in without a password." xmlns="http://www.w3.org/2000/svg">
<style>
.a2a{ font-family:var(--sans),sans-serif; }
.a2a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a2a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a2a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a2a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a2a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a2a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a2a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a2a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a2a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a2a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a2a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a2a .s{ fill:var(--text-dim); font-size:13px; }
.a2a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a2a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a2a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a2a .c{ fill:var(--text-faint); font-size:12.5px; }
.a2a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a2a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a2a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a2a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2a-a); }
.a2a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2a-a); marker-start:url(#a2a-s); }
.a2a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a2a-g); }
.a2a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a2a-a); }
.a2a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a2a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a2a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a2a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a2a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a2a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2a-tri); }
.a2a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2a-tri); }
.a2a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2a-open); }
.a2a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2a-open); }
.a2a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2a-dia); }
.a2a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2a-odia); }
.a2a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2a-dia); marker-end:url(#a2a-open); }
.a2a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2a-solid); }
.a2a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a2a-open); }
.a2a .fill{ fill:var(--text); stroke:none; }
.a2a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a2a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a2a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a2a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a2a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a2a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a2a .ks-blue{ fill:#185FA5; font-size:13px; }
.a2a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a2a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a2a .kb-blue, :root[data-theme="dark"] .a2a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a2a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a2a .kh-blue{ fill:#85B7EB; }
.a2a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a2a-blue); }
.a2a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2a-blue); }
.a2a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a2a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a2a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a2a .ks-purple{ fill:#534AB7; font-size:13px; }
.a2a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a2a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a2a .kb-purple, :root[data-theme="dark"] .a2a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a2a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a2a .kh-purple{ fill:#AFA9EC; }
.a2a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a2a-purple); }
.a2a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2a-purple); }
.a2a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a2a .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a2a .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a2a .ks-green{ fill:#3B6D11; font-size:13px; }
.a2a .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a2a .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2a .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a2a .kb-green, :root[data-theme="dark"] .a2a .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a2a .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a2a .kh-green{ fill:#97C459; }
.a2a .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a2a-green); }
.a2a .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2a-green); }
.a2a .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a2a .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a2a .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a2a .ks-red{ fill:#A32D2D; font-size:13px; }
.a2a .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a2a .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2a .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a2a .kb-red, :root[data-theme="dark"] .a2a .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a2a .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a2a .kh-red{ fill:#F09595; }
.a2a .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a2a-red); }
.a2a .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2a-red); }
.a2a .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a2a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a2a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a2a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a2a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a2a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a2a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a2a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a2a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a2a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a2a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a2a-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a2a-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a2a">
<rect x="20" y="20" width="250" height="260" rx="10" class="k-purple"/>
<text x="145" y="44" text-anchor="middle" class="kb-purple">control</text>
<rect x="40" y="64" width="210" height="50" rx="7" class="k-red"/>
<text x="145.0" y="84.5" text-anchor="middle" class="km-red">id_ed25519</text>
<text x="145.0" y="102.5" text-anchor="middle" class="ks-red">private key: never leaves</text>
<rect x="40" y="130" width="210" height="50" rx="7" class="k-green"/>
<text x="145.0" y="150.5" text-anchor="middle" class="km-green">id_ed25519.pub</text>
<text x="145.0" y="168.5" text-anchor="middle" class="ks-green">public key: safe to share</text>
<rect x="40" y="200" width="210" height="60" rx="7" class="pnl"/>
<text x="145.0" y="225.5" text-anchor="middle" class="m">ssh-keygen -t ed25519</text>
<text x="145.0" y="243.5" text-anchor="middle" class="c">makes the pair, once</text>
<rect x="450" y="20" width="250" height="76" rx="10" class="k-blue"/>
<text x="475" y="44" text-anchor="start" class="kb-blue">web1</text>
<rect x="560" y="30" width="128" height="56" rx="7" class="k-green"/>
<text x="624.0" y="54.0" text-anchor="middle" class="km-green">~/.ssh/</text>
<text x="624.0" y="71.0" text-anchor="middle" class="km-green">authorized_keys</text>
<path d="M252 155 C 350 155, 380 58, 556 58" class="ar-green"/>
<rect x="450" y="110" width="250" height="76" rx="10" class="k-blue"/>
<text x="475" y="134" text-anchor="start" class="kb-blue">web2</text>
<rect x="560" y="120" width="128" height="56" rx="7" class="k-green"/>
<text x="624.0" y="144.0" text-anchor="middle" class="km-green">~/.ssh/</text>
<text x="624.0" y="161.0" text-anchor="middle" class="km-green">authorized_keys</text>
<path d="M252 155 C 350 155, 380 148, 556 148" class="ar-green"/>
<rect x="450" y="200" width="250" height="76" rx="10" class="k-blue"/>
<text x="475" y="224" text-anchor="start" class="kb-blue">db1</text>
<rect x="560" y="210" width="128" height="56" rx="7" class="k-green"/>
<text x="624.0" y="234.0" text-anchor="middle" class="km-green">~/.ssh/</text>
<text x="624.0" y="251.0" text-anchor="middle" class="km-green">authorized_keys</text>
<path d="M252 155 C 350 155, 380 238, 556 238" class="ar-green"/>
<text x="360" y="138" text-anchor="middle" class="kh-green">ssh-copy-id</text>
<text x="360" y="304" text-anchor="middle" class="s">then: ssh web1 hostname  works with no password</text>
</g>
</svg>
<figcaption>The private key stays on control; each server keeps a copy of the public key.</figcaption>
</figure>

```
ssh-keygen -t ed25519 -N '' -f ~/.ssh/id_ed25519
ssh-copy-id student@web1
ssh-copy-id student@web2
ssh-copy-id student@db1
ssh web1 hostname
```

`ssh-copy-id` asks you to trust each server's **host key** the first time (type `yes`) and for the password `student`, once. The last line logs in without a password: key login works. In every later lab, the control image's helper script `setup-keys web1 web2 db1` does these steps in one go.

> `-N ''` makes a key without a passphrase, convenient for a lab. On real systems, protect the key with a passphrase and load it into `ssh-agent`, or use a dedicated automation account whose key is stored in a secrets manager.

## A first inventory

The **inventory** lists the machines Ansible manages, sorted into **groups**. The simplest format is INI:

```
mkdir -p ~/northwind && cd ~/northwind
cat > inventory <<'EOF'
[web]
web1
web2

[db]
db1
EOF
ansible all -i inventory --list-hosts
ansible web -i inventory --list-hosts
```

`[web]` starts a group; the lines below it are its hosts. Two groups always exist without being written: **all**, every host, and **ungrouped**, hosts in no other group. `-i inventory` says which inventory to use; Chapter 4 makes it the default.

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="An INI inventory file with groups web (web1, web2) and db (db1), and the group tree Ansible builds from it: all contains ungrouped, web and db; web contains web1 and web2; db contains db1." xmlns="http://www.w3.org/2000/svg">
<style>
.a2b{ font-family:var(--sans),sans-serif; }
.a2b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a2b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a2b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a2b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a2b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a2b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a2b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a2b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a2b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a2b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a2b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a2b .s{ fill:var(--text-dim); font-size:13px; }
.a2b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a2b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a2b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a2b .c{ fill:var(--text-faint); font-size:12.5px; }
.a2b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a2b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a2b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a2b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2b-a); }
.a2b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2b-a); marker-start:url(#a2b-s); }
.a2b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a2b-g); }
.a2b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a2b-a); }
.a2b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a2b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a2b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a2b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a2b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a2b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2b-tri); }
.a2b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2b-tri); }
.a2b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2b-open); }
.a2b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2b-open); }
.a2b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2b-dia); }
.a2b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2b-odia); }
.a2b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2b-dia); marker-end:url(#a2b-open); }
.a2b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2b-solid); }
.a2b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a2b-open); }
.a2b .fill{ fill:var(--text); stroke:none; }
.a2b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a2b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a2b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a2b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a2b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a2b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a2b .ks-blue{ fill:#185FA5; font-size:13px; }
.a2b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a2b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a2b .kb-blue, :root[data-theme="dark"] .a2b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a2b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a2b .kh-blue{ fill:#85B7EB; }
.a2b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a2b-blue); }
.a2b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2b-blue); }
.a2b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a2b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a2b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a2b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a2b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a2b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a2b .kb-teal, :root[data-theme="dark"] .a2b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a2b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a2b .kh-teal{ fill:#5DCAA5; }
.a2b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a2b-teal); }
.a2b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2b-teal); }
.a2b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a2b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a2b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a2b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a2b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a2b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a2b .kb-gray, :root[data-theme="dark"] .a2b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a2b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a2b .kh-gray{ fill:#B4B2A9; }
.a2b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a2b-gray); }
.a2b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2b-gray); }
.a2b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a2b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a2b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a2b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a2b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a2b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a2b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a2b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a2b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a2b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a2b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a2b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a2b">
<rect x="20" y="20" width="200" height="146" rx="6" class="k-teal"/>
<text x="30" y="40" text-anchor="start" class="kh-teal">inventory</text>
<text x="32.0" y="62" text-anchor="start" class="m">[web]</text>
<text x="32.0" y="80" text-anchor="start" class="m">web1</text>
<text x="32.0" y="98" text-anchor="start" class="m">web2</text>
<text x="32.0" y="116" text-anchor="start" class="m"></text>
<text x="32.0" y="134" text-anchor="start" class="m">[db]</text>
<text x="32.0" y="152" text-anchor="start" class="m">db1</text>
<path d="M232 110 L282 110" class="ar"/>
<rect x="300" y="20" width="110" height="40" rx="7" class="k-gray"/>
<text x="355.0" y="44.5" text-anchor="middle" class="kb-gray">all</text>
<path d="M355 60 L355 96" class="ag"/>
<rect x="300" y="100" width="110" height="40" rx="7" class="k-gray"/>
<text x="355.0" y="124.5" text-anchor="middle" class="kb-gray">ungrouped</text>
<path d="M355 60 L495 96" class="ag"/>
<rect x="440" y="100" width="110" height="40" rx="7" class="k-teal"/>
<text x="495.0" y="124.5" text-anchor="middle" class="kb-teal">web</text>
<path d="M495 140 L460 184" class="ag"/>
<rect x="425" y="188" width="70" height="36" rx="7" class="k-blue"/>
<text x="460.0" y="210.5" text-anchor="middle" class="kb-blue">web1</text>
<path d="M495 140 L530 184" class="ag"/>
<rect x="495" y="188" width="70" height="36" rx="7" class="k-blue"/>
<text x="530.0" y="210.5" text-anchor="middle" class="kb-blue">web2</text>
<path d="M355 60 L635 96" class="ag"/>
<rect x="580" y="100" width="110" height="40" rx="7" class="k-teal"/>
<text x="635.0" y="124.5" text-anchor="middle" class="kb-teal">db</text>
<path d="M635 140 L600 184" class="ag"/>
<rect x="565" y="188" width="70" height="36" rx="7" class="k-blue"/>
<text x="600.0" y="210.5" text-anchor="middle" class="kb-blue">db1</text>
<text x="355" y="238" text-anchor="middle" class="c">(empty here)</text>
</g>
</svg>
<figcaption>Two groups written; all and ungrouped come free.</figcaption>
</figure>

## Ping them all

```
ansible all -i inventory -m ping
```

```
web2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.13"
    },
    "changed": false,
    "ping": "pong"
}
...
```

`-m ping` runs the **ping module** on every host in *all*. It is not an ICMP ping. Ansible logs in with SSH, runs a small Python program, and gets back `pong`. **SUCCESS** therefore proves that the name resolves, SSH works, the key is accepted, and Python runs. That is everything Ansible needs. The hosts answer in any order: Ansible works on several at once.

## Anatomy of an ad-hoc command

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="An ad-hoc command taken apart: ansible, then the host pattern web, then -i inventory naming the inventory file, then -m command naming the module, then -a &#x27;df -h /&#x27; giving the module&#x27;s arguments, then -b to become root." xmlns="http://www.w3.org/2000/svg">
<style>
.a2c{ font-family:var(--sans),sans-serif; }
.a2c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a2c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a2c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a2c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a2c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a2c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a2c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a2c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a2c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a2c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a2c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a2c .s{ fill:var(--text-dim); font-size:13px; }
.a2c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a2c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .c{ fill:var(--text-faint); font-size:12.5px; }
.a2c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a2c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a2c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a2c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2c-a); }
.a2c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2c-a); marker-start:url(#a2c-s); }
.a2c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a2c-g); }
.a2c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-a); }
.a2c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a2c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a2c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a2c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a2c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a2c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2c-tri); }
.a2c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-tri); }
.a2c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2c-open); }
.a2c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-open); }
.a2c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2c-dia); }
.a2c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2c-odia); }
.a2c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2c-dia); marker-end:url(#a2c-open); }
.a2c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2c-solid); }
.a2c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a2c-open); }
.a2c .fill{ fill:var(--text); stroke:none; }
.a2c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a2c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a2c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a2c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a2c .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a2c .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a2c .ks-teal{ fill:#0F6E56; font-size:13px; }
.a2c .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2c .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a2c .kb-teal, :root[data-theme="dark"] .a2c .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a2c .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a2c .kh-teal{ fill:#5DCAA5; }
.a2c .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a2c-teal); }
.a2c .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-teal); }
.a2c .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a2c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a2c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a2c .ks-amber{ fill:#854F0B; font-size:13px; }
.a2c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a2c .kb-amber, :root[data-theme="dark"] .a2c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a2c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a2c .kh-amber{ fill:#EF9F27; }
.a2c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a2c-amber); }
.a2c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-amber); }
.a2c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a2c .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a2c .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a2c .ks-coral{ fill:#993C1D; font-size:13px; }
.a2c .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2c .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a2c .kb-coral, :root[data-theme="dark"] .a2c .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a2c .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a2c .kh-coral{ fill:#F0997B; }
.a2c .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a2c-coral); }
.a2c .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-coral); }
.a2c .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a2c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a2c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a2c .ks-purple{ fill:#534AB7; font-size:13px; }
.a2c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a2c .kb-purple, :root[data-theme="dark"] .a2c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a2c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a2c .kh-purple{ fill:#AFA9EC; }
.a2c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a2c-purple); }
.a2c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-purple); }
.a2c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a2c .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a2c .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a2c .ks-green{ fill:#3B6D11; font-size:13px; }
.a2c .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2c .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a2c .kb-green, :root[data-theme="dark"] .a2c .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a2c .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a2c .kh-green{ fill:#97C459; }
.a2c .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a2c-green); }
.a2c .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-green); }
.a2c .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a2c .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a2c .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a2c .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a2c .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a2c .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2c .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a2c .kb-gray, :root[data-theme="dark"] .a2c .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a2c .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a2c .kh-gray{ fill:#B4B2A9; }
.a2c .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a2c-gray); }
.a2c .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2c-gray); }
.a2c .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a2c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a2c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a2c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a2c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a2c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a2c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a2c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a2c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a2c-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a2c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a2c-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a2c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a2c-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a2c-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a2c">
<rect x="20" y="50" width="90" height="44" rx="5" class="k-gray"/>
<text x="65.0" y="76.5" text-anchor="middle" class="km-gray">ansible</text>
<line x1="65.0" y1="98" x2="65.0" y2="130" class="ln-gray"/>
<text x="65.0" y="150" text-anchor="middle" class="kh-gray">the command</text>
<rect x="120" y="50" width="70" height="44" rx="5" class="k-teal"/>
<text x="155.0" y="76.5" text-anchor="middle" class="km-teal">web</text>
<line x1="155.0" y1="98" x2="155.0" y2="130" class="ln-teal"/>
<text x="155.0" y="150" text-anchor="middle" class="kh-teal">host pattern</text>
<rect x="200" y="50" width="130" height="44" rx="5" class="k-purple"/>
<text x="265.0" y="76.5" text-anchor="middle" class="km-purple">-i inventory</text>
<line x1="265.0" y1="98" x2="265.0" y2="130" class="ln-purple"/>
<text x="265.0" y="150" text-anchor="middle" class="kh-purple">which inventory</text>
<rect x="340" y="50" width="120" height="44" rx="5" class="k-green"/>
<text x="400.0" y="76.5" text-anchor="middle" class="km-green">-m command</text>
<line x1="400.0" y1="98" x2="400.0" y2="130" class="ln-green"/>
<text x="400.0" y="150" text-anchor="middle" class="kh-green">which module</text>
<rect x="470" y="50" width="130" height="44" rx="5" class="k-amber"/>
<text x="535.0" y="76.5" text-anchor="middle" class="km-amber">-a 'df -h /'</text>
<line x1="535.0" y1="98" x2="535.0" y2="130" class="ln-amber"/>
<text x="535.0" y="150" text-anchor="middle" class="kh-amber">its arguments</text>
<rect x="610" y="50" width="50" height="44" rx="5" class="k-coral"/>
<text x="635.0" y="76.5" text-anchor="middle" class="km-coral">-b</text>
<line x1="635.0" y1="98" x2="635.0" y2="130" class="ln-coral"/>
<text x="635.0" y="150" text-anchor="middle" class="kh-coral">as root</text>
<text x="360" y="26" text-anchor="middle" class="s">ansible  PATTERN  [-i INVENTORY]  [-m MODULE]  [-a ARGS]  [options]</text>
<text x="360" y="190" text-anchor="middle" class="c">-m defaults to command; other useful options: --limit, -f (forks), -v ... -vvvv, -K, -C (check)</text>
</g>
</svg>
<figcaption>Every ad-hoc command has the same shape.</figcaption>
</figure>

```
ansible all -i inventory -a uptime
ansible web -i inventory -a 'df -h /'
ansible db1 -i inventory -a 'cat /etc/os-release'
```

Without `-m`, the module is **command**, which runs a program on each host. The first word after `ansible` is a **host pattern**: a group, a host, or *all*. Each result says **CHANGED**, because the `command` module cannot know whether the program it ran changed anything. Real modules can, as Chapter 3 shows.

## Host patterns

Patterns choose hosts from the inventory, combining groups like sets:

| Pattern | Means | Here |
|---|---|---|
| `all` or `*` | every host | web1, web2, db1 |
| `web` | a group | web1, web2 |
| `web:db` | in web **or** db (union) | web1, web2, db1 |
| `all:!db` | all **except** db | web1, web2 |
| `web:&production` | in web **and** production (intersection) | Chapter 4 |
| `web*` | names matching a wildcard | web1, web2 |
| `~web[0-9]` | names matching a regular expression | web1, web2 |

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Host patterns as set operations on the groups web and db and a third group production. web:db is the union, web:&amp;production the intersection, all:!db everything except db." xmlns="http://www.w3.org/2000/svg">
<style>
.a2d{ font-family:var(--sans),sans-serif; }
.a2d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a2d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a2d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a2d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a2d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a2d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a2d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a2d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a2d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a2d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a2d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a2d .s{ fill:var(--text-dim); font-size:13px; }
.a2d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a2d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a2d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a2d .c{ fill:var(--text-faint); font-size:12.5px; }
.a2d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a2d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a2d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a2d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2d-a); }
.a2d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2d-a); marker-start:url(#a2d-s); }
.a2d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a2d-g); }
.a2d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a2d-a); }
.a2d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a2d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a2d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a2d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a2d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a2d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2d-tri); }
.a2d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2d-tri); }
.a2d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2d-open); }
.a2d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2d-open); }
.a2d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2d-dia); }
.a2d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2d-odia); }
.a2d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2d-dia); marker-end:url(#a2d-open); }
.a2d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2d-solid); }
.a2d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a2d-open); }
.a2d .fill{ fill:var(--text); stroke:none; }
.a2d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a2d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a2d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a2d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a2d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a2d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a2d .ks-blue{ fill:#185FA5; font-size:13px; }
.a2d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a2d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a2d .kb-blue, :root[data-theme="dark"] .a2d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a2d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a2d .kh-blue{ fill:#85B7EB; }
.a2d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a2d-blue); }
.a2d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2d-blue); }
.a2d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a2d .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a2d .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a2d .ks-teal{ fill:#0F6E56; font-size:13px; }
.a2d .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a2d .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2d .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a2d .kb-teal, :root[data-theme="dark"] .a2d .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a2d .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a2d .kh-teal{ fill:#5DCAA5; }
.a2d .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a2d-teal); }
.a2d .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2d-teal); }
.a2d .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a2d .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a2d .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a2d .ks-amber{ fill:#854F0B; font-size:13px; }
.a2d .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a2d .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2d .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a2d .kb-amber, :root[data-theme="dark"] .a2d .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a2d .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a2d .kh-amber{ fill:#EF9F27; }
.a2d .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a2d-amber); }
.a2d .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2d-amber); }
.a2d .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
</style>
<defs><marker id="a2d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a2d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a2d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a2d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a2d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a2d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a2d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a2d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a2d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a2d-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a2d-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker></defs>
<g class="a2d">
<text x="125" y="26" text-anchor="middle" class="ma">web:db</text>
<text x="125" y="46" text-anchor="middle" class="c">union</text>
<circle cx="93" cy="140" r="58" class="k-blue"/>
<circle cx="157" cy="140" r="58" class="k-blue"/>
<text x="70" y="136" text-anchor="middle" class="kb-blue">web</text>
<text x="67" y="154" text-anchor="middle" class="ks-blue">web1</text>
<text x="67" y="170" text-anchor="middle" class="ks-blue">web2</text>
<text x="175" y="136" text-anchor="middle" class="kb-blue">db</text>
<text x="175" y="154" text-anchor="middle" class="ks-blue">db1</text>
<text x="125" y="232" text-anchor="middle" class="s">web1, web2, db1</text>
<text x="360" y="26" text-anchor="middle" class="ma">web:&amp;production</text>
<text x="360" y="46" text-anchor="middle" class="c">intersection</text>
<circle cx="330" cy="140" r="62" class="k-teal" style="opacity:.35"/>
<circle cx="390" cy="140" r="62" class="k-amber" style="opacity:.35"/>
<path d="M360 88 A62 62 0 0 1 360 192 A62 62 0 0 1 360 88 Z" class="k-blue"/>
<text x="302" y="140" text-anchor="middle" class="kb-teal">web</text>
<text x="418" y="140" text-anchor="middle" class="kb-amber">prod.</text>
<text x="360" y="145" text-anchor="middle" class="kb-blue">web1</text>
<text x="360" y="232" text-anchor="middle" class="s">web1 (Chapter 4)</text>
<text x="595" y="26" text-anchor="middle" class="ma">all:!db</text>
<text x="595" y="46" text-anchor="middle" class="c">exclusion</text>
<rect x="495" y="80" width="200" height="120" rx="10" class="k-blue"/>
<circle cx="647" cy="140" r="38" class="pnl"/>
<text x="647" y="145" text-anchor="middle" class="c">db</text>
<text x="557" y="136" text-anchor="middle" class="kb-blue">all</text>
<text x="557" y="154" text-anchor="middle" class="ks-blue">web1 web2</text>
<text x="595" y="232" text-anchor="middle" class="s">web1, web2</text>
</g>
</svg>
<figcaption>Patterns combine groups: : for or, :&amp; for and, :! for not.</figcaption>
</figure>

```
ansible 'web:db' -i inventory --list-hosts
ansible 'all:!db' -i inventory --list-hosts
ansible all -i inventory --limit web2 -a hostname
```

Quote patterns containing `!`, `&` or `*` so the shell leaves them alone. `--limit` narrows any pattern further: the usual way to try something on one server before all of them.

## Several hosts at once: forks

Ansible runs each task on up to **5 hosts in parallel** by default, its **forks**. With three hosts, all three run at once; with fifty, they go in batches of five.

<figure class="diagram">
<svg viewBox="0 0 720 246" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Forks control parallelism. With forks 1 the task runs on web1, then web2, then db1, taking nine seconds for a three-second command. With forks 3 all three run at once and finish in three seconds." xmlns="http://www.w3.org/2000/svg">
<style>
.a2e{ font-family:var(--sans),sans-serif; }
.a2e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a2e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a2e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a2e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a2e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a2e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a2e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a2e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a2e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a2e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a2e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a2e .s{ fill:var(--text-dim); font-size:13px; }
.a2e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a2e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a2e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a2e .c{ fill:var(--text-faint); font-size:12.5px; }
.a2e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a2e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a2e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a2e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2e-a); }
.a2e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2e-a); marker-start:url(#a2e-s); }
.a2e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a2e-g); }
.a2e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a2e-a); }
.a2e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a2e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a2e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a2e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a2e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a2e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2e-tri); }
.a2e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2e-tri); }
.a2e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2e-open); }
.a2e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2e-open); }
.a2e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2e-dia); }
.a2e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2e-odia); }
.a2e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2e-dia); marker-end:url(#a2e-open); }
.a2e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2e-solid); }
.a2e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a2e-open); }
.a2e .fill{ fill:var(--text); stroke:none; }
.a2e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a2e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a2e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a2e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a2e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a2e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a2e .ks-blue{ fill:#185FA5; font-size:13px; }
.a2e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a2e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a2e .kb-blue, :root[data-theme="dark"] .a2e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a2e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a2e .kh-blue{ fill:#85B7EB; }
.a2e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a2e-blue); }
.a2e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2e-blue); }
.a2e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a2e .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a2e .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a2e .ks-amber{ fill:#854F0B; font-size:13px; }
.a2e .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a2e .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2e .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a2e .kb-amber, :root[data-theme="dark"] .a2e .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a2e .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a2e .kh-amber{ fill:#EF9F27; }
.a2e .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a2e-amber); }
.a2e .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2e-amber); }
.a2e .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
</style>
<defs><marker id="a2e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a2e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a2e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a2e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a2e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a2e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a2e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a2e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a2e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a2e-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker></defs>
<g class="a2e">
<text x="40" y="70" text-anchor="middle" class="ma">-f 1</text>
<rect x="90" y="30" width="180" height="20" rx="3" class="k-blue"/>
<text x="180" y="44.5" text-anchor="middle" class="kb-blue">web1</text>
<rect x="270" y="54" width="180" height="20" rx="3" class="k-blue"/>
<text x="360" y="68.5" text-anchor="middle" class="kb-blue">web2</text>
<rect x="450" y="78" width="180" height="20" rx="3" class="k-amber"/>
<text x="540" y="92.5" text-anchor="middle" class="kb-amber">db1</text>
<text x="642" y="70" text-anchor="start" class="a">9 s</text>
<text x="40" y="170" text-anchor="middle" class="ma">-f 3</text>
<rect x="90" y="130" width="180" height="20" rx="3" class="k-blue"/>
<text x="180" y="144.5" text-anchor="middle" class="kb-blue">web1</text>
<rect x="90" y="154" width="180" height="20" rx="3" class="k-blue"/>
<text x="180" y="168.5" text-anchor="middle" class="kb-blue">web2</text>
<rect x="90" y="178" width="180" height="20" rx="3" class="k-amber"/>
<text x="180" y="192.5" text-anchor="middle" class="kb-amber">db1</text>
<text x="282" y="170" text-anchor="start" class="a">3 s</text>
<line x1="90" y1="214" x2="90" y2="208" class="ln"/>
<text x="90" y="236" text-anchor="middle" class="c">0</text>
<line x1="150" y1="214" x2="150" y2="208" class="ln"/>
<line x1="210" y1="214" x2="210" y2="208" class="ln"/>
<line x1="270" y1="214" x2="270" y2="208" class="ln"/>
<text x="270" y="236" text-anchor="middle" class="c">3</text>
<line x1="330" y1="214" x2="330" y2="208" class="ln"/>
<line x1="390" y1="214" x2="390" y2="208" class="ln"/>
<line x1="450" y1="214" x2="450" y2="208" class="ln"/>
<text x="450" y="236" text-anchor="middle" class="c">6</text>
<line x1="510" y1="214" x2="510" y2="208" class="ln"/>
<line x1="570" y1="214" x2="570" y2="208" class="ln"/>
<line x1="630" y1="214" x2="630" y2="208" class="ln"/>
<text x="630" y="236" text-anchor="middle" class="c">9</text>
<line x1="90" y1="214" x2="630" y2="214" class="ln"/>
</g>
</svg>
<figcaption>Forks: how many hosts work at the same time (default 5).</figcaption>
</figure>

```
time ansible all -i inventory -a 'sleep 3' -f 1
time ansible all -i inventory -a 'sleep 3' -f 3
```

With `-f 1` the three hosts take about nine seconds, one after another; with `-f 3`, about three. Chapter 4 sets `forks` in the configuration file. More forks mean faster runs on big fleets, at the cost of more load on the control node.

## Facts

The **setup** module gathers **facts**: hundreds of details about each host, from its operating system and CPUs to its memory, disks and network interfaces.

```
ansible web1 -i inventory -m setup | head -20
ansible all -i inventory -m setup -a 'filter=ansible_distribution*'
ansible db1 -i inventory -m setup -a 'filter=ansible_eth0'
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The setup module returns facts about a host, grouped as a tree: ansible_distribution and version, processor_vcpus, memtotal_mb, interfaces such as eth0 with its ipv4 address and netmask, default_ipv4, hostname, python_version and many more." xmlns="http://www.w3.org/2000/svg">
<style>
.a2f{ font-family:var(--sans),sans-serif; }
.a2f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a2f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a2f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a2f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a2f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a2f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a2f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a2f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a2f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a2f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a2f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a2f .s{ fill:var(--text-dim); font-size:13px; }
.a2f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a2f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a2f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a2f .c{ fill:var(--text-faint); font-size:12.5px; }
.a2f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a2f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a2f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a2f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2f-a); }
.a2f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2f-a); marker-start:url(#a2f-s); }
.a2f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a2f-g); }
.a2f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a2f-a); }
.a2f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a2f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a2f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a2f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a2f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a2f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2f-tri); }
.a2f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2f-tri); }
.a2f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2f-open); }
.a2f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2f-open); }
.a2f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2f-dia); }
.a2f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2f-odia); }
.a2f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2f-dia); marker-end:url(#a2f-open); }
.a2f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2f-solid); }
.a2f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a2f-open); }
.a2f .fill{ fill:var(--text); stroke:none; }
.a2f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a2f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a2f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a2f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a2f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a2f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a2f .ks-blue{ fill:#185FA5; font-size:13px; }
.a2f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a2f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a2f .kb-blue, :root[data-theme="dark"] .a2f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a2f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a2f .kh-blue{ fill:#85B7EB; }
.a2f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a2f-blue); }
.a2f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2f-blue); }
.a2f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a2f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a2f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a2f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a2f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a2f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a2f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a2f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a2f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a2f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker></defs>
<g class="a2f">
<rect x="20" y="100" width="130" height="54" rx="7" class="k-blue"/>
<line x1="30" y1="109" x2="46" y2="109" class="ln-blue"/>
<line x1="30" y1="114" x2="46" y2="114" class="ln-blue"/>
<text x="93.0" y="122" text-anchor="middle" class="kb-blue">web1</text>
<text x="85.0" y="141" text-anchor="middle" class="ks-blue">setup module</text>
<path d="M152 127 L200 127" class="ar"/>
<rect x="210" y="16" width="490" height="206" rx="6" class="k-blue"/>
<text x="222.0" y="36" text-anchor="start" class="m">"ansible_facts": {</text>
<text x="237.2" y="55" text-anchor="start" class="m">"distribution": "Debian",</text>
<text x="237.2" y="74" text-anchor="start" class="m">"distribution_major_version": "13",</text>
<text x="237.2" y="93" text-anchor="start" class="m">"hostname": "web1",  "os_family": "Debian",</text>
<text x="237.2" y="112" text-anchor="start" class="m">"processor_vcpus": 2,  "memtotal_mb": 7944,</text>
<text x="237.2" y="131" text-anchor="start" class="m">"eth0": { "ipv4": { "address": "10.0.0.11",</text>
<text x="374.0" y="150" text-anchor="start" class="m">"netmask": "255.255.255.0" } },</text>
<text x="237.2" y="169" text-anchor="start" class="m">"python_version": "3.13.5",</text>
<text x="237.2" y="188" text-anchor="start" class="m">... several hundred more</text>
<text x="222.0" y="207" text-anchor="start" class="m">}</text>
</g>
</svg>
<figcaption>Facts describe each host; playbooks read them as ansible_facts['...'].</figcaption>
</figure>

Playbooks gather facts automatically at the start of each play, and use them to adapt: install with `apt` on Debian but `dnf` on Rocky Linux, write each server's own address into a configuration file, size a cache from the memory. Chapter 6 uses them in earnest.

## Becoming root

Ansible logs in as `student`. Most system changes need root, so Ansible can **become** another user, root by default, through `sudo`:

```
ansible all -i inventory -a whoami
ansible db1 -i inventory -a 'head -1 /etc/shadow'
ansible db1 -i inventory --become -a 'head -1 /etc/shadow'
ansible all -i inventory -b -a whoami
```

The second command fails, *Permission denied*; with `--become` (short: `-b`) the same command runs as root. On the lab servers, student may use `sudo` without a password. Elsewhere, add `-K` (`--ask-become-pass`) and Ansible asks for the sudo password once.

<figure class="diagram">
<svg viewBox="0 0 720 200" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Become: Ansible logs in to db1 as student over SSH, then runs the module through sudo as root. Without become, reading /etc/shadow is denied; with -b it succeeds." xmlns="http://www.w3.org/2000/svg">
<style>
.a2g{ font-family:var(--sans),sans-serif; }
.a2g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a2g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a2g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a2g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a2g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a2g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a2g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a2g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a2g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a2g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a2g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a2g .s{ fill:var(--text-dim); font-size:13px; }
.a2g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a2g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a2g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a2g .c{ fill:var(--text-faint); font-size:12.5px; }
.a2g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a2g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a2g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a2g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2g-a); }
.a2g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a2g-a); marker-start:url(#a2g-s); }
.a2g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a2g-g); }
.a2g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a2g-a); }
.a2g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a2g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a2g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a2g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a2g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a2g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2g-tri); }
.a2g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2g-tri); }
.a2g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2g-open); }
.a2g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a2g-open); }
.a2g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2g-dia); }
.a2g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2g-odia); }
.a2g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a2g-dia); marker-end:url(#a2g-open); }
.a2g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a2g-solid); }
.a2g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a2g-open); }
.a2g .fill{ fill:var(--text); stroke:none; }
.a2g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a2g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a2g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a2g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a2g .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a2g .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a2g .ks-blue{ fill:#185FA5; font-size:13px; }
.a2g .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a2g .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2g .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a2g .kb-blue, :root[data-theme="dark"] .a2g .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a2g .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a2g .kh-blue{ fill:#85B7EB; }
.a2g .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a2g-blue); }
.a2g .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2g-blue); }
.a2g .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a2g .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a2g .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a2g .ks-coral{ fill:#993C1D; font-size:13px; }
.a2g .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a2g .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2g .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a2g .kb-coral, :root[data-theme="dark"] .a2g .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a2g .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a2g .kh-coral{ fill:#F0997B; }
.a2g .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a2g-coral); }
.a2g .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2g-coral); }
.a2g .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a2g .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a2g .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a2g .ks-green{ fill:#3B6D11; font-size:13px; }
.a2g .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a2g .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2g .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a2g .kb-green, :root[data-theme="dark"] .a2g .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a2g .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a2g .kh-green{ fill:#97C459; }
.a2g .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a2g-green); }
.a2g .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2g-green); }
.a2g .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a2g .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a2g .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a2g .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a2g .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a2g .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a2g .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a2g .kb-gray, :root[data-theme="dark"] .a2g .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a2g .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a2g .kh-gray{ fill:#B4B2A9; }
.a2g .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a2g-gray); }
.a2g .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a2g-gray); }
.a2g .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a2g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a2g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a2g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a2g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a2g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a2g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a2g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a2g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a2g-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a2g-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a2g-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a2g-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a2g">
<rect x="10" y="70" width="110" height="54" rx="7" class="dark"/>
<text x="65.0" y="93" text-anchor="middle" class="on">control</text>
<text x="65.0" y="111" text-anchor="middle" class="on" style="fill:var(--bg);font-weight:400">ansible -b</text>
<path d="M122 97 L182 97" class="ar"/>
<text x="152" y="88" text-anchor="middle" class="a">SSH</text>
<rect x="186" y="70" width="120" height="54" rx="7" class="k-blue"/>
<text x="246.0" y="92.5" text-anchor="middle" class="kb-blue">student</text>
<text x="246.0" y="110.5" text-anchor="middle" class="ks-blue">logs in</text>
<path d="M308 97 L368 97" class="ar-coral"/>
<text x="338" y="88" text-anchor="middle" class="kh-coral">sudo</text>
<rect x="372" y="70" width="130" height="54" rx="7" class="k-coral"/>
<text x="437.0" y="92.5" text-anchor="middle" class="kb-coral">root</text>
<text x="437.0" y="110.5" text-anchor="middle" class="ks-coral">runs the module</text>
<rect x="540" y="24" width="160" height="56" rx="7" class="k-gray"/>
<text x="620.0" y="47.5" text-anchor="middle" class="kb-gray">without -b</text>
<text x="620.0" y="65.5" text-anchor="middle" class="ks-gray">Permission denied</text>
<rect x="540" y="104" width="160" height="56" rx="7" class="k-green"/>
<text x="620.0" y="127.5" text-anchor="middle" class="kb-green">with -b</text>
<text x="620.0" y="145.5" text-anchor="middle" class="km-green">root:*:...</text>
<path d="M504 90 L536 60" class="ag"/>
<path d="M504 104 L536 128" class="ag"/>
<text x="360" y="186" text-anchor="middle" class="c">become_user (default root), become_method (default sudo), -K to type the sudo password</text>
</g>
</svg>
<figcaption>Log in as an ordinary user; become root only where needed.</figcaption>
</figure>

Log in as an ordinary user and become root only for the tasks that need it. Logging in directly as root is usually disabled on hardened servers (Chapter 14 does that for Northwind).

## What Ansible actually does

`-v` makes Ansible more talkative; `-vvv` shows every connection and command:

```
ansible web1 -i inventory -m ping -vvv 2>&1 | grep -oE 'ESTABLISH SSH CONNECTION|PUT |AnsiballZ_ping\.py|rm -f -r' | sort | uniq -c
```

```
      4 AnsiballZ_ping.py
      6 ESTABLISH SSH CONNECTION
      1 PUT
      1 rm -f -r
```

Read the whole output without the `grep` to see the story of Chapter 1: SSH to the host, make a temporary folder, `PUT` (copy) the module packed as `AnsiballZ_ping.py`, run it with the host's Python, read its JSON answer, and remove the folder. The six "connections" reuse one SSH connection: Ansible keeps it open between steps (OpenSSH's *ControlPersist*), which is why later tasks are faster than the first.

## Hands on

1. Which fact gives the number of CPUs? Find it with `-m setup -a 'filter=*vcpu*'`.
2. Stop db1 in DockerLab and run `ansible all -i inventory -m ping`. What does **UNREACHABLE** look like, and do the other hosts still answer? Start db1 again.
3. Write host patterns for: the web servers except web2; every host whose name ends in `1`.
4. Capture web1's cable in DockerLab during `ansible web1 -i inventory -m ping`. How many TCP connections to port 22 do you see?

## Summary

- Ansible needs **key-based SSH login** to every managed node: a key pair on control, the public key in each node's `authorized_keys`.
- The **inventory** lists hosts in groups; `all` and `ungrouped` always exist.
- `ansible PATTERN -m MODULE -a ARGS` runs one module on many hosts: an **ad-hoc command**. The default module is `command`.
- `ping` proves the whole chain: name, SSH, key, Python.
- **Host patterns** select hosts like sets (`:` union, `:&` intersection, `:!` exclusion); `--limit` narrows a run.
- **Forks** set how many hosts run in parallel (5 by default).
- The `setup` module gathers **facts**; `-b` (**become**) runs a module as root through sudo.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-2-first-contact).

**1. Why does Ansible need key-based SSH login?**

- A. Passwords are not supported by SSH
- B. Keys make modules run faster
- C. Keys install Python
- D. It logs in to every host many times per run, so typing passwords is impractical

**2. Which file on a managed node lists the public keys allowed to log in?**

- A. ~/.ssh/id_ed25519
- B. ~/.ssh/authorized_keys
- C. ~/.ssh/known_hosts
- D. /etc/ssh/sshd_config

**3. Which groups exist in every inventory, even if not written?**

- A. web and db
- B. control and managed
- C. production and staging
- D. all and ungrouped

**4. What does a SUCCESS from the ping module prove?**

- A. Name resolution, SSH, the key and Python on the host all work
- B. ICMP reaches the host
- C. The host has internet access
- D. The host runs a web server

**5. Without -m, which module does the ansible command use?**

- A. shell
- B. ping
- C. command
- D. raw

**6. Which pattern selects every host except those in db?**

- A. db:!all
- B. all:&db
- C. !db:all
- D. all:!db

**7. What do forks control?**

- A. How many tasks run per host
- B. How many hosts Ansible works on in parallel (default 5)
- C. How many SSH keys are used
- D. How many retries a task gets

**8. What does -b (--become) do?**

- A. Runs the command in the background
- B. Skips the inventory
- C. Runs the module as another user, root by default, through sudo
- D. Runs on control only
