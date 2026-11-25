# Chapter 4: Inventory and Configuration

Northwind's two web servers are not quite twins. `web1` is the production site customers see; `web2` is where new versions are tried first, the **staging** server. Both are web servers, and every server is a Linux machine that needs the same baseline. One list of hosts has to express all of that: what each server does, which environment it belongs to, and what settings each needs. This chapter turns `~/northwind` into a proper Ansible **project**, with a configuration file, a richer inventory, and variables organized in folders.

> **Try it:** DockerLab, *Ansible lab 3, Inventory and configuration*. Same topology. The finished files are in `examples/ch04`.

## ansible.cfg: the project's settings

Ansible reads **one** configuration file: the first it finds in this order.

<figure class="diagram">
<svg viewBox="0 0 720 190" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Ansible looks for its configuration in this order and uses only the first it finds: the file named by the ANSIBLE_CONFIG environment variable, ansible.cfg in the current folder, .ansible.cfg in the home folder, and /etc/ansible/ansible.cfg." xmlns="http://www.w3.org/2000/svg">
<style>
.a4a{ font-family:var(--sans),sans-serif; }
.a4a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a4a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a4a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a4a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a4a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a4a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a4a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a4a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a4a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a4a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a4a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a4a .s{ fill:var(--text-dim); font-size:13px; }
.a4a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a4a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a4a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a4a .c{ fill:var(--text-faint); font-size:12.5px; }
.a4a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a4a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a4a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a4a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4a-a); }
.a4a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4a-a); marker-start:url(#a4a-s); }
.a4a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a4a-g); }
.a4a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a4a-a); }
.a4a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a4a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a4a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a4a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a4a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a4a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4a-tri); }
.a4a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4a-tri); }
.a4a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4a-open); }
.a4a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4a-open); }
.a4a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4a-dia); }
.a4a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4a-odia); }
.a4a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4a-dia); marker-end:url(#a4a-open); }
.a4a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4a-solid); }
.a4a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a4a-open); }
.a4a .fill{ fill:var(--text); stroke:none; }
.a4a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a4a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a4a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a4a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a4a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a4a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a4a .ks-blue{ fill:#185FA5; font-size:13px; }
.a4a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a4a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a4a .kb-blue, :root[data-theme="dark"] .a4a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a4a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a4a .kh-blue{ fill:#85B7EB; }
.a4a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a4a-blue); }
.a4a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4a-blue); }
.a4a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a4a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a4a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a4a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a4a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a4a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a4a .kb-teal, :root[data-theme="dark"] .a4a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a4a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a4a .kh-teal{ fill:#5DCAA5; }
.a4a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a4a-teal); }
.a4a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4a-teal); }
.a4a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a4a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a4a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a4a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a4a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a4a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a4a .kb-gray, :root[data-theme="dark"] .a4a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a4a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a4a .kh-gray{ fill:#B4B2A9; }
.a4a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a4a-gray); }
.a4a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4a-gray); }
.a4a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a4a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a4a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a4a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a4a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a4a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a4a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a4a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a4a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a4a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a4a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a4a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a4a">
<rect x="20" y="40" width="158" height="60" rx="7" class="k-gray"/>
<text x="99.0" y="65.5" text-anchor="middle" class="km-gray">$ANSIBLE_CONFIG</text>
<text x="99.0" y="83.5" text-anchor="middle" class="ks-gray">a file you name</text>
<path d="M180 70 L190 70" class="ag"/>
<text x="99" y="28" text-anchor="middle" class="c">1.</text>
<rect x="192" y="40" width="158" height="60" rx="7" class="k-teal"/>
<text x="271.0" y="65.5" text-anchor="middle" class="km-teal">./ansible.cfg</text>
<text x="271.0" y="83.5" text-anchor="middle" class="ks-teal">the project folder</text>
<path d="M352 70 L362 70" class="ag"/>
<text x="271" y="28" text-anchor="middle" class="c">2.</text>
<rect x="364" y="40" width="158" height="60" rx="7" class="k-blue"/>
<text x="443.0" y="65.5" text-anchor="middle" class="km-blue">~/.ansible.cfg</text>
<text x="443.0" y="83.5" text-anchor="middle" class="ks-blue">your own defaults</text>
<path d="M524 70 L534 70" class="ag"/>
<text x="443" y="28" text-anchor="middle" class="c">3.</text>
<rect x="536" y="40" width="158" height="60" rx="7" class="k-gray"/>
<text x="615.0" y="65.5" text-anchor="middle" class="km-gray">/etc/ansible/</text>
<text x="615.0" y="83.5" text-anchor="middle" class="ks-gray">system-wide</text>
<text x="615" y="28" text-anchor="middle" class="c">4.</text>
<text x="360" y="136" text-anchor="middle" class="s">the first one found wins; the others are not read at all</text>
<text x="360" y="160" text-anchor="middle" class="kh-teal">a project's own ansible.cfg makes it behave the same for everyone</text>
</g>
</svg>
<figcaption>One configuration file, chosen by search order.</figcaption>
</figure>

