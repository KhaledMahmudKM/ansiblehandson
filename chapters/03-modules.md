# Chapter 3: Modules and Idempotency

Northwind's servers answer `ping`. Now you want to change something on them: install a few tools, put a notice in place, create an account for deployments, set one line in the SSH configuration. You could send shell commands with `-a`, but Ansible offers something better: **modules**, each an expert in bringing one kind of thing into a wanted state. This chapter uses the most common ones from the command line, and shows the property that makes them better than commands: **idempotency**.

> **Try it:** DockerLab, *Ansible lab 2, Modules and idempotency*. Same topology as lab 1. Start with `setup-keys web1 web2 db1`.

## A project folder

From here on, everything lives in one project folder, `~/northwind`. A two-line `ansible.cfg` makes the inventory the default for commands run there, so `-i inventory` is no longer needed (Chapter 4 explains the file):

```
setup-keys web1 web2 db1
mkdir -p ~/northwind && cd ~/northwind
printf '[web]\nweb1\nweb2\n\n[db]\ndb1\n' > inventory
printf '[defaults]\ninventory = inventory\n' > ansible.cfg
ansible all -m ping
```

## Modules, and how to read about them

A module is a small program with **parameters**, which describe the state you want. Ansible ships thousands, grouped by job:

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Families of commonly used modules: packages (apt, dnf, package, pip); files (copy, template, file, lineinfile, blockinfile, fetch); users and access (user, group, authorized_key); services (service, systemd); commands (command, shell, script, raw); information (setup, stat, ping, debug); network and web (uri, get_url, wait_for); security (iptables, ufw)." xmlns="http://www.w3.org/2000/svg">
<style>
.a3a{ font-family:var(--sans),sans-serif; }
.a3a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a3a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a3a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a3a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a3a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a3a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a3a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a3a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a3a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a3a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a3a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a3a .s{ fill:var(--text-dim); font-size:13px; }
.a3a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a3a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .c{ fill:var(--text-faint); font-size:12.5px; }
.a3a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a3a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a3a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a3a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3a-a); }
.a3a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3a-a); marker-start:url(#a3a-s); }
.a3a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a3a-g); }
.a3a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-a); }
.a3a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a3a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a3a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a3a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a3a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a3a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3a-tri); }
.a3a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-tri); }
.a3a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3a-open); }
.a3a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-open); }
.a3a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3a-dia); }
.a3a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3a-odia); }
.a3a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3a-dia); marker-end:url(#a3a-open); }
.a3a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3a-solid); }
.a3a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a3a-open); }
.a3a .fill{ fill:var(--text); stroke:none; }
.a3a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a3a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a3a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a3a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a3a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a3a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a3a .ks-blue{ fill:#185FA5; font-size:13px; }
.a3a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a3a .kb-blue, :root[data-theme="dark"] .a3a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a3a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a3a .kh-blue{ fill:#85B7EB; }
.a3a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a3a-blue); }
.a3a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-blue); }
.a3a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a3a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a3a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a3a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a3a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a3a .kb-teal, :root[data-theme="dark"] .a3a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a3a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a3a .kh-teal{ fill:#5DCAA5; }
.a3a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a3a-teal); }
.a3a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-teal); }
.a3a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a3a .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a3a .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a3a .ks-amber{ fill:#854F0B; font-size:13px; }
.a3a .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a3a .kb-amber, :root[data-theme="dark"] .a3a .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a3a .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a3a .kh-amber{ fill:#EF9F27; }
.a3a .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a3a-amber); }
.a3a .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-amber); }
.a3a .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a3a .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a3a .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a3a .ks-coral{ fill:#993C1D; font-size:13px; }
.a3a .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a3a .kb-coral, :root[data-theme="dark"] .a3a .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a3a .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a3a .kh-coral{ fill:#F0997B; }
.a3a .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a3a-coral); }
.a3a .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-coral); }
.a3a .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a3a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a3a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a3a .ks-purple{ fill:#534AB7; font-size:13px; }
.a3a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a3a .kb-purple, :root[data-theme="dark"] .a3a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a3a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a3a .kh-purple{ fill:#AFA9EC; }
.a3a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a3a-purple); }
.a3a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-purple); }
.a3a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a3a .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a3a .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a3a .ks-green{ fill:#3B6D11; font-size:13px; }
.a3a .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a3a .kb-green, :root[data-theme="dark"] .a3a .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a3a .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a3a .kh-green{ fill:#97C459; }
.a3a .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a3a-green); }
.a3a .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-green); }
.a3a .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a3a .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a3a .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a3a .ks-red{ fill:#A32D2D; font-size:13px; }
.a3a .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a3a .kb-red, :root[data-theme="dark"] .a3a .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a3a .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a3a .kh-red{ fill:#F09595; }
.a3a .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a3a-red); }
.a3a .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-red); }
.a3a .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a3a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a3a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a3a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a3a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a3a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a3a .kb-gray, :root[data-theme="dark"] .a3a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a3a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a3a .kh-gray{ fill:#B4B2A9; }
.a3a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a3a-gray); }
.a3a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3a-gray); }
.a3a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a3a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a3a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a3a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a3a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a3a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a3a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a3a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a3a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a3a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a3a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a3a-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a3a-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a3a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a3a-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a3a-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a3a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a3a">
<rect x="20" y="16" width="160" height="130" rx="10" class="k-teal"/>
<text x="100" y="40" text-anchor="middle" class="kb-teal">packages</text>
<text x="100" y="64" text-anchor="middle" class="km-teal">apt</text>
<text x="100" y="81" text-anchor="middle" class="km-teal">dnf</text>
<text x="100" y="98" text-anchor="middle" class="km-teal">package</text>
<text x="100" y="115" text-anchor="middle" class="km-teal">pip</text>
<rect x="192" y="16" width="160" height="130" rx="10" class="k-blue"/>
<text x="272" y="40" text-anchor="middle" class="kb-blue">files</text>
<text x="272" y="64" text-anchor="middle" class="km-blue">copy</text>
<text x="272" y="81" text-anchor="middle" class="km-blue">template</text>
<text x="272" y="98" text-anchor="middle" class="km-blue">file</text>
<text x="272" y="115" text-anchor="middle" class="km-blue">lineinfile</text>
<text x="272" y="132" text-anchor="middle" class="km-blue">fetch</text>
<rect x="364" y="16" width="160" height="130" rx="10" class="k-purple"/>
<text x="444" y="40" text-anchor="middle" class="kb-purple">users</text>
<text x="444" y="64" text-anchor="middle" class="km-purple">user</text>
<text x="444" y="81" text-anchor="middle" class="km-purple">group</text>
<text x="444" y="98" text-anchor="middle" class="km-purple">authorized_key</text>
<rect x="536" y="16" width="160" height="130" rx="10" class="k-coral"/>
<text x="616" y="40" text-anchor="middle" class="kb-coral">services</text>
<text x="616" y="64" text-anchor="middle" class="km-coral">service</text>
<text x="616" y="81" text-anchor="middle" class="km-coral">systemd</text>
<rect x="20" y="158" width="160" height="130" rx="10" class="k-gray"/>
<text x="100" y="182" text-anchor="middle" class="kb-gray">commands</text>
<text x="100" y="206" text-anchor="middle" class="km-gray">command</text>
<text x="100" y="223" text-anchor="middle" class="km-gray">shell</text>
<text x="100" y="240" text-anchor="middle" class="km-gray">script</text>
<text x="100" y="257" text-anchor="middle" class="km-gray">raw</text>
<rect x="192" y="158" width="160" height="130" rx="10" class="k-amber"/>
<text x="272" y="182" text-anchor="middle" class="kb-amber">information</text>
<text x="272" y="206" text-anchor="middle" class="km-amber">setup</text>
<text x="272" y="223" text-anchor="middle" class="km-amber">stat</text>
<text x="272" y="240" text-anchor="middle" class="km-amber">ping</text>
<text x="272" y="257" text-anchor="middle" class="km-amber">debug</text>
<rect x="364" y="158" width="160" height="130" rx="10" class="k-green"/>
<text x="444" y="182" text-anchor="middle" class="kb-green">web</text>
<text x="444" y="206" text-anchor="middle" class="km-green">uri</text>
<text x="444" y="223" text-anchor="middle" class="km-green">get_url</text>
<text x="444" y="240" text-anchor="middle" class="km-green">wait_for</text>
<rect x="536" y="158" width="160" height="130" rx="10" class="k-red"/>
<text x="616" y="182" text-anchor="middle" class="kb-red">security</text>
<text x="616" y="206" text-anchor="middle" class="km-red">iptables</text>
<text x="616" y="223" text-anchor="middle" class="km-red">ufw</text>
<text x="616" y="240" text-anchor="middle" class="km-red">sysctl</text>
</g>
</svg>
<figcaption>A few dozen modules cover most daily work; full names are ansible.builtin.copy and so on.</figcaption>
</figure>

Each module has a documentation page: what it does, its parameters (required ones marked with `=`), what it returns, and, most useful of all, **examples** at the end.

```
ansible-doc copy | head -8
ansible-doc copy | grep -A12 'EXAMPLES:'
ansible-doc -s file | head -12
```

`-s` prints a short template of the parameters. Module names have a **fully qualified collection name** (FQCN), such as `ansible.builtin.copy`; the short name `copy` works too, but playbooks use the full name so there is never any doubt which `copy` is meant.

## command and shell

```
ansible web1 -a 'echo $HOME | tr a-z A-Z'
ansible web1 -m shell -a 'echo $HOME | tr a-z A-Z'
```

The first prints `/home/student | tr a-z A-Z`; the second prints `/HOME/STUDENT`. The **command** module runs a program directly, without a shell: `|`, `>`, `;`, `&&` and wildcards are passed to the program as plain text. The **shell** module runs the line in `/bin/sh`, so pipes and redirections work.

<figure class="diagram">
<svg viewBox="0 0 720 240" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The command module runs the program echo directly with its arguments, so the pipe symbol and tr are printed as text. The shell module hands the whole line to /bin/sh, which runs echo, pipes its output into tr, and prints the result in capitals." xmlns="http://www.w3.org/2000/svg">
<style>
.a3b{ font-family:var(--sans),sans-serif; }
.a3b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a3b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a3b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a3b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a3b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a3b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a3b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a3b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a3b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a3b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a3b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a3b .s{ fill:var(--text-dim); font-size:13px; }
.a3b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a3b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a3b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a3b .c{ fill:var(--text-faint); font-size:12.5px; }
.a3b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a3b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a3b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a3b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3b-a); }
.a3b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3b-a); marker-start:url(#a3b-s); }
.a3b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a3b-g); }
.a3b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a3b-a); }
.a3b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a3b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a3b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a3b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a3b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a3b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3b-tri); }
.a3b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3b-tri); }
.a3b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3b-open); }
.a3b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3b-open); }
.a3b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3b-dia); }
.a3b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3b-odia); }
.a3b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3b-dia); marker-end:url(#a3b-open); }
.a3b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3b-solid); }
.a3b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a3b-open); }
.a3b .fill{ fill:var(--text); stroke:none; }
.a3b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a3b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a3b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a3b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a3b .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a3b .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a3b .ks-coral{ fill:#993C1D; font-size:13px; }
.a3b .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a3b .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3b .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a3b .kb-coral, :root[data-theme="dark"] .a3b .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a3b .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a3b .kh-coral{ fill:#F0997B; }
.a3b .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a3b-coral); }
.a3b .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3b-coral); }
.a3b .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a3b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a3b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a3b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a3b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a3b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a3b .kb-gray, :root[data-theme="dark"] .a3b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a3b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a3b .kh-gray{ fill:#B4B2A9; }
.a3b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a3b-gray); }
.a3b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3b-gray); }
.a3b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a3b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a3b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a3b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a3b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a3b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a3b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a3b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a3b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a3b-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a3b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a3b">
<text x="180" y="22" text-anchor="middle" class="h">command (default)</text>
<text x="540" y="22" text-anchor="middle" class="h">shell</text>
<rect x="20" y="36" width="320" height="34" rx="6" class="k-gray"/>
<text x="32.0" y="56" text-anchor="start" class="m">-a 'echo $HOME | tr a-z A-Z'</text>
<rect x="380" y="36" width="320" height="34" rx="6" class="k-coral"/>
<text x="392.0" y="56" text-anchor="start" class="m">-m shell -a 'echo $HOME | tr a-z A-Z'</text>
<rect x="30" y="92" width="300" height="60" rx="7" class="pnl"/>
<text x="180.0" y="117.5" text-anchor="middle" class="c">echo gets the arguments:</text>
<text x="180.0" y="135.5" text-anchor="middle" class="m">/home/student | tr a-z A-Z</text>
<rect x="400" y="100" width="120" height="44" rx="7" class="k-coral"/>
<text x="460.0" y="126.5" text-anchor="middle" class="km-coral">/bin/sh</text>
<rect x="560" y="92" width="120" height="28" rx="4" class="pnl"/>
<text x="620.0" y="110.5" text-anchor="middle" class="m">echo</text>
<rect x="560" y="128" width="120" height="28" rx="4" class="pnl"/>
<text x="620.0" y="146.5" text-anchor="middle" class="m">| tr</text>
<path d="M522 122 L556 106" class="ar"/>
<path d="M522 122 L556 142" class="ar"/>
<text x="180" y="176" text-anchor="middle" class="m">/home/student | tr a-z A-Z</text>
<text x="540" y="176" text-anchor="middle" class="m">/HOME/STUDENT</text>
<text x="180" y="206" text-anchor="middle" class="kh-gray">no shell: safer, predictable</text>
<text x="540" y="206" text-anchor="middle" class="kh-coral">pipes, redirection, globs</text>
</g>
</svg>
<figcaption>Both always report CHANGED; a dedicated module is better than either.</figcaption>
</figure>