So a project keeps its own `ansible.cfg` next to its playbooks, and behaves the same for everyone who runs it from that folder:

```
cd ~/northwind
cat > ansible.cfg <<'EOF'
[defaults]
inventory = inventory
remote_user = student
forks = 10

[privilege_escalation]
become_method = sudo
EOF
ansible --version | grep 'config file'
ansible-config dump --only-changed
```

```
CONFIG_FILE() = /home/student/northwind/ansible.cfg
DEFAULT_BECOME_METHOD(/home/student/northwind/ansible.cfg) = sudo
DEFAULT_FORKS(/home/student/northwind/ansible.cfg) = 10
DEFAULT_HOST_LIST(/home/student/northwind/ansible.cfg) = ['/home/student/northwind/inventory']
DEFAULT_REMOTE_USER(/home/student/northwind/ansible.cfg) = student
INTERPRETER_PYTHON(env: ANSIBLE_PYTHON_INTERPRETER) = auto_silent
```

`ansible-config dump --only-changed` shows every setting that differs from the default, and **where it came from**: the file, or an environment variable. Every setting also has an environment variable (`ANSIBLE_FORKS=20`) and many have a command-line option (`-f 20`), which win over the file. The control image sets `ANSIBLE_PYTHON_INTERPRETER=auto_silent` so Ansible finds each host's Python without a warning.

> Ansible ignores an `ansible.cfg` in a **world-writable** folder, since anyone could plant settings there. If `config file = None` appears unexpectedly, check the folder's permissions.

## Groups of groups

A host can be in many groups, and groups can contain groups. Northwind's inventory sorts servers two ways, by job and by environment:

```
cat > inventory <<'EOF'
# Northwind's servers, by job
[web]
web1
web2

[db]
db1 ansible_host=10.0.0.21

# ... and by environment
[production]
web1
db1

[staging]
web2

# every Linux server, whatever its job
[linux:children]
web
db
EOF
ansible-inventory --graph
```

```
@all:
  |--@ungrouped:
  |--@production:
  |  |--web1
  |  |--db1
  |--@staging:
  |  |--web2
  |--@linux:
  |  |--@web:
  |  |  |--web1
  |  |  |--web2
  |  |--@db:
  |  |  |--db1
```

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Northwind&#x27;s hosts sorted two ways. By job: the linux group contains the web group, with web1 and web2, and the db group, with db1. By environment: production contains web1 and db1, staging contains web2. Each host belongs to one job group and one environment group." xmlns="http://www.w3.org/2000/svg">
<style>
.a4b{ font-family:var(--sans),sans-serif; }
.a4b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a4b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a4b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a4b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a4b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a4b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a4b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a4b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a4b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a4b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a4b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a4b .s{ fill:var(--text-dim); font-size:13px; }
.a4b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a4b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a4b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a4b .c{ fill:var(--text-faint); font-size:12.5px; }
.a4b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a4b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a4b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a4b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4b-a); }
.a4b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4b-a); marker-start:url(#a4b-s); }
.a4b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a4b-g); }
.a4b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-a); }
.a4b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a4b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a4b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a4b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a4b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a4b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4b-tri); }
.a4b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-tri); }
.a4b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4b-open); }
.a4b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-open); }
.a4b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4b-dia); }
.a4b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4b-odia); }
.a4b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4b-dia); marker-end:url(#a4b-open); }
.a4b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4b-solid); }
.a4b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a4b-open); }
.a4b .fill{ fill:var(--text); stroke:none; }
.a4b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a4b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a4b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a4b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a4b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a4b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a4b .ks-blue{ fill:#185FA5; font-size:13px; }
.a4b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a4b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a4b .kb-blue, :root[data-theme="dark"] .a4b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a4b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a4b .kh-blue{ fill:#85B7EB; }
.a4b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a4b-blue); }
.a4b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-blue); }
.a4b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a4b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a4b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a4b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a4b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a4b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a4b .kb-teal, :root[data-theme="dark"] .a4b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a4b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a4b .kh-teal{ fill:#5DCAA5; }
.a4b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a4b-teal); }
.a4b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-teal); }
.a4b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a4b .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a4b .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a4b .ks-amber{ fill:#854F0B; font-size:13px; }
.a4b .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a4b .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4b .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a4b .kb-amber, :root[data-theme="dark"] .a4b .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a4b .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a4b .kh-amber{ fill:#EF9F27; }
.a4b .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a4b-amber); }
.a4b .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-amber); }
.a4b .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a4b .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a4b .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a4b .ks-coral{ fill:#993C1D; font-size:13px; }
.a4b .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a4b .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4b .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a4b .kb-coral, :root[data-theme="dark"] .a4b .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a4b .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a4b .kh-coral{ fill:#F0997B; }
.a4b .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a4b-coral); }
.a4b .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-coral); }
.a4b .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a4b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a4b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a4b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a4b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a4b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a4b .kb-gray, :root[data-theme="dark"] .a4b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a4b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a4b .kh-gray{ fill:#B4B2A9; }
.a4b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a4b-gray); }
.a4b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4b-gray); }
.a4b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a4b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a4b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a4b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a4b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a4b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a4b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a4b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a4b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a4b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a4b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a4b-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a4b-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a4b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a4b">
<rect x="20" y="20" width="420" height="230" rx="12" class="k-gray"/>
<text x="40" y="44" text-anchor="start" class="kb-gray">linux  (children: web, db)</text>
<rect x="40" y="60" width="230" height="120" rx="10" class="k-blue"/>
<text x="56" y="82" text-anchor="start" class="kb-blue">web</text>
<rect x="290" y="60" width="130" height="120" rx="10" class="k-amber"/>
<text x="306" y="82" text-anchor="start" class="kb-amber">db</text>
<rect x="60" y="110" width="90" height="40" rx="7" class="pnl"/>
<text x="105.0" y="134.5" text-anchor="middle" class="b">web1</text>
<rect x="160" y="110" width="90" height="40" rx="7" class="pnl"/>
<text x="205.0" y="134.5" text-anchor="middle" class="b">web2</text>
<rect x="310" y="110" width="90" height="40" rx="7" class="pnl"/>
<text x="355.0" y="134.5" text-anchor="middle" class="b">db1</text>
<rect x="470" y="40" width="230" height="80" rx="7" class="k-teal"/>
<text x="585.0" y="75.5" text-anchor="middle" class="kb-teal">production</text>
<text x="585.0" y="93.5" text-anchor="middle" class="ks-teal">web1, db1</text>
<rect x="470" y="150" width="230" height="80" rx="7" class="k-coral"/>
<text x="585.0" y="185.5" text-anchor="middle" class="kb-coral">staging</text>
<text x="585.0" y="203.5" text-anchor="middle" class="ks-coral">web2</text>
<path d="M152 130 C 300 210, 420 120, 466 90" class="ad-teal"/>
<path d="M402 130 C 440 110, 450 90, 466 84" class="ad-teal"/>
<path d="M252 130 C 330 220, 420 200, 466 190" class="ad-coral"/>
<text x="230" y="230" text-anchor="middle" class="c">by job</text>
<text x="585" y="250" text-anchor="middle" class="c">by environment</text>
</g>
</svg>
<figcaption>Two ways of grouping the same hosts; patterns can combine them.</figcaption>
</figure>