Prefer `command`: no shell means no surprises from special characters in a variable. Use `shell` only when you need a shell feature. Better still, use a module made for the job, because both always report **CHANGED**: they cannot know whether the program they ran changed anything.

## Idempotency, seen

The **copy** module puts a file in place, with content given inline or copied from control:

```
ansible all -m copy -a 'content="Managed by Ansible. Do not edit by hand.\n" dest=/tmp/notice.txt'
```

The first run creates the file on each host: **CHANGED**, in yellow. Run exactly the same command again:

```
web1 | SUCCESS => {
    "changed": false,
    "checksum": "9d472c0f7ae8fa577b4941d9cbc7c8d9c529ca01",
    "dest": "/tmp/notice.txt",
    ...
```

This time the file is already as described, so nothing is done: **SUCCESS**, `"changed": false`, in green. That is **idempotency**: an operation that gives the same result whether it runs once or many times. The module compares the current state with the desired state and acts only on the difference.

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="How an idempotent module decides. It reads the current state of the target, compares it with the desired state from its parameters, and if they are equal reports ok without changing anything; if they differ, it makes the change (unless in check mode) and reports changed." xmlns="http://www.w3.org/2000/svg">
<style>
.a3c{ font-family:var(--sans),sans-serif; }
.a3c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a3c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a3c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a3c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a3c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a3c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a3c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a3c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a3c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a3c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a3c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a3c .s{ fill:var(--text-dim); font-size:13px; }
.a3c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a3c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a3c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a3c .c{ fill:var(--text-faint); font-size:12.5px; }
.a3c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a3c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a3c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a3c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3c-a); }
.a3c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3c-a); marker-start:url(#a3c-s); }
.a3c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a3c-g); }
.a3c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a3c-a); }
.a3c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a3c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a3c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a3c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a3c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a3c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3c-tri); }
.a3c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3c-tri); }
.a3c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3c-open); }
.a3c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3c-open); }
.a3c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3c-dia); }
.a3c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3c-odia); }
.a3c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3c-dia); marker-end:url(#a3c-open); }
.a3c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3c-solid); }
.a3c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a3c-open); }
.a3c .fill{ fill:var(--text); stroke:none; }
.a3c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a3c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a3c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a3c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a3c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a3c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a3c .ks-blue{ fill:#185FA5; font-size:13px; }
.a3c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a3c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a3c .kb-blue, :root[data-theme="dark"] .a3c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a3c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a3c .kh-blue{ fill:#85B7EB; }
.a3c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a3c-blue); }
.a3c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3c-blue); }
.a3c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a3c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a3c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a3c .ks-amber{ fill:#854F0B; font-size:13px; }
.a3c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a3c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a3c .kb-amber, :root[data-theme="dark"] .a3c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a3c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a3c .kh-amber{ fill:#EF9F27; }
.a3c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a3c-amber); }
.a3c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3c-amber); }
.a3c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a3c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a3c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a3c .ks-purple{ fill:#534AB7; font-size:13px; }
.a3c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a3c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a3c .kb-purple, :root[data-theme="dark"] .a3c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a3c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a3c .kh-purple{ fill:#AFA9EC; }
.a3c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a3c-purple); }
.a3c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3c-purple); }
.a3c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a3c .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a3c .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a3c .ks-green{ fill:#3B6D11; font-size:13px; }
.a3c .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a3c .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3c .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a3c .kb-green, :root[data-theme="dark"] .a3c .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a3c .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a3c .kh-green{ fill:#97C459; }
.a3c .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a3c-green); }
.a3c .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3c-green); }
.a3c .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
</style>
<defs><marker id="a3c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a3c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a3c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a3c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a3c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a3c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a3c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a3c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a3c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a3c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a3c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a3c-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker></defs>
<g class="a3c">
<rect x="20" y="110" width="150" height="60" rx="7" class="k-purple"/>
<text x="95.0" y="135.5" text-anchor="middle" class="kb-purple">parameters</text>
<text x="95.0" y="153.5" text-anchor="middle" class="ks-purple">the desired state</text>
<rect x="200" y="110" width="150" height="60" rx="7" class="k-blue"/>
<text x="275.0" y="135.5" text-anchor="middle" class="kb-blue">read the target</text>
<text x="275.0" y="153.5" text-anchor="middle" class="ks-blue">current state</text>
<path d="M172 140 L196 140" class="ar"/>
<path d="M420 106 L454 140 L420 174 L386 140 z" class="k-amber"/>
<text x="420" y="144.5" text-anchor="middle" class="kb-amber">same?</text>
<path d="M352 140 L384 140" class="ar"/>
<rect x="520" y="30" width="180" height="56" rx="7" class="k-green"/>
<text x="610.0" y="53.5" text-anchor="middle" class="kb-green">ok</text>
<text x="610.0" y="71.5" text-anchor="middle" class="ks-green">do nothing</text>
<rect x="520" y="190" width="180" height="70" rx="7" class="k-amber"/>
<text x="610.0" y="211.5" text-anchor="middle" class="kb-amber">make the change</text>
<text x="610.0" y="229.5" text-anchor="middle" class="ks-amber">(not in --check)</text>
<text x="610.0" y="247.5" text-anchor="middle" class="ks-amber">report changed</text>
<path d="M440 112 L516 62" class="ar-green"/>
<text x="462" y="76" text-anchor="middle" class="kh-green">yes</text>
<path d="M440 168 L516 220" class="ar-amber"/>
<text x="462" y="212" text-anchor="middle" class="kh-amber">no</text>
</g>
</svg>
<figcaption>Check first, then change only the difference.</figcaption>
</figure>

Each result has one of five states. They appear in every ad-hoc command and every playbook run:

| Result | Meaning | Colour |
|---|---|---|
| **ok** (SUCCESS) | already in the desired state; nothing done | green |
| **changed** | the module made a change | yellow |
| **failed** | the module could not reach the desired state | red |
| **skipped** | a condition said not to run here (Chapter 8) | cyan |
| **unreachable** | Ansible could not connect to the host | red |

## Drift, and a dry run

Someone edits web2's copy of the notice by hand:

```
ansible web2 -m shell -a 'echo tampered >> /tmp/notice.txt'
ansible all -m copy -a 'content="Managed by Ansible. Do not edit by hand.\n" dest=/tmp/notice.txt' --check --diff
```

```
--- before: /tmp/notice.txt
+++ after: /tmp/notice.txt
@@ -1,2 +1 @@
 Managed by Ansible. Do not edit by hand.
-tampered
web2 | CHANGED => { ... }
web1 | SUCCESS => { ... }
db1 | SUCCESS => { ... }
```

`--check` (`-C`) is a **dry run**: each module reports what it *would* change without changing anything. `--diff` (`-D`) shows how, as a unified diff. Only web2 differs, and the extra line would go. Run the command again without `--check` to put it right.

<figure class="diagram">
<svg viewBox="0 0 720 282" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Detecting drift with check and diff. The desired file content is the same for all three hosts. web1 and db1 match and report SUCCESS. web2 has an extra line, tampered, added by hand; --check --diff reports CHANGED for web2 and shows the line that would be removed, without removing it." xmlns="http://www.w3.org/2000/svg">
<style>
.a3d{ font-family:var(--sans),sans-serif; }
.a3d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a3d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a3d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a3d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a3d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a3d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a3d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a3d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a3d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a3d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a3d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a3d .s{ fill:var(--text-dim); font-size:13px; }
.a3d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a3d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a3d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a3d .c{ fill:var(--text-faint); font-size:12.5px; }
.a3d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a3d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a3d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a3d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3d-a); }
.a3d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3d-a); marker-start:url(#a3d-s); }
.a3d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a3d-g); }
.a3d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a3d-a); }
.a3d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a3d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a3d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a3d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a3d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a3d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3d-tri); }
.a3d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3d-tri); }
.a3d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3d-open); }
.a3d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3d-open); }
.a3d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3d-dia); }
.a3d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3d-odia); }
.a3d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3d-dia); marker-end:url(#a3d-open); }
.a3d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3d-solid); }
.a3d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a3d-open); }
.a3d .fill{ fill:var(--text); stroke:none; }
.a3d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a3d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a3d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a3d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a3d .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a3d .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a3d .ks-amber{ fill:#854F0B; font-size:13px; }
.a3d .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a3d .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3d .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a3d .kb-amber, :root[data-theme="dark"] .a3d .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a3d .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a3d .kh-amber{ fill:#EF9F27; }
.a3d .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a3d-amber); }
.a3d .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3d-amber); }
.a3d .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a3d .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a3d .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a3d .ks-purple{ fill:#534AB7; font-size:13px; }
.a3d .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a3d .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3d .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a3d .kb-purple, :root[data-theme="dark"] .a3d .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a3d .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a3d .kh-purple{ fill:#AFA9EC; }
.a3d .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a3d-purple); }
.a3d .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3d-purple); }
.a3d .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a3d .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a3d .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a3d .ks-green{ fill:#3B6D11; font-size:13px; }
.a3d .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a3d .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3d .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a3d .kb-green, :root[data-theme="dark"] .a3d .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a3d .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a3d .kh-green{ fill:#97C459; }
.a3d .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a3d-green); }
.a3d .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3d-green); }
.a3d .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a3d .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a3d .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a3d .ks-red{ fill:#A32D2D; font-size:13px; }
.a3d .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a3d .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3d .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a3d .kb-red, :root[data-theme="dark"] .a3d .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a3d .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a3d .kh-red{ fill:#F09595; }
.a3d .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a3d-red); }
.a3d .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3d-red); }
.a3d .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a3d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a3d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a3d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a3d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a3d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a3d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a3d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a3d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a3d-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a3d-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a3d-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a3d-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a3d">
<rect x="20" y="20" width="220" height="74" rx="6" class="k-purple"/>
<text x="30" y="40" text-anchor="start" class="kh-purple">desired</text>
<text x="32.0" y="62" text-anchor="start" class="m">Managed by Ansible.</text>
<text x="32.0" y="80" text-anchor="start" class="m">Do not edit by hand.</text>
<rect x="290" y="16" width="250" height="72" rx="8" class="k-green"/>
<text x="302" y="34" text-anchor="start" class="kb-green">web1</text>
<text x="350" y="36" text-anchor="start" class="m">Managed by Ansible.</text>
<text x="350" y="54" text-anchor="start" class="m">Do not edit by hand.</text>
<path d="M242 50 L286 49" class="ag"/>
<text x="570" y="54" text-anchor="start" class="kh-green">SUCCESS</text>
<rect x="290" y="98" width="250" height="72" rx="8" class="k-red"/>
<text x="302" y="116" text-anchor="start" class="kb-red">web2</text>
<text x="350" y="118" text-anchor="start" class="m">Managed by Ansible.</text>
<text x="350" y="136" text-anchor="start" class="m">Do not edit by hand.</text>
<text x="350" y="154" text-anchor="start" class="kb-red">tampered</text>
<path d="M242 50 L286 131" class="ag"/>
<text x="570" y="136" text-anchor="start" class="kh-amber">CHANGED</text>
<rect x="290" y="180" width="250" height="72" rx="8" class="k-green"/>
<text x="302" y="198" text-anchor="start" class="kb-green">db1</text>
<text x="350" y="200" text-anchor="start" class="m">Managed by Ansible.</text>
<text x="350" y="218" text-anchor="start" class="m">Do not edit by hand.</text>
<path d="M242 50 L286 213" class="ag"/>
<text x="570" y="218" text-anchor="start" class="kh-green">SUCCESS</text>
<text x="360" y="274" text-anchor="middle" class="c">--check: change nothing   --diff: show what would change</text>
</g>
</svg>
<figcaption>Check mode compares reality with the description and reports every difference.</figcaption>
</figure>