- `[linux:children]` makes a group whose members are groups. Everything in *web* and *db* is in *linux*.
- `ansible_host=10.0.0.21` gives the address to connect to. The inventory name stays `db1`; the address can be anything reachable.
- `ansible-inventory --graph` draws the tree as Ansible understands it: the quickest check that an inventory says what you meant.

## Choosing hosts with groups

The two dimensions combine with host patterns from Chapter 2:

```
ansible 'web:&production' --list-hosts      # web servers in production: web1
ansible 'linux:!staging' --list-hosts       # every Linux server except staging: web1, db1
ansible 'production:staging' --list-hosts   # both environments: all three
```

Typical uses: *update staging first*, *restart the production web servers*, *patch every Linux server except the database during business hours*. One inventory serves all of them.

## Variables in folders: group_vars and host_vars

Settings differ by group and by host. Variables could go in the inventory file (`[web:vars]`), but that gets messy fast. The usual place is a **folder per group and per host**, next to the inventory, written in YAML:

```
mkdir -p group_vars host_vars
cat > group_vars/all.yml <<'EOF'
company: Northwind
admin_email: ops@northwind.lab
ntp_server: pool.ntp.org
EOF
cat > group_vars/web.yml <<'EOF'
site_name: Northwind Coffee
http_port: 80
EOF
printf 'environment_name: production\n' > group_vars/production.yml
printf 'environment_name: staging\n' > group_vars/staging.yml
printf 'site_name: Northwind Coffee (staging)\n' > host_vars/web2.yml
```

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The project&#x27;s variable files and which of them apply to web2: group_vars/all.yml, group_vars/web.yml, group_vars/staging.yml and host_vars/web2.yml apply; group_vars/production.yml does not, because web2 is not in production." xmlns="http://www.w3.org/2000/svg">
<style>
.a4c{ font-family:var(--sans),sans-serif; }
.a4c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a4c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a4c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a4c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a4c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a4c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a4c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a4c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a4c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a4c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a4c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a4c .s{ fill:var(--text-dim); font-size:13px; }
.a4c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a4c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a4c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a4c .c{ fill:var(--text-faint); font-size:12.5px; }
.a4c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a4c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a4c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a4c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4c-a); }
.a4c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4c-a); marker-start:url(#a4c-s); }
.a4c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a4c-g); }
.a4c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-a); }
.a4c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a4c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a4c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a4c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a4c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a4c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4c-tri); }
.a4c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-tri); }
.a4c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4c-open); }
.a4c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-open); }
.a4c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4c-dia); }
.a4c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4c-odia); }
.a4c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4c-dia); marker-end:url(#a4c-open); }
.a4c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4c-solid); }
.a4c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a4c-open); }
.a4c .fill{ fill:var(--text); stroke:none; }
.a4c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a4c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a4c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a4c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a4c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a4c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a4c .ks-blue{ fill:#185FA5; font-size:13px; }
.a4c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a4c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a4c .kb-blue, :root[data-theme="dark"] .a4c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a4c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a4c .kh-blue{ fill:#85B7EB; }
.a4c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a4c-blue); }
.a4c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-blue); }
.a4c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a4c .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a4c .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a4c .ks-teal{ fill:#0F6E56; font-size:13px; }
.a4c .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a4c .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4c .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a4c .kb-teal, :root[data-theme="dark"] .a4c .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a4c .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a4c .kh-teal{ fill:#5DCAA5; }
.a4c .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a4c-teal); }
.a4c .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-teal); }
.a4c .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a4c .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a4c .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a4c .ks-coral{ fill:#993C1D; font-size:13px; }
.a4c .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a4c .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4c .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a4c .kb-coral, :root[data-theme="dark"] .a4c .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a4c .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a4c .kh-coral{ fill:#F0997B; }
.a4c .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a4c-coral); }
.a4c .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-coral); }
.a4c .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a4c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a4c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a4c .ks-purple{ fill:#534AB7; font-size:13px; }
.a4c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a4c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a4c .kb-purple, :root[data-theme="dark"] .a4c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a4c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a4c .kh-purple{ fill:#AFA9EC; }
.a4c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a4c-purple); }
.a4c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-purple); }
.a4c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a4c .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a4c .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a4c .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a4c .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a4c .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4c .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a4c .kb-gray, :root[data-theme="dark"] .a4c .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a4c .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a4c .kh-gray{ fill:#B4B2A9; }
.a4c .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a4c-gray); }
.a4c .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4c-gray); }
.a4c .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a4c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a4c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a4c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a4c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a4c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a4c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a4c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a4c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a4c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a4c-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a4c-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a4c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a4c-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a4c">
<rect x="20" y="20" width="250" height="232" rx="6" class="pnl"/>
<text x="32.0" y="40" text-anchor="start" class="m">northwind/</text>
<text x="32.0" y="64" text-anchor="start" class="m">├── inventory</text>
<text x="32.0" y="88" text-anchor="start" class="m">├── group_vars/</text>
<text x="32.0" y="112" text-anchor="start" class="m">│   ├── all.yml</text>
<text x="32.0" y="136" text-anchor="start" class="m">│   ├── web.yml</text>
<text x="32.0" y="160" text-anchor="start" class="m">│   ├── production.yml</text>
<text x="32.0" y="184" text-anchor="start" class="m">│   └── staging.yml</text>
<text x="32.0" y="208" text-anchor="start" class="m">└── host_vars/</text>
<text x="62.4" y="232" text-anchor="start" class="m">└── web2.yml</text>
<text x="500" y="26" text-anchor="middle" class="b">applied to web2?</text>
<rect x="300" y="40" width="170" height="40" rx="5" class="k-gray"/>
<text x="385.0" y="64.5" text-anchor="middle" class="km-gray">all.yml</text>
<text x="490" y="65" text-anchor="start" class="s">company, admin_email</text>
<text x="680" y="65" text-anchor="middle" class="kh-teal">yes</text>
<rect x="300" y="90" width="170" height="40" rx="5" class="k-blue"/>
<text x="385.0" y="114.5" text-anchor="middle" class="km-blue">web.yml</text>
<text x="490" y="115" text-anchor="start" class="s">site_name, http_port</text>
<text x="680" y="115" text-anchor="middle" class="kh-teal">yes</text>
<rect x="300" y="140" width="170" height="40" rx="5" class="pnl"/>
<text x="385.0" y="164.5" text-anchor="middle" class="m">production.yml</text>
<text x="490" y="165" text-anchor="start" class="c">environment_name</text>
<text x="680" y="165" text-anchor="middle" class="c">no</text>
<rect x="300" y="190" width="170" height="40" rx="5" class="k-coral"/>
<text x="385.0" y="214.5" text-anchor="middle" class="km-coral">staging.yml</text>
<text x="490" y="215" text-anchor="start" class="s">environment_name</text>
<text x="680" y="215" text-anchor="middle" class="kh-teal">yes</text>
<rect x="300" y="240" width="170" height="40" rx="5" class="k-purple"/>
<text x="385.0" y="264.5" text-anchor="middle" class="km-purple">web2.yml</text>
<text x="490" y="265" text-anchor="start" class="s">site_name (override)</text>
<text x="680" y="265" text-anchor="middle" class="kh-teal">yes</text>
</g>
</svg>
<figcaption>Every group a host is in contributes its file; the host's own file comes last.</figcaption>
</figure>

Ansible loads `group_vars/GROUP.yml` (or every file in a folder `group_vars/GROUP/`) for **every group a host belongs to**, then `host_vars/HOST.yml`. To see what a host ends up with:

```
ansible-inventory --host web2
ansible web -m debug -a 'msg="{{ inventory_hostname }}: {{ site_name }} on port {{ http_port }} ({{ environment_name }})"'
```

```
web1 | SUCCESS => {
    "msg": "web1: Northwind Coffee on port 80 (production)"
}
web2 | SUCCESS => {
    "msg": "web2: Northwind Coffee (staging) on port 80 (staging)"
}
```

The `debug` module prints a message, and `{{ }}` puts a variable's value in it: a **Jinja2** expression, the templating language behind all of Ansible's variables (Chapters 6 and 7).

## Which value wins

When the same variable is set at several levels, the **more specific** one wins:

<figure class="diagram">
<svg viewBox="0 0 720 240" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Inventory variable precedence from weakest to strongest: group all, parent groups such as linux, child groups such as web and staging, and finally the host&#x27;s own variables. For web2, site_name from host_vars/web2.yml beats the value from group_vars/web.yml." xmlns="http://www.w3.org/2000/svg">
<style>
.a4d{ font-family:var(--sans),sans-serif; }
.a4d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a4d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a4d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a4d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a4d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a4d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a4d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a4d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a4d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a4d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a4d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a4d .s{ fill:var(--text-dim); font-size:13px; }
.a4d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a4d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a4d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a4d .c{ fill:var(--text-faint); font-size:12.5px; }
.a4d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a4d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a4d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a4d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4d-a); }
.a4d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4d-a); marker-start:url(#a4d-s); }
.a4d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a4d-g); }
.a4d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a4d-a); }
.a4d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a4d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a4d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a4d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a4d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a4d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4d-tri); }
.a4d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4d-tri); }
.a4d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4d-open); }
.a4d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4d-open); }
.a4d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4d-dia); }
.a4d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4d-odia); }
.a4d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4d-dia); marker-end:url(#a4d-open); }
.a4d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4d-solid); }
.a4d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a4d-open); }
.a4d .fill{ fill:var(--text); stroke:none; }
.a4d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a4d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a4d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a4d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a4d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a4d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a4d .ks-blue{ fill:#185FA5; font-size:13px; }
.a4d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a4d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a4d .kb-blue, :root[data-theme="dark"] .a4d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a4d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a4d .kh-blue{ fill:#85B7EB; }
.a4d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a4d-blue); }
.a4d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4d-blue); }
.a4d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a4d .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a4d .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a4d .ks-purple{ fill:#534AB7; font-size:13px; }
.a4d .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a4d .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4d .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a4d .kb-purple, :root[data-theme="dark"] .a4d .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a4d .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a4d .kh-purple{ fill:#AFA9EC; }
.a4d .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a4d-purple); }
.a4d .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4d-purple); }
.a4d .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a4d .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a4d .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a4d .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a4d .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a4d .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4d .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a4d .kb-gray, :root[data-theme="dark"] .a4d .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a4d .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a4d .kh-gray{ fill:#B4B2A9; }
.a4d .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a4d-gray); }
.a4d .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4d-gray); }
.a4d .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a4d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a4d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a4d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a4d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a4d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a4d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a4d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a4d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a4d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a4d-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a4d-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a4d">
<rect x="20" y="130" width="158" height="60" rx="8" class="k-gray"/>
<text x="99" y="152" text-anchor="middle" class="kb-gray">all</text>
<text x="99" y="172" text-anchor="middle" class="ks-gray">company: Northwind</text>
<rect x="192" y="100" width="158" height="90" rx="8" class="k-gray"/>
<text x="271" y="122" text-anchor="middle" class="kb-gray">parent group: linux</text>
<rect x="364" y="70" width="158" height="120" rx="8" class="k-blue"/>
<text x="443" y="92" text-anchor="middle" class="kb-blue">child groups</text>
<text x="443" y="112" text-anchor="middle" class="ks-blue">web, staging</text>
<rect x="536" y="40" width="158" height="150" rx="8" class="k-purple"/>
<text x="615" y="62" text-anchor="middle" class="kb-purple">host: web2</text>
<text x="615" y="82" text-anchor="middle" class="ks-purple">host_vars/web2.yml</text>
<path d="M30 212 L690 212" class="ar"/>
<text x="360" y="226" text-anchor="middle" class="a">more specific: wins</text>
</g>
</svg>
<figcaption>For the same variable, the more specific level wins.</figcaption>
</figure>