Together they make Ansible a **drift detector**: run your description of the servers in check mode, and every `changed` is a place where reality no longer matches it.

## file: paths and their state

```
ansible web -b -m file -a 'path=/opt/app/releases/v1 state=directory owner=student mode=0755'
ansible web -m file -a 'src=/opt/app/releases/v1 dest=/opt/app/current state=link'
ansible web1 -a 'ls -la /opt/app'
ansible web -m file -a 'path=/tmp/notice.txt state=absent'
```

The **file** module sets a path's **state**: `directory`, `file` (must exist), `touch`, `link`, `hard`, or `absent` (removed, recursively for a folder). Owner, group and mode are part of the state, so they are put right if someone changes them. `-b` is needed to write in `/opt`.

<figure class="diagram">
<svg viewBox="0 0 720 210" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The file module&#x27;s states: directory creates a folder, touch creates an empty file or updates its time, link makes a symbolic link from dest to src, absent deletes the path. owner, group and mode apply to each." xmlns="http://www.w3.org/2000/svg">
<style>
.a3e{ font-family:var(--sans),sans-serif; }
.a3e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a3e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a3e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a3e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a3e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a3e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a3e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a3e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a3e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a3e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a3e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a3e .s{ fill:var(--text-dim); font-size:13px; }
.a3e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a3e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a3e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a3e .c{ fill:var(--text-faint); font-size:12.5px; }
.a3e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a3e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a3e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a3e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3e-a); }
.a3e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3e-a); marker-start:url(#a3e-s); }
.a3e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a3e-g); }
.a3e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a3e-a); }
.a3e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a3e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a3e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a3e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a3e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a3e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3e-tri); }
.a3e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3e-tri); }
.a3e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3e-open); }
.a3e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3e-open); }
.a3e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3e-dia); }
.a3e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3e-odia); }
.a3e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3e-dia); marker-end:url(#a3e-open); }
.a3e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3e-solid); }
.a3e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a3e-open); }
.a3e .fill{ fill:var(--text); stroke:none; }
.a3e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a3e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a3e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a3e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a3e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a3e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a3e .ks-blue{ fill:#185FA5; font-size:13px; }
.a3e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a3e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a3e .kb-blue, :root[data-theme="dark"] .a3e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a3e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a3e .kh-blue{ fill:#85B7EB; }
.a3e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a3e-blue); }
.a3e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3e-blue); }
.a3e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a3e .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a3e .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a3e .ks-teal{ fill:#0F6E56; font-size:13px; }
.a3e .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a3e .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3e .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a3e .kb-teal, :root[data-theme="dark"] .a3e .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a3e .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a3e .kh-teal{ fill:#5DCAA5; }
.a3e .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a3e-teal); }
.a3e .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3e-teal); }
.a3e .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a3e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a3e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a3e .ks-purple{ fill:#534AB7; font-size:13px; }
.a3e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a3e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a3e .kb-purple, :root[data-theme="dark"] .a3e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a3e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a3e .kh-purple{ fill:#AFA9EC; }
.a3e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a3e-purple); }
.a3e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3e-purple); }
.a3e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a3e .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a3e .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a3e .ks-red{ fill:#A32D2D; font-size:13px; }
.a3e .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a3e .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3e .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a3e .kb-red, :root[data-theme="dark"] .a3e .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a3e .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a3e .kh-red{ fill:#F09595; }
.a3e .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a3e-red); }
.a3e .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3e-red); }
.a3e .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a3e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a3e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a3e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a3e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a3e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a3e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a3e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a3e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a3e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a3e-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a3e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a3e-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a3e">
<rect x="20" y="20" width="160" height="44" rx="5" class="k-teal"/>
<text x="100.0" y="46.5" text-anchor="middle" class="km-teal">state=directory</text>
<path d="M100 66 L100 96" class="ar-teal"/>
<rect x="20" y="100" width="160" height="50" rx="7" class="pnl"/>
<text x="100.0" y="129.5" text-anchor="middle" class="m">/opt/app/releases/v1</text>
<rect x="192" y="20" width="160" height="44" rx="5" class="k-blue"/>
<text x="272.0" y="46.5" text-anchor="middle" class="km-blue">state=touch</text>
<path d="M272 66 L272 96" class="ar-blue"/>
<rect x="192" y="100" width="160" height="50" rx="7" class="pnl"/>
<text x="272.0" y="129.5" text-anchor="middle" class="m">/var/log/app.log</text>
<rect x="364" y="20" width="160" height="44" rx="5" class="k-purple"/>
<text x="444.0" y="46.5" text-anchor="middle" class="km-purple">state=link</text>
<path d="M444 66 L444 96" class="ar-purple"/>
<rect x="364" y="100" width="160" height="50" rx="7" class="pnl"/>
<text x="444.0" y="129.5" text-anchor="middle" class="m">current -&gt; v1</text>
<rect x="536" y="20" width="160" height="44" rx="5" class="k-red"/>
<text x="616.0" y="46.5" text-anchor="middle" class="km-red">state=absent</text>
<path d="M616 66 L616 96" class="ar-red"/>
<rect x="536" y="100" width="160" height="50" rx="7" class="pnl"/>
<text x="616.0" y="129.5" text-anchor="middle" class="m">(removed)</text>
<text x="360" y="186" text-anchor="middle" class="c">plus owner=, group=, mode=: part of the desired state, put right on every run</text>
</g>
</svg>
<figcaption>One module, several states.</figcaption>
</figure>