- `all` is the weakest group, then parent groups, then child groups, then the host.
- A host's own `host_vars` beats every group.
- Groups at the same level (web2 is in *web* and in *staging*) are merged in alphabetical order, so the later name wins a tie. Avoid relying on that: set each variable at one level.
- The `group_vars` and `host_vars` folders beat variables written in the inventory file itself.

Chapter 6 shows the full order, including variables set in plays and on the command line.

## The same inventory in YAML

INI is compact; **YAML** can hold lists and nested values that INI can't, and is the format of everything else in Ansible:

```
cat > inventory.yml <<'EOF'
all:
  children:
    web:
      hosts:
        web[1:2]:
    db:
      hosts:
        db1:
          ansible_host: 10.0.0.21
    production:
      hosts:
        web1:
        db1:
    staging:
      hosts:
        web2:
    linux:
      children:
        web:
        db:
EOF
ansible-inventory -i inventory.yml --graph
```

`web[1:2]` is a **range**: web1 and web2. `web[01:20]` would be twenty hosts, `db-[a:c]` three. `group_vars/` and `host_vars/` still apply, because they sit next to the inventory file. Teams usually pick one format and stay with it; this book uses INI for the small inventories and YAML for the router inventory of Chapter 13, where hosts carry structured data.

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The same inventory in INI and YAML. INI: section headers in brackets list hosts; children sections make groups of groups; host variables follow the name. YAML: a nested tree under all and children, with hosts as keys and their variables beneath them, and a range web[1:2]." xmlns="http://www.w3.org/2000/svg">
<style>
.a4e{ font-family:var(--sans),sans-serif; }
.a4e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a4e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a4e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a4e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a4e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a4e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a4e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a4e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a4e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a4e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a4e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a4e .s{ fill:var(--text-dim); font-size:13px; }
.a4e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a4e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a4e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a4e .c{ fill:var(--text-faint); font-size:12.5px; }
.a4e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a4e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a4e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a4e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4e-a); }
.a4e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4e-a); marker-start:url(#a4e-s); }
.a4e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a4e-g); }
.a4e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a4e-a); }
.a4e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a4e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a4e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a4e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a4e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a4e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4e-tri); }
.a4e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4e-tri); }
.a4e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4e-open); }
.a4e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4e-open); }
.a4e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4e-dia); }
.a4e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4e-odia); }
.a4e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4e-dia); marker-end:url(#a4e-open); }
.a4e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4e-solid); }
.a4e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a4e-open); }
.a4e .fill{ fill:var(--text); stroke:none; }
.a4e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a4e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a4e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a4e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a4e .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a4e .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a4e .ks-teal{ fill:#0F6E56; font-size:13px; }
.a4e .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a4e .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4e .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a4e .kb-teal, :root[data-theme="dark"] .a4e .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a4e .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a4e .kh-teal{ fill:#5DCAA5; }
.a4e .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a4e-teal); }
.a4e .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4e-teal); }
.a4e .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a4e .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a4e .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a4e .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a4e .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a4e .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4e .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a4e .kb-gray, :root[data-theme="dark"] .a4e .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a4e .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a4e .kh-gray{ fill:#B4B2A9; }
.a4e .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a4e-gray); }
.a4e .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4e-gray); }
.a4e .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a4e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a4e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a4e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a4e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a4e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a4e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a4e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a4e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a4e-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a4e-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a4e">
<rect x="20" y="16" width="300" height="228" rx="6" class="k-gray"/>
<text x="30" y="36" text-anchor="start" class="kh-gray">inventory (INI)</text>
<text x="32.0" y="58" text-anchor="start" class="m">[web]</text>
<text x="32.0" y="77" text-anchor="start" class="m">web1</text>
<text x="32.0" y="96" text-anchor="start" class="m">web2</text>
<text x="32.0" y="115" text-anchor="start" class="m"></text>
<text x="32.0" y="134" text-anchor="start" class="m">[db]</text>
<text x="32.0" y="153" text-anchor="start" class="m">db1 ansible_host=10.0.0.21</text>
<text x="32.0" y="172" text-anchor="start" class="m"></text>
<text x="32.0" y="191" text-anchor="start" class="m">[linux:children]</text>
<text x="32.0" y="210" text-anchor="start" class="m">web</text>
<text x="32.0" y="229" text-anchor="start" class="m">db</text>
<rect x="360" y="16" width="340" height="247" rx="6" class="k-teal"/>
<text x="370" y="36" text-anchor="start" class="kh-teal">inventory.yml (YAML)</text>
<text x="372.0" y="58" text-anchor="start" class="m">all:</text>
<text x="387.2" y="77" text-anchor="start" class="m">children:</text>
<text x="402.4" y="96" text-anchor="start" class="m">web:</text>
<text x="417.6" y="115" text-anchor="start" class="m">hosts:</text>
<text x="432.8" y="134" text-anchor="start" class="m">web[1:2]:</text>
<text x="402.4" y="153" text-anchor="start" class="m">db:</text>
<text x="417.6" y="172" text-anchor="start" class="m">hosts:</text>
<text x="432.8" y="191" text-anchor="start" class="m">db1:</text>
<text x="448.0" y="210" text-anchor="start" class="m">ansible_host: 10.0.0.21</text>
<text x="402.4" y="229" text-anchor="start" class="m">linux:</text>
<text x="417.6" y="248" text-anchor="start" class="m">children: {web: {}, db: {}}</text>
</g>
</svg>
<figcaption>Two formats, one meaning; YAML holds structured data more easily.</figcaption>
</figure>