## apt: software

```
ansible all -b -m apt -a 'name=tree,jq state=present update_cache=yes'
ansible all -b -m apt -a 'name=tree,jq state=present'
ansible all -b -m apt -a 'name=jq state=absent'
```

`state=present` installs if missing, `absent` removes, `latest` upgrades. `update_cache=yes` runs `apt update` first; with `cache_valid_time=3600` it does so only when the cache is more than an hour old, which keeps repeated runs fast. The second command reports SUCCESS: both packages are already there. A script would run `apt install` every time. On Red Hat family systems the equivalent is `dnf`; the generic **package** module picks the right one.

## user and group

```
ansible all -b -m group -a 'name=deploy'
ansible all -b -m user -a 'name=deploy group=deploy shell=/bin/bash comment="Deployment account"'
ansible all -b -m user -a 'name=deploy groups=sudo append=true'
ansible all -a 'id deploy'
```

One command makes the same account on every server, with the same settings. With `groups=`, the list of extra groups **replaces** the user's current ones unless `append=true`, like the difference between `usermod -G` and `usermod -aG`. Mistakes here can lock people out, which is a good reason to try changes with `--check` first.

## lineinfile: one line in a big file

```
ansible all -b -m lineinfile -a 'path=/etc/ssh/sshd_config regexp="^#?PermitRootLogin" line="PermitRootLogin no"'
ansible all -a 'grep ^PermitRootLogin /etc/ssh/sshd_config'
```