## Connection variables

Variables starting with `ansible_` tell Ansible **how** to reach a host:

| Variable | Meaning |
|---|---|
| `ansible_host` | address or name to connect to |
| `ansible_port` | SSH port, if not 22 |
| `ansible_user` | login user (or `remote_user` in `ansible.cfg`) |
| `ansible_ssh_private_key_file` | key to use |
| `ansible_become`, `ansible_become_user` | become, and as whom |
| `ansible_python_interpreter` | Python to use on the host |
| `ansible_connection` | `ssh` (default), `local`, `docker`, `network_cli` ... |

```
ansible web1 -m ping -e ansible_user=nobody
```

`-e` sets a variable for one run: as *nobody*, the login fails and web1 is **UNREACHABLE**. Set connection variables in the inventory or `group_vars` only for hosts that differ from the defaults.

## Dynamic inventories

Northwind's inventory is a file because its servers rarely change. In a cloud, servers come and go every hour, and a file would always be out of date. A **dynamic inventory** asks a source of truth, such as a cloud provider's API, a virtualization platform or a CMDB, for the current list, every time Ansible runs.

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A static inventory is a file you edit. A dynamic inventory is a plugin or script that queries a source of truth, such as a cloud API, VMware or a CMDB, each time Ansible runs, and returns hosts and groups as JSON, for example groups built from instance tags." xmlns="http://www.w3.org/2000/svg">
<style>
.a4f{ font-family:var(--sans),sans-serif; }
.a4f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a4f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a4f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a4f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a4f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a4f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a4f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a4f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a4f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a4f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a4f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a4f .s{ fill:var(--text-dim); font-size:13px; }
.a4f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a4f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a4f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a4f .c{ fill:var(--text-faint); font-size:12.5px; }
.a4f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a4f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a4f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a4f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4f-a); }
.a4f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a4f-a); marker-start:url(#a4f-s); }
.a4f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a4f-g); }
.a4f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a4f-a); }
.a4f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a4f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a4f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a4f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a4f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a4f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4f-tri); }
.a4f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4f-tri); }
.a4f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4f-open); }
.a4f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a4f-open); }
.a4f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4f-dia); }
.a4f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4f-odia); }
.a4f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a4f-dia); marker-end:url(#a4f-open); }
.a4f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a4f-solid); }
.a4f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a4f-open); }
.a4f .fill{ fill:var(--text); stroke:none; }
.a4f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a4f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a4f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a4f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a4f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a4f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a4f .ks-blue{ fill:#185FA5; font-size:13px; }
.a4f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a4f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a4f .kb-blue, :root[data-theme="dark"] .a4f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a4f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a4f .kh-blue{ fill:#85B7EB; }
.a4f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a4f-blue); }
.a4f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4f-blue); }
.a4f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a4f .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a4f .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a4f .ks-amber{ fill:#854F0B; font-size:13px; }
.a4f .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a4f .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4f .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a4f .kb-amber, :root[data-theme="dark"] .a4f .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a4f .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a4f .kh-amber{ fill:#EF9F27; }
.a4f .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a4f-amber); }
.a4f .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4f-amber); }
.a4f .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a4f .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a4f .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a4f .ks-purple{ fill:#534AB7; font-size:13px; }
.a4f .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a4f .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4f .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a4f .kb-purple, :root[data-theme="dark"] .a4f .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a4f .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a4f .kh-purple{ fill:#AFA9EC; }
.a4f .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a4f-purple); }
.a4f .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4f-purple); }
.a4f .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a4f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a4f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a4f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a4f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a4f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a4f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a4f .kb-gray, :root[data-theme="dark"] .a4f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a4f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a4f .kh-gray{ fill:#B4B2A9; }
.a4f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a4f-gray); }
.a4f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a4f-gray); }
.a4f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a4f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a4f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a4f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a4f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a4f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a4f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a4f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a4f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a4f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a4f-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a4f-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a4f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a4f">
<rect x="20" y="30" width="180" height="60" rx="7" class="k-gray"/>
<text x="110.0" y="55.5" text-anchor="middle" class="kb-gray">static: inventory</text>
<text x="110.0" y="73.5" text-anchor="middle" class="ks-gray">a file you edit</text>
<rect x="20" y="130" width="180" height="70" rx="7" class="k-amber"/>
<text x="110.0" y="151.5" text-anchor="middle" class="kb-amber">cloud API, VMware,</text>
<text x="110.0" y="169.5" text-anchor="middle" class="kb-amber">CMDB, NetBox ...</text>
<text x="110.0" y="187.5" text-anchor="middle" class="ks-amber">source of truth</text>
<rect x="250" y="130" width="180" height="70" rx="7" class="k-purple"/>
<text x="340.0" y="160.5" text-anchor="middle" class="kb-purple">inventory plugin</text>
<text x="340.0" y="178.5" text-anchor="middle" class="km-purple">aws_ec2, azure_rm ...</text>
<path d="M202 165 L246 165" class="ar-amber"/>
<rect x="450" y="20" width="260" height="128" rx="6" class="k-blue"/>
<text x="460" y="40" text-anchor="start" class="kh-blue">JSON, like --list</text>
<text x="462.0" y="62" text-anchor="start" class="m">{ "web": {</text>
<text x="492.4" y="80" text-anchor="start" class="m">"hosts": ["web1","web2"] },</text>
<text x="477.2" y="98" text-anchor="start" class="m">"tag_env_prod": {</text>
<text x="492.4" y="116" text-anchor="start" class="m">"hosts": ["web1","db1"] },</text>
<text x="477.2" y="134" text-anchor="start" class="m">"_meta": { "hostvars": ... } }</text>
<path d="M432 165 L446 130" class="ar-purple"/>
<path d="M202 60 L446 60" class="ag"/>
</g>
</svg>
<figcaption>Dynamic inventories rebuild the host list every run.</figcaption>
</figure>