**lineinfile** makes sure one line is present: the last line matching `regexp` is replaced by `line`, or, if nothing matches, `line` is added at the end. Run it again and it reports ok. It is ideal for one setting in a file owned by a package. For more than a few lines, manage the whole file with `copy` or `template` (Chapter 7), or a marked block with `blockinfile`.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="lineinfile with regexp ^#?PermitRootLogin and line PermitRootLogin no. Before: the file contains #PermitRootLogin prohibit-password; the regexp matches it and the line is replaced. If nothing matched, the line would be appended at the end. Running it again finds the line already right and reports ok." xmlns="http://www.w3.org/2000/svg">
<style>
.a3f{ font-family:var(--sans),sans-serif; }
.a3f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a3f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a3f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a3f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a3f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a3f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a3f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a3f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a3f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a3f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a3f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a3f .s{ fill:var(--text-dim); font-size:13px; }
.a3f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a3f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a3f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a3f .c{ fill:var(--text-faint); font-size:12.5px; }
.a3f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a3f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a3f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a3f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3f-a); }
.a3f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a3f-a); marker-start:url(#a3f-s); }
.a3f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a3f-g); }
.a3f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a3f-a); }
.a3f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a3f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a3f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a3f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a3f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a3f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3f-tri); }
.a3f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3f-tri); }
.a3f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3f-open); }
.a3f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a3f-open); }
.a3f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3f-dia); }
.a3f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3f-odia); }
.a3f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a3f-dia); marker-end:url(#a3f-open); }
.a3f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a3f-solid); }
.a3f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a3f-open); }
.a3f .fill{ fill:var(--text); stroke:none; }
.a3f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a3f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a3f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a3f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a3f .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a3f .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a3f .ks-teal{ fill:#0F6E56; font-size:13px; }
.a3f .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a3f .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3f .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a3f .kb-teal, :root[data-theme="dark"] .a3f .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a3f .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a3f .kh-teal{ fill:#5DCAA5; }
.a3f .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a3f-teal); }
.a3f .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3f-teal); }
.a3f .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a3f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a3f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a3f .ks-green{ fill:#3B6D11; font-size:13px; }
.a3f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a3f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a3f .kb-green, :root[data-theme="dark"] .a3f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a3f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a3f .kh-green{ fill:#97C459; }
.a3f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a3f-green); }
.a3f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3f-green); }
.a3f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a3f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a3f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a3f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a3f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a3f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a3f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a3f .kb-gray, :root[data-theme="dark"] .a3f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a3f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a3f .kh-gray{ fill:#B4B2A9; }
.a3f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a3f-gray); }
.a3f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a3f-gray); }
.a3f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a3f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a3f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a3f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a3f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a3f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a3f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a3f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a3f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a3f-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a3f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a3f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a3f">
<rect x="20" y="20" width="300" height="110" rx="6" class="k-gray"/>
<text x="30" y="40" text-anchor="start" class="kh-gray">before</text>
<text x="32.0" y="62" text-anchor="start" class="m">Port 22</text>
<text x="32.0" y="80" text-anchor="start" class="m">#PermitRootLogin prohibit-password</text>
<text x="32.0" y="98" text-anchor="start" class="m">PubkeyAuthentication yes</text>
<text x="32.0" y="116" text-anchor="start" class="m">...</text>
<rect x="400" y="20" width="300" height="110" rx="6" class="k-teal"/>
<text x="410" y="40" text-anchor="start" class="kh-teal">after</text>
<text x="412.0" y="62" text-anchor="start" class="m">Port 22</text>
<text x="412.0" y="80" text-anchor="start" class="kb-teal">PermitRootLogin no</text>
<text x="412.0" y="98" text-anchor="start" class="m">PubkeyAuthentication yes</text>
<text x="412.0" y="116" text-anchor="start" class="m">...</text>
<path d="M324 80 L396 80" class="ar-teal"/>
<text x="360" y="70" text-anchor="middle" class="kh-teal">replace</text>
<rect x="20" y="160" width="330" height="76" rx="7" class="pnl"/>
<text x="185.0" y="184.5" text-anchor="middle" class="b">regexp matches a line?</text>
<text x="185.0" y="202.5" text-anchor="middle" class="s">yes: replace the last match</text>
<text x="185.0" y="220.5" text-anchor="middle" class="s">no: add line at the end</text>
<rect x="380" y="160" width="320" height="76" rx="7" class="k-green"/>
<text x="540.0" y="184.5" text-anchor="middle" class="kb-green">run again</text>
<text x="540.0" y="202.5" text-anchor="middle" class="ks-green">the line is already there:</text>
<text x="540.0" y="220.5" text-anchor="middle" class="ks-green">ok, nothing changed</text>
</g>
</svg>
<figcaption>lineinfile owns one line; the rest of the file belongs to someone else.</figcaption>
</figure>