```
ansible-inventory --list | head -20
```

`--list` prints the inventory as JSON. A dynamic inventory is any program or **inventory plugin** that produces exactly this: the `amazon.aws.aws_ec2`, `azure.azcollection.azure_rm` and `community.vmware.vmware_vm_inventory` plugins, for example, turn instance tags into groups. The rest of the project, playbooks, `group_vars` and roles, does not change.

## The project so far

```
northwind/
├── ansible.cfg
├── inventory
├── group_vars/
│   ├── all.yml
│   ├── web.yml
│   ├── production.yml
│   └── staging.yml
└── host_vars/
    └── web2.yml
```

## Hands on

1. web2 is in groups *web*, *staging* and *linux*. List every file in `group_vars` and `host_vars` that applies to it, then check with `ansible-inventory --host web2`.
2. Put `site_name: Linux box` in `group_vars/linux.yml`. Which value does web1 get, and why?
3. Add `stdout_callback = yaml` (or, on newer versions, `callback_result_format = yaml`) to `[defaults]` and run a `debug`. Easier to read?
4. Write a tiny dynamic inventory: a script that prints the JSON of `ansible-inventory --list`, made executable, passed with `-i`.

## Summary

- Ansible reads **one** `ansible.cfg`: `$ANSIBLE_CONFIG`, then `./ansible.cfg`, then `~/.ansible.cfg`, then `/etc/ansible/ansible.cfg`. Keep one per project. `ansible-config dump --only-changed` shows what is set and where.
- A host can be in many groups; `[group:children]` builds groups of groups. Sorting by **job** and by **environment** lets patterns like `web:&production` pick exactly the right hosts.
- Variables live in `group_vars/` and `host_vars/`; the more specific level wins.
- `ansible-inventory --graph`, `--host` and `--list` show the inventory as Ansible sees it.
- `ansible_*` variables control the connection. **Dynamic inventories** build the host list from a live source.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-4-inventory-and-configuration).