## service: daemons

```
ansible db1 -b -m apt -a 'name=cron install_recommends=no'
ansible db1 -b -m service -a 'name=cron state=started'
ansible db1 -a 'pgrep -a cron'
ansible db1 -b -m service -a 'name=cron state=stopped'
```

**service** starts, stops, restarts and reloads daemons (`state=started|stopped|restarted|reloaded`) and enables them at boot (`enabled=true`). On a normal server it talks to systemd. The lab containers have no systemd, so it falls back to the `service` command and the package's init script, transparently. `started` is idempotent; `restarted` always restarts, which is why Chapter 9 restarts services only when their configuration changed.

## Choosing a module

| You want to ... | Use | Not |
|---|---|---|
| install, remove, upgrade software | `apt`, `dnf`, `package` | `command: apt install` |
| put a whole file in place | `copy`, `template` | `shell: echo ... >` |
| set one line in a file | `lineinfile` | `shell: sed -i` |
| create folders, links, permissions | `file` | `command: mkdir` |
| manage accounts | `user`, `group`, `authorized_key` | `command: useradd` |
| start, stop, enable a service | `service`, `systemd` | `command: service ...` |
| check something exists | `stat` | `command: test -f` |
| call a web service | `uri`, `get_url` | `command: curl` |
| something no module does | `command`, `shell` | |

## Hands on

1. Copy a file *from* the servers: `ansible all -m fetch -a 'src=/etc/os-release dest=./collected'`. What folder layout does `fetch` create, and why?
2. Run `ansible all -m stat -a 'path=/etc/ssh/sshd_config'`. Which fields would let you detect a changed file?
3. Find a module to set the time zone (`ansible-doc -l | grep -i timezone`) and set every server to `America/Toronto`. Run it twice.
4. Which modules would you use to: install nginx; set `MaxAuthTries 3` in `sshd_config`; make sure `/srv/www` exists, owned by `www-data`?

## Summary

- **Modules** describe a desired state through parameters; `ansible-doc MODULE` documents each, with examples.
- `command` runs a program without a shell; `shell` uses `/bin/sh`. Both always report changed; prefer a dedicated module.
- **Idempotency**: a module checks first and changes only what differs, so results are **ok** or **changed**, and running again is safe.
- `--check` previews changes without making them; `--diff` shows them. Together they detect **drift**.
- Everyday modules: `copy`, `file`, `apt`, `user`, `group`, `lineinfile`, `service`, `stat`, `fetch`.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-3-modules-and-idempotency).

**1. How do the command and shell modules differ?**

- A. command runs a program without a shell; shell runs the line in /bin/sh, so pipes work
- B. command is faster
- C. shell is idempotent
- D. command needs become

**2. You run the same copy task twice. What does the second run report?**

- A. CHANGED
- B. FAILED
- C. SUCCESS / ok, with changed false
- D. SKIPPED

**3. What do --check and --diff do together?**

- A. Apply changes and print them
- B. Undo the last run
- C. Check SSH connectivity
- D. Report what would change, with diffs, without changing anything

**4. Which file module state removes a path?**

- A. state=directory
- B. state=absent
- C. state=touch
- D. state=link

**5. Why add cache_valid_time to an apt task with update_cache?**

- A. To pin package versions
- B. To clean old packages
- C. To run apt update only when the cache is older than that many seconds, keeping reruns fast
- D. To install recommended packages

**6. What happens with user: name=deploy groups=sudo without append=true?**

- A. The group list replaces the user's current extra groups
- B. sudo is added to the existing groups
- C. The task fails
- D. The user is removed from sudo

**7. What does lineinfile do when its regexp matches no line?**

- A. Adds the line at the end of the file
- B. Fails
- C. Replaces the first line
- D. Does nothing

**8. Which module should you use to check whether a file exists, without changing it?**

- A. file
- B. command: test -f
- C. stat
- D. copy