**1. Which ansible.cfg does Ansible read when you run it in ~/northwind, which has its own ansible.cfg, and no ANSIBLE_CONFIG is set?**

- A. /etc/ansible/ansible.cfg
- B. ./ansible.cfg, in the project folder
- C. ~/.ansible.cfg
- D. All of them, merged

**2. What does ansible-config dump --only-changed show?**

- A. Every setting
- B. Changed files in the project
- C. Settings that differ from the defaults, and where each came from
- D. The inventory

**3. What does [linux:children] in an INI inventory create?**

- A. A group whose members are other groups
- B. Hosts named linux
- C. A host variable
- D. A dynamic inventory

**4. Which pattern selects the web servers that are also in production?**

- A. web:&production
- B. web:production
- C. web:!production
- D. production:!web

**5. web2 is in groups web and staging, and has host_vars/web2.yml. Which value of site_name wins if all three set it?**

- A. group_vars/web.yml
- B. group_vars/staging.yml
- C. host_vars/web2.yml
- D. group_vars/all.yml

**6. What does ansible_host=10.0.0.21 in an inventory line do?**

- A. Renames the host
- B. Gives the address to connect to, keeping the inventory name
- C. Sets the SSH port
- D. Adds the host to a group

**7. What does web[1:2] mean in an inventory?**

- A. Host web with ports 1 to 2
- B. Two groups named web
- C. A list variable
- D. Hosts web1 and web2

**8. What is a dynamic inventory?**

- A. An inventory that changes every second
- B. A plugin or script that builds the host list at run time from a source such as a cloud API
- C. An inventory in YAML
- D. An inventory with variables
