# Chapter 14: Final Project: a Secure Web Stack

The last stage of Northwind's story brings everything together. The *Orders* application gets an internal **API server**, `api1`, that only the web servers may call. Customers reach only the **load balancer**. Every server gets a security baseline: an admin account with key login, **hardened SSH**, and a **host firewall** that lets through only what its job needs. The whole stack is built from bare servers by one run of `site.yml`, then **proved** correct by a second playbook, then deliberately broken and repaired. This is the shape of real infrastructure as code: roles, data, idempotent runs, and verification.

> **Try it:** DockerLab, *Ansible lab 12, Project: secure web stack*. Topology: control `10.0.0.10`, lb `10.0.0.20`, web1 `10.0.0.11`, web2 `10.0.0.12`, api1 `10.0.0.30`. Start with `setup-keys lb web1 web2 api1`. The finished project, exactly as the lab builds it, is in `examples/ch14`.

## The design

<figure class="diagram">
<svg viewBox="0 0 720 385" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The secure web stack. Customers reach only lb on port 80. lb reaches web1 and web2 on port 80. web1 and web2 reach api1 on port 8080. Control at 10.0.0.10 reaches every server by SSH, keys only. Every server has a host firewall that drops everything else; crossed arrows show blocked paths: customers to the web servers or the API, and lb to the API." xmlns="http://www.w3.org/2000/svg">
<style>
.a14a{ font-family:var(--sans),sans-serif; }
.a14a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14a .s{ fill:var(--text-dim); font-size:13px; }
.a14a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14a .c{ fill:var(--text-faint); font-size:12.5px; }
.a14a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14a-a); }
.a14a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14a-a); marker-start:url(#a14a-s); }
.a14a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14a-g); }
.a14a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-a); }
.a14a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14a-tri); }
.a14a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-tri); }
.a14a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14a-open); }
.a14a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-open); }
.a14a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14a-dia); }
.a14a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14a-odia); }
.a14a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14a-dia); marker-end:url(#a14a-open); }
.a14a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14a-solid); }
.a14a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14a-open); }
.a14a .fill{ fill:var(--text); stroke:none; }
.a14a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a14a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a14a .ks-blue{ fill:#185FA5; font-size:13px; }
.a14a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a14a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a14a .kb-blue, :root[data-theme="dark"] .a14a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a14a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a14a .kh-blue{ fill:#85B7EB; }
.a14a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a14a-blue); }
.a14a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-blue); }
.a14a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a14a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a14a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a14a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a14a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a14a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a14a .kb-teal, :root[data-theme="dark"] .a14a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a14a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a14a .kh-teal{ fill:#5DCAA5; }
.a14a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a14a-teal); }
.a14a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-teal); }
.a14a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a14a .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a14a .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a14a .ks-coral{ fill:#993C1D; font-size:13px; }
.a14a .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a14a .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14a .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a14a .kb-coral, :root[data-theme="dark"] .a14a .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a14a .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a14a .kh-coral{ fill:#F0997B; }
.a14a .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a14a-coral); }
.a14a .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-coral); }
.a14a .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a14a .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a14a .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a14a .ks-red{ fill:#A32D2D; font-size:13px; }
.a14a .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a14a .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14a .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a14a .kb-red, :root[data-theme="dark"] .a14a .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a14a .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a14a .kh-red{ fill:#F09595; }
.a14a .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a14a-red); }
.a14a .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-red); }
.a14a .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a14a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a14a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a14a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a14a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a14a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a14a .kb-gray, :root[data-theme="dark"] .a14a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a14a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a14a .kh-gray{ fill:#B4B2A9; }
.a14a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a14a-gray); }
.a14a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14a-gray); }
.a14a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a14a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a14a-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a14a-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a14a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a14a">
<rect x="20" y="20" width="150" height="56" rx="7" class="k-gray"/>
<text x="95.0" y="43.5" text-anchor="middle" class="kb-gray">customers</text>
<text x="95.0" y="61.5" text-anchor="middle" class="ks-gray">anywhere</text>
<rect x="280" y="20" width="160" height="54" rx="7" class="k-teal"/>
<line x1="290" y1="29" x2="306" y2="29" class="ln-teal"/>
<line x1="290" y1="34" x2="306" y2="34" class="ln-teal"/>
<text x="368.0" y="42" text-anchor="middle" class="kb-teal">lb</text>
<text x="360.0" y="61" text-anchor="middle" class="ks-teal">10.0.0.20  :80</text>
<rect x="180" y="150" width="160" height="54" rx="7" class="k-blue"/>
<line x1="190" y1="159" x2="206" y2="159" class="ln-blue"/>
<line x1="190" y1="164" x2="206" y2="164" class="ln-blue"/>
<text x="268.0" y="172" text-anchor="middle" class="kb-blue">web1</text>
<text x="260.0" y="191" text-anchor="middle" class="ks-blue">10.0.0.11  :80</text>
<rect x="380" y="150" width="160" height="54" rx="7" class="k-blue"/>
<line x1="390" y1="159" x2="406" y2="159" class="ln-blue"/>
<line x1="390" y1="164" x2="406" y2="164" class="ln-blue"/>
<text x="468.0" y="172" text-anchor="middle" class="kb-blue">web2</text>
<text x="460.0" y="191" text-anchor="middle" class="ks-blue">10.0.0.12  :80</text>
<rect x="280" y="280" width="160" height="54" rx="7" class="k-coral"/>
<line x1="290" y1="289" x2="306" y2="289" class="ln-coral"/>
<line x1="290" y1="294" x2="306" y2="294" class="ln-coral"/>
<text x="368.0" y="302" text-anchor="middle" class="kb-coral">api1</text>
<text x="360.0" y="321" text-anchor="middle" class="ks-coral">10.0.0.30  :8080</text>
<path d="M172 48 L276 48" class="ar-teal"/>
<text x="224" y="40" text-anchor="middle" class="kh-teal">:80</text>
<path d="M330 76 L262 146" class="ar-teal"/>
<path d="M390 76 L458 146" class="ar-teal"/>
<path d="M262 206 L330 276" class="ar-coral"/>
<path d="M458 206 L390 276" class="ar-coral"/>
<text x="360" y="248" text-anchor="middle" class="kh-coral">:8080</text>
<path d="M100 78 C 100 160, 120 180, 176 180" class="ad-red"/>
<text x="100" y="130" text-anchor="middle" class="kh-red">X</text>
<path d="M95 78 C 80 250, 160 310, 276 308" class="ad-red"/>
<text x="84" y="250" text-anchor="middle" class="kh-red">X</text>
<path d="M442 48 C 600 60, 620 260, 444 300" class="ad-red"/>
<text x="600" y="160" text-anchor="start" class="kh-red">X lb to API</text>
<rect x="560" y="300" width="150" height="54" rx="7" class="dark"/>
<text x="635.0" y="323" text-anchor="middle" class="on">control</text>
<text x="635.0" y="341" text-anchor="middle" class="on" style="fill:var(--bg);font-weight:400">SSH to all, keys only</text>
<rect x="270" y="16" width="180" height="70" rx="10" class="dash"/>
<rect x="170" y="146" width="180" height="70" rx="10" class="dash"/>
<rect x="370" y="146" width="180" height="70" rx="10" class="dash"/>
<rect x="270" y="276" width="180" height="70" rx="10" class="dash"/>
<text x="360" y="378" text-anchor="middle" class="c">dashed outline: each server's host firewall</text>
</g>
</svg>
<figcaption>Least privilege: every path that isn't needed is closed.</figcaption>
</figure>

| Traffic | Allowed |
|---|---|
| anyone to lb, port 80 | yes |
| lb to web1, web2, port 80 | yes; nobody else reaches the web servers directly |
| web1, web2 to api1, port 8080 | yes; nobody else reaches the API |
| control to every server, SSH | yes, with keys only; nobody else |

The design follows two security principles. **Least privilege**: each server accepts only the connections its role needs, from only the hosts that need them. **Defence in depth**: SSH is hardened even though the firewall already limits who can reach it, so one mistake doesn't open everything.

## The project

```
~/stack/
├── ansible.cfg
├── inventory                [loadbalancers] lb   [web] web1 web2   [api] api1
├── site.yml
├── verify.yml
├── group_vars/  all.yml  loadbalancers.yml  web.yml  api.yml
└── roles/
    ├── common/        the admin account, its key, sudo, banner
    ├── hardening/     SSH settings, validated
    ├── firewall/      iptables rules from data
    ├── webserver/     nginx, the site, a proxy to the API
    ├── loadbalancer/  nginx, balancing between the web servers
    └── api/           the internal API (a stand-in)
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Roles per group. Every server gets common, hardening and firewall. api1 also gets api; web1 and web2 get webserver; lb gets loadbalancer. Each group&#x27;s firewall data opens what its role needs." xmlns="http://www.w3.org/2000/svg">
<style>
.a14b{ font-family:var(--sans),sans-serif; }
.a14b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14b .s{ fill:var(--text-dim); font-size:13px; }
.a14b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .c{ fill:var(--text-faint); font-size:12.5px; }
.a14b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14b-a); }
.a14b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14b-a); marker-start:url(#a14b-s); }
.a14b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14b-g); }
.a14b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-a); }
.a14b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14b-tri); }
.a14b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-tri); }
.a14b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14b-open); }
.a14b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-open); }
.a14b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14b-dia); }
.a14b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14b-odia); }
.a14b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14b-dia); marker-end:url(#a14b-open); }
.a14b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14b-solid); }
.a14b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14b-open); }
.a14b .fill{ fill:var(--text); stroke:none; }
.a14b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a14b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a14b .ks-blue{ fill:#185FA5; font-size:13px; }
.a14b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a14b .kb-blue, :root[data-theme="dark"] .a14b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a14b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a14b .kh-blue{ fill:#85B7EB; }
.a14b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a14b-blue); }
.a14b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-blue); }
.a14b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a14b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a14b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a14b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a14b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a14b .kb-teal, :root[data-theme="dark"] .a14b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a14b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a14b .kh-teal{ fill:#5DCAA5; }
.a14b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a14b-teal); }
.a14b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-teal); }
.a14b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a14b .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a14b .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a14b .ks-amber{ fill:#854F0B; font-size:13px; }
.a14b .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14b .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a14b .kb-amber, :root[data-theme="dark"] .a14b .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a14b .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a14b .kh-amber{ fill:#EF9F27; }
.a14b .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a14b-amber); }
.a14b .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-amber); }
.a14b .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a14b .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a14b .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a14b .ks-coral{ fill:#993C1D; font-size:13px; }
.a14b .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14b .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a14b .kb-coral, :root[data-theme="dark"] .a14b .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a14b .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a14b .kh-coral{ fill:#F0997B; }
.a14b .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a14b-coral); }
.a14b .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-coral); }
.a14b .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a14b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a14b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a14b .ks-purple{ fill:#534AB7; font-size:13px; }
.a14b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a14b .kb-purple, :root[data-theme="dark"] .a14b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a14b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a14b .kh-purple{ fill:#AFA9EC; }
.a14b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a14b-purple); }
.a14b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-purple); }
.a14b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a14b .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a14b .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a14b .ks-red{ fill:#A32D2D; font-size:13px; }
.a14b .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a14b .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14b .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a14b .kb-red, :root[data-theme="dark"] .a14b .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a14b .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a14b .kh-red{ fill:#F09595; }
.a14b .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a14b-red); }
.a14b .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14b-red); }
.a14b .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a14b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a14b-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a14b-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a14b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a14b-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a14b">
<text x="100" y="28" text-anchor="middle" class="b">lb</text>
<rect x="20" y="40" width="160" height="34" rx="5" class="k-teal"/>
<text x="100.0" y="61.5" text-anchor="middle" class="kb-teal">loadbalancer</text>
<rect x="20" y="82" width="160" height="30" rx="5" class="k-red"/>
<text x="100.0" y="101.5" text-anchor="middle" class="ks-red">firewall: open: 80</text>
<rect x="20" y="120" width="160" height="30" rx="5" class="k-purple"/>
<text x="100.0" y="139.5" text-anchor="middle" class="ks-purple">hardening</text>
<rect x="20" y="158" width="160" height="30" rx="5" class="k-amber"/>
<text x="100.0" y="177.5" text-anchor="middle" class="ks-amber">common</text>
<text x="272" y="28" text-anchor="middle" class="b">web1</text>
<rect x="192" y="40" width="160" height="34" rx="5" class="k-blue"/>
<text x="272.0" y="61.5" text-anchor="middle" class="kb-blue">webserver</text>
<rect x="192" y="82" width="160" height="30" rx="5" class="k-red"/>
<text x="272.0" y="101.5" text-anchor="middle" class="ks-red">firewall: 80 from lb</text>
<rect x="192" y="120" width="160" height="30" rx="5" class="k-purple"/>
<text x="272.0" y="139.5" text-anchor="middle" class="ks-purple">hardening</text>
<rect x="192" y="158" width="160" height="30" rx="5" class="k-amber"/>
<text x="272.0" y="177.5" text-anchor="middle" class="ks-amber">common</text>
<text x="444" y="28" text-anchor="middle" class="b">web2</text>
<rect x="364" y="40" width="160" height="34" rx="5" class="k-blue"/>
<text x="444.0" y="61.5" text-anchor="middle" class="kb-blue">webserver</text>
<rect x="364" y="82" width="160" height="30" rx="5" class="k-red"/>
<text x="444.0" y="101.5" text-anchor="middle" class="ks-red">firewall: 80 from lb</text>
<rect x="364" y="120" width="160" height="30" rx="5" class="k-purple"/>
<text x="444.0" y="139.5" text-anchor="middle" class="ks-purple">hardening</text>
<rect x="364" y="158" width="160" height="30" rx="5" class="k-amber"/>
<text x="444.0" y="177.5" text-anchor="middle" class="ks-amber">common</text>
<text x="616" y="28" text-anchor="middle" class="b">api1</text>
<rect x="536" y="40" width="160" height="34" rx="5" class="k-coral"/>
<text x="616.0" y="61.5" text-anchor="middle" class="kb-coral">api</text>
<rect x="536" y="82" width="160" height="30" rx="5" class="k-red"/>
<text x="616.0" y="101.5" text-anchor="middle" class="ks-red">firewall: 8080 from web</text>
<rect x="536" y="120" width="160" height="30" rx="5" class="k-purple"/>
<text x="616.0" y="139.5" text-anchor="middle" class="ks-purple">hardening</text>
<rect x="536" y="158" width="160" height="30" rx="5" class="k-amber"/>
<text x="616.0" y="177.5" text-anchor="middle" class="ks-amber">common</text>
<text x="360" y="220" text-anchor="middle" class="c">baseline on every server (bottom three), one job on top</text>
</g>
</svg>
<figcaption>A security baseline everywhere, a job role per group.</figcaption>
</figure>

```yaml
---
- name: Security baseline, on every server
  hosts: all
  become: true
  roles:
    - common
    - hardening
    - firewall

- name: API
  hosts: api
  become: true
  roles:
    - api

- name: Web servers
  hosts: web
  become: true
  roles:
    - webserver

- name: Load balancer
  hosts: loadbalancers
  become: true
  roles:
    - loadbalancer
```

The baseline comes first, on every server. Then each group gets its own role. The order of the last three plays follows the dependencies: the API exists before the web servers proxy to it, and the web servers before the balancer sends them traffic.

## Role: common

```yaml
---
- name: Baseline packages
  ansible.builtin.apt:
    name: "{{ common_packages }}"
    update_cache: true
    cache_valid_time: 3600

- name: The admin account
  ansible.builtin.user:
    name: "{{ common_admin }}"
    groups: [sudo]
    append: true
    shell: /bin/bash

- name: The admin's SSH key
  ansible.posix.authorized_key:
    user: "{{ common_admin }}"
    key: "{{ common_admin_key }}"

- name: Passwordless sudo for the admin (keys are the only way in)
  ansible.builtin.copy:
    dest: /etc/sudoers.d/{{ common_admin }}
    content: "{{ common_admin }} ALL=(ALL) NOPASSWD: ALL\n"
    mode: "0440"
    validate: /usr/sbin/visudo -cf %s

- name: Login banner
  ansible.builtin.copy:
    dest: /etc/motd
    content: |
      {{ inventory_hostname }}: {{ group_names | join(', ') }} server. Managed by Ansible; authorised use only.
```

with `common_admin: opsadmin` and `common_admin_key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"` as defaults. A **lookup** runs on control: here it reads your public key, which `authorized_key`, from the `ansible.posix` collection, installs for *opsadmin* on every server. The sudoers file is validated, as in Chapter 7.

## Role: hardening

```
# roles/hardening/templates/hardening.conf.j2
# {{ ansible_managed }}
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
MaxAuthTries {{ hardening_ssh_max_auth_tries }}
X11Forwarding no
AllowGroups {{ hardening_ssh_allow_groups | join(' ') }}
```

```yaml
---
- name: Hardened SSH settings, checked by sshd before use
  ansible.builtin.template:
    src: hardening.conf.j2
    dest: /etc/ssh/sshd_config.d/50-hardening.conf
    mode: "0644"
    validate: /usr/sbin/sshd -t -f %s
  notify: Restart ssh
```

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="SSH hardening with a drop-in file. The package&#x27;s /etc/ssh/sshd_config includes /etc/ssh/sshd_config.d/*.conf; Ansible owns 50-hardening.conf there: no root login, no passwords, MaxAuthTries 3, AllowGroups sudo student. The template is validated with sshd -t before it is used, and the handler restarts ssh only when it changed." xmlns="http://www.w3.org/2000/svg">
<style>
.a14c{ font-family:var(--sans),sans-serif; }
.a14c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14c .s{ fill:var(--text-dim); font-size:13px; }
.a14c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14c .c{ fill:var(--text-faint); font-size:12.5px; }
.a14c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14c-a); }
.a14c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14c-a); marker-start:url(#a14c-s); }
.a14c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14c-g); }
.a14c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14c-a); }
.a14c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14c-tri); }
.a14c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14c-tri); }
.a14c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14c-open); }
.a14c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14c-open); }
.a14c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14c-dia); }
.a14c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14c-odia); }
.a14c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14c-dia); marker-end:url(#a14c-open); }
.a14c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14c-solid); }
.a14c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14c-open); }
.a14c .fill{ fill:var(--text); stroke:none; }
.a14c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a14c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a14c .ks-amber{ fill:#854F0B; font-size:13px; }
.a14c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a14c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a14c .kb-amber, :root[data-theme="dark"] .a14c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a14c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a14c .kh-amber{ fill:#EF9F27; }
.a14c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a14c-amber); }
.a14c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14c-amber); }
.a14c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a14c .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a14c .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a14c .ks-coral{ fill:#993C1D; font-size:13px; }
.a14c .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a14c .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14c .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a14c .kb-coral, :root[data-theme="dark"] .a14c .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a14c .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a14c .kh-coral{ fill:#F0997B; }
.a14c .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a14c-coral); }
.a14c .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14c-coral); }
.a14c .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a14c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a14c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a14c .ks-purple{ fill:#534AB7; font-size:13px; }
.a14c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a14c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a14c .kb-purple, :root[data-theme="dark"] .a14c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a14c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a14c .kh-purple{ fill:#AFA9EC; }
.a14c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a14c-purple); }
.a14c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14c-purple); }
.a14c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a14c .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a14c .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a14c .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a14c .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a14c .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14c .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a14c .kb-gray, :root[data-theme="dark"] .a14c .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a14c .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a14c .kh-gray{ fill:#B4B2A9; }
.a14c .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a14c-gray); }
.a14c .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14c-gray); }
.a14c .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a14c-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a14c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a14c-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a14c">
<rect x="10" y="20" width="310" height="70" rx="6" class="k-gray"/>
<text x="22.0" y="40" text-anchor="start" class="m"># /etc/ssh/sshd_config</text>
<text x="22.0" y="58" text-anchor="start" class="m">Include /etc/ssh/sshd_config.d/*.conf</text>
<text x="22.0" y="76" text-anchor="start" class="m">...  (the package's file)</text>
<rect x="350" y="20" width="360" height="128" rx="6" class="k-purple"/>
<text x="360" y="40" text-anchor="start" class="kh-purple">sshd_config.d/50-hardening.conf (Ansible)</text>
<text x="362.0" y="62" text-anchor="start" class="m">PermitRootLogin no</text>
<text x="362.0" y="80" text-anchor="start" class="m">PasswordAuthentication no</text>
<text x="362.0" y="98" text-anchor="start" class="m">KbdInteractiveAuthentication no</text>
<text x="362.0" y="116" text-anchor="start" class="m">MaxAuthTries 3</text>
<text x="362.0" y="134" text-anchor="start" class="m">AllowGroups sudo student</text>
<path d="M322 50 L346 60" class="ag"/>
<rect x="10" y="130" width="310" height="60" rx="7" class="k-amber"/>
<text x="165.0" y="155.5" text-anchor="middle" class="km-amber">validate: sshd -t -f %s</text>
<text x="165.0" y="173.5" text-anchor="middle" class="ks-amber">a typo fails the task, not SSH</text>
<rect x="350" y="180" width="360" height="40" rx="7" class="k-coral"/>
<text x="530.0" y="204.5" text-anchor="middle" class="km-coral">notify: Restart ssh  (only when changed)</text>
</g>
</svg>
<figcaption>Own a file, not a line; validate before using it.</figcaption>
</figure>

Debian's `sshd_config` includes every file in `sshd_config.d/`, and the first value set for a setting wins, so the role adds its **own file** instead of editing the package's. `sshd -t` checks the file before it is used, so a typo fails the task rather than locking everyone out, and the handler restarts SSH only when the settings changed. `AllowGroups` lists *sudo* (opsadmin) and *student*, so the lab's `setup-keys` still works after a Redeploy.

## Role: firewall

The firewall is the heart of the project. Each group says, as **data**, what it needs:

```yaml
# group_vars/loadbalancers.yml
firewall_open_ports: [80]

# group_vars/web.yml
firewall_allow:
  - port: 80
    hosts: "{{ groups['loadbalancers'] }}"

# group_vars/api.yml
firewall_allow:
  - port: 8080
    hosts: "{{ groups['web'] }}"
```

with defaults in the role: `firewall_admin_source: 10.0.0.10/32` (control), no open ports, and no allowed hosts. The role turns that data into `iptables` rules:

```yaml
---
- name: Allow loopback
  ansible.builtin.iptables:
    chain: INPUT
    in_interface: lo
    jump: ACCEPT

- name: Allow replies to connections this host made
  ansible.builtin.iptables:
    chain: INPUT
    ctstate: [ESTABLISHED, RELATED]
    jump: ACCEPT

- name: Allow ping
  ansible.builtin.iptables:
    chain: INPUT
    protocol: icmp
    icmp_type: echo-request
    jump: ACCEPT

- name: Allow SSH from the admin network only
  ansible.builtin.iptables:
    chain: INPUT
    protocol: tcp
    destination_port: "22"
    source: "{{ firewall_admin_source }}"
    jump: ACCEPT
    comment: admin SSH

- name: Know the addresses of the hosts in the rules, even with --limit
  ansible.builtin.setup:
    gather_subset: [network]
  delegate_to: "{{ item }}"
  delegate_facts: true
  loop: "{{ firewall_allow | map(attribute='hosts') | flatten | unique }}"
  when: hostvars[item]['ansible_facts']['eth0'] is not defined

- name: Open the ports anyone may reach
  ansible.builtin.iptables:
    chain: INPUT
    protocol: tcp
    destination_port: "{{ item }}"
    jump: ACCEPT
    comment: "open to all"
  loop: "{{ firewall_open_ports }}"

- name: Allow the ports some hosts may reach, from each host's address
  ansible.builtin.iptables:
    chain: INPUT
    protocol: tcp
    destination_port: "{{ item.0.port }}"
    source: "{{ hostvars[item.1]['ansible_facts']['eth0']['ipv4']['address'] }}"
    jump: ACCEPT
    comment: "from {{ item.1 }}"
  loop: "{{ firewall_allow | subelements('hosts') }}"
  loop_control:
    label: "{{ item.0.port }} from {{ item.1 }}"

- name: Drop everything else (last, once the allow rules are in)
  ansible.builtin.iptables:
    chain: INPUT
    policy: DROP
```

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="api1&#x27;s INPUT chain as built by the firewall role, in order: accept loopback; accept established and related; accept ICMP echo request; accept TCP 22 from 10.0.0.10; accept TCP 8080 from 10.0.0.11 (web1); accept TCP 8080 from 10.0.0.12 (web2); then the default policy DROP for everything else. The DROP policy is set last." xmlns="http://www.w3.org/2000/svg">
<style>
.a14d{ font-family:var(--sans),sans-serif; }
.a14d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14d .s{ fill:var(--text-dim); font-size:13px; }
.a14d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14d .c{ fill:var(--text-faint); font-size:12.5px; }
.a14d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14d-a); }
.a14d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14d-a); marker-start:url(#a14d-s); }
.a14d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14d-g); }
.a14d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14d-a); }
.a14d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14d-tri); }
.a14d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14d-tri); }
.a14d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14d-open); }
.a14d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14d-open); }
.a14d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14d-dia); }
.a14d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14d-odia); }
.a14d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14d-dia); marker-end:url(#a14d-open); }
.a14d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14d-solid); }
.a14d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14d-open); }
.a14d .fill{ fill:var(--text); stroke:none; }
.a14d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a14d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a14d .ks-blue{ fill:#185FA5; font-size:13px; }
.a14d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a14d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a14d .kb-blue, :root[data-theme="dark"] .a14d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a14d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a14d .kh-blue{ fill:#85B7EB; }
.a14d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a14d-blue); }
.a14d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14d-blue); }
.a14d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a14d .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a14d .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a14d .ks-green{ fill:#3B6D11; font-size:13px; }
.a14d .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a14d .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14d .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a14d .kb-green, :root[data-theme="dark"] .a14d .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a14d .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a14d .kh-green{ fill:#97C459; }
.a14d .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a14d-green); }
.a14d .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14d-green); }
.a14d .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a14d .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a14d .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a14d .ks-red{ fill:#A32D2D; font-size:13px; }
.a14d .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a14d .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14d .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a14d .kb-red, :root[data-theme="dark"] .a14d .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a14d .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a14d .kh-red{ fill:#F09595; }
.a14d .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a14d-red); }
.a14d .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14d-red); }
.a14d .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a14d-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a14d-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a14d">
<text x="30" y="36" text-anchor="middle" class="c">1</text>
<rect x="50" y="16" width="300" height="30" rx="4" class="k-green"/>
<text x="200.0" y="35.5" text-anchor="middle" class="km-green">lo</text>
<text x="370" y="36" text-anchor="start" class="s">ACCEPT  loopback</text>
<text x="30" y="74" text-anchor="middle" class="c">2</text>
<rect x="50" y="54" width="300" height="30" rx="4" class="k-green"/>
<text x="200.0" y="73.5" text-anchor="middle" class="km-green">ESTABLISHED, RELATED</text>
<text x="370" y="74" text-anchor="start" class="s">ACCEPT  replies to its own connections</text>
<text x="30" y="112" text-anchor="middle" class="c">3</text>
<rect x="50" y="92" width="300" height="30" rx="4" class="k-green"/>
<text x="200.0" y="111.5" text-anchor="middle" class="km-green">icmp echo-request</text>
<text x="370" y="112" text-anchor="start" class="s">ACCEPT  ping</text>
<text x="30" y="150" text-anchor="middle" class="c">4</text>
<rect x="50" y="130" width="300" height="30" rx="4" class="k-green"/>
<text x="200.0" y="149.5" text-anchor="middle" class="km-green">tcp 22 from 10.0.0.10</text>
<text x="370" y="150" text-anchor="start" class="s">ACCEPT  admin SSH (control)</text>
<text x="30" y="188" text-anchor="middle" class="c">5</text>
<rect x="50" y="168" width="300" height="30" rx="4" class="k-blue"/>
<text x="200.0" y="187.5" text-anchor="middle" class="km-blue">tcp 8080 from 10.0.0.11</text>
<text x="370" y="188" text-anchor="start" class="s">ACCEPT  from web1 (facts)</text>
<text x="30" y="226" text-anchor="middle" class="c">6</text>
<rect x="50" y="206" width="300" height="30" rx="4" class="k-blue"/>
<text x="200.0" y="225.5" text-anchor="middle" class="km-blue">tcp 8080 from 10.0.0.12</text>
<text x="370" y="226" text-anchor="start" class="s">ACCEPT  from web2 (facts)</text>
<rect x="50" y="250" width="300" height="36" rx="4" class="k-red"/>
<text x="200.0" y="272.5" text-anchor="middle" class="km-red">policy DROP</text>
<text x="370" y="273" text-anchor="start" class="kh-red">everything else; set LAST</text>
</g>
</svg>
<figcaption>api1's INPUT chain: allow what's needed, in order, then drop the rest.</figcaption>
</figure>

Four ideas from earlier chapters meet here:

- **Order matters.** The allow rules go in first and the `DROP` policy last. The other way round, Ansible's own SSH connection would be cut halfway through the run.
- **Idempotency.** The `iptables` module adds a rule only if an identical one isn't there, so the second run changes nothing.
- **Data to rules.** `subelements('hosts')` pairs each entry with each of its hosts, giving one rule per port and host. Each host's address comes from its **facts**.
- **Facts across hosts, even with `--limit`.** Those facts exist only for hosts in the current run. With `--limit web2`, lb's facts would be missing, so the role first gathers them where needed: `delegate_to` the other host, and `delegate_facts: true` to store them as **that** host's facts, which solves Chapter 7's problem.

<figure class="diagram">
<svg viewBox="0 0 720 240" style="width:100%;height:auto;max-width:720px" role="img" aria-label="From data to rules on api1. firewall_allow has one entry, port 8080 with hosts groups web, which is web1 and web2. subelements(&#x27;hosts&#x27;) produces two items: (8080, web1) and (8080, web2). Each becomes a rule with the host&#x27;s address from hostvars facts: 10.0.0.11 and 10.0.0.12." xmlns="http://www.w3.org/2000/svg">
<style>
.a14e{ font-family:var(--sans),sans-serif; }
.a14e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14e .s{ fill:var(--text-dim); font-size:13px; }
.a14e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14e .c{ fill:var(--text-faint); font-size:12.5px; }
.a14e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14e-a); }
.a14e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14e-a); marker-start:url(#a14e-s); }
.a14e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14e-g); }
.a14e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14e-a); }
.a14e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14e-tri); }
.a14e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14e-tri); }
.a14e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14e-open); }
.a14e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14e-open); }
.a14e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14e-dia); }
.a14e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14e-odia); }
.a14e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14e-dia); marker-end:url(#a14e-open); }
.a14e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14e-solid); }
.a14e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14e-open); }
.a14e .fill{ fill:var(--text); stroke:none; }
.a14e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a14e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a14e .ks-blue{ fill:#185FA5; font-size:13px; }
.a14e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a14e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a14e .kb-blue, :root[data-theme="dark"] .a14e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a14e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a14e .kh-blue{ fill:#85B7EB; }
.a14e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a14e-blue); }
.a14e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14e-blue); }
.a14e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a14e .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a14e .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a14e .ks-teal{ fill:#0F6E56; font-size:13px; }
.a14e .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a14e .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14e .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a14e .kb-teal, :root[data-theme="dark"] .a14e .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a14e .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a14e .kh-teal{ fill:#5DCAA5; }
.a14e .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a14e-teal); }
.a14e .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14e-teal); }
.a14e .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a14e .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a14e .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a14e .ks-amber{ fill:#854F0B; font-size:13px; }
.a14e .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a14e .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14e .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a14e .kb-amber, :root[data-theme="dark"] .a14e .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a14e .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a14e .kh-amber{ fill:#EF9F27; }
.a14e .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a14e-amber); }
.a14e .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14e-amber); }
.a14e .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a14e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a14e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a14e .ks-purple{ fill:#534AB7; font-size:13px; }
.a14e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a14e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a14e .kb-purple, :root[data-theme="dark"] .a14e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a14e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a14e .kh-purple{ fill:#AFA9EC; }
.a14e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a14e-purple); }
.a14e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14e-purple); }
.a14e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a14e-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a14e-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a14e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a14e">
<rect x="10" y="30" width="280" height="110" rx="6" class="k-amber"/>
<text x="20" y="50" text-anchor="start" class="kh-amber">group_vars/api.yml</text>
<text x="22.0" y="72" text-anchor="start" class="m">firewall_allow:</text>
<text x="37.2" y="90" text-anchor="start" class="m">- port: 8080</text>
<text x="52.4" y="108" text-anchor="start" class="m">hosts: "{{ groups['web'] }}"</text>
<text x="52.4" y="126" text-anchor="start" class="m">#  = [web1, web2]</text>
<rect x="306" y="40" width="130" height="70" rx="7" class="k-purple"/>
<text x="371.0" y="70.0" text-anchor="middle" class="km-purple">subelements(</text>
<text x="371.0" y="89.0" text-anchor="middle" class="km-purple">'hosts')</text>
<path d="M292 75 L302 75" class="ar"/>
<rect x="470" y="20" width="230" height="70" rx="7" class="k-blue"/>
<text x="585.0" y="41.5" text-anchor="middle" class="km-blue">item = (8080, web1)</text>
<text x="585.0" y="59.5" text-anchor="middle" class="km-blue">source: 10.0.0.11</text>
<text x="585.0" y="77.5" text-anchor="middle" class="ks-blue">from hostvars facts</text>
<path d="M438 75 L466 55" class="ar-purple"/>
<rect x="470" y="110" width="230" height="70" rx="7" class="k-blue"/>
<text x="585.0" y="131.5" text-anchor="middle" class="km-blue">item = (8080, web2)</text>
<text x="585.0" y="149.5" text-anchor="middle" class="km-blue">source: 10.0.0.12</text>
<text x="585.0" y="167.5" text-anchor="middle" class="ks-blue">from hostvars facts</text>
<path d="M438 75 L466 145" class="ar-purple"/>
<text x="360" y="226" text-anchor="middle" class="kh-teal">add web3 to the inventory: a third rule appears on the next run</text>
</g>
</svg>
<figcaption>Rules are generated from groups and facts, never typed.</figcaption>
</figure>

Add a web server to the inventory, run `site.yml`, and the API's firewall lets it in, without anyone writing a rule. (Rules set this way last until the container restarts; a real server would save them with the `iptables-persistent` package, in a handler.)

## Roles: webserver, loadbalancer, api

The application roles are short, because the earlier chapters did the hard work. The web servers' site proxies `/api/` to the API, found through the inventory:

```
server {
    listen 80 default_server;
    root /var/www/html;
    location = /health { return 200 "OK\n"; }
    location /api/ { proxy_pass http://{{ groups['api'][0] }}:8080/; }
}
```

The load balancer's backends come from the web servers' **facts**:

```
upstream app {
{% for h in groups['web'] %}
    server {{ hostvars[h]['ansible_facts']['eth0']['ipv4']['address'] }}:80;
{% endfor %}
}
server {
    listen 80 default_server;
    location / { proxy_pass http://app; }
}
```

and the API is a stand-in, nginx returning fixed JSON on port 8080, so the lab needs no application code:

```
server {
    listen {{ api_port }};
    default_type application/json;
    location = /orders { return 200 '{"orders": 42, "served_by": "{{ inventory_hostname }}"}'; }
    location / { return 404 '{"error": "no such endpoint"}'; }
}
```

A real API would be a service in Python, Java or Node behind exactly the same firewall rules. Nothing in any role names an address: everything comes from the inventory and facts.

## Build it

```
time ansible-playbook site.yml | tail -8
```

```
PLAY RECAP *********************************************************************
api1   : ok=20   changed=18   unreachable=0    failed=0    skipped=2 ...
lb     : ok=19   changed=17   unreachable=0    failed=0    skipped=2 ...
web1   : ok=20   changed=18   unreachable=0    failed=0    skipped=2 ...
web2   : ok=20   changed=18   unreachable=0    failed=0    skipped=2 ...

real	0m35s
```

Four bare servers to a working, hardened, firewalled stack in one command, most of it spent installing packages. Now the real test:

```
ansible-playbook site.yml | tail -6
```

```
api1   : ok=18   changed=0    unreachable=0    failed=0    skipped=2 ...
lb     : ok=17   changed=0    unreachable=0    failed=0    skipped=2 ...
web1   : ok=18   changed=0    unreachable=0    failed=0    skipped=2 ...
web2   : ok=18   changed=0    unreachable=0    failed=0    skipped=2 ...
```

`changed=0` everywhere: the whole stack is **idempotent**. That is what lets you run it every night to undo drift, and in a pipeline after every change.

## Prove it: a verification playbook

Configuration describes what **should** be; verification checks what **is**, from where it matters. `verify.yml` tests the stack from three viewpoints:

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="verify.yml checks from three viewpoints. From control, outside: the site and the API through lb work; the web servers and the API directly do not; password login is refused. From the web servers: the API answers. From the load balancer: the API does not answer." xmlns="http://www.w3.org/2000/svg">
<style>
.a14f{ font-family:var(--sans),sans-serif; }
.a14f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14f .s{ fill:var(--text-dim); font-size:13px; }
.a14f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14f .c{ fill:var(--text-faint); font-size:12.5px; }
.a14f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14f-a); }
.a14f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14f-a); marker-start:url(#a14f-s); }
.a14f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14f-g); }
.a14f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-a); }
.a14f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14f-tri); }
.a14f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-tri); }
.a14f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14f-open); }
.a14f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-open); }
.a14f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14f-dia); }
.a14f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14f-odia); }
.a14f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14f-dia); marker-end:url(#a14f-open); }
.a14f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14f-solid); }
.a14f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14f-open); }
.a14f .fill{ fill:var(--text); stroke:none; }
.a14f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a14f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a14f .ks-blue{ fill:#185FA5; font-size:13px; }
.a14f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a14f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a14f .kb-blue, :root[data-theme="dark"] .a14f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a14f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a14f .kh-blue{ fill:#85B7EB; }
.a14f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a14f-blue); }
.a14f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-blue); }
.a14f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a14f .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a14f .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a14f .ks-teal{ fill:#0F6E56; font-size:13px; }
.a14f .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a14f .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14f .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a14f .kb-teal, :root[data-theme="dark"] .a14f .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a14f .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a14f .kh-teal{ fill:#5DCAA5; }
.a14f .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a14f-teal); }
.a14f .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-teal); }
.a14f .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a14f .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a14f .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a14f .ks-purple{ fill:#534AB7; font-size:13px; }
.a14f .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a14f .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14f .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a14f .kb-purple, :root[data-theme="dark"] .a14f .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a14f .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a14f .kh-purple{ fill:#AFA9EC; }
.a14f .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a14f-purple); }
.a14f .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-purple); }
.a14f .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a14f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a14f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a14f .ks-green{ fill:#3B6D11; font-size:13px; }
.a14f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a14f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a14f .kb-green, :root[data-theme="dark"] .a14f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a14f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a14f .kh-green{ fill:#97C459; }
.a14f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a14f-green); }
.a14f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-green); }
.a14f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a14f .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a14f .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a14f .ks-red{ fill:#A32D2D; font-size:13px; }
.a14f .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a14f .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14f .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a14f .kb-red, :root[data-theme="dark"] .a14f .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a14f .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a14f .kh-red{ fill:#F09595; }
.a14f .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a14f-red); }
.a14f .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14f-red); }
.a14f .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a14f-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a14f-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a14f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a14f-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a14f">
<rect x="20" y="16" width="260" height="236" rx="10" class="k-purple"/>
<text x="150.0" y="40" text-anchor="middle" class="kb-purple">from control (outside)</text>
<rect x="32" y="56" width="236" height="30" rx="4" class="k-green"/>
<text x="150.0" y="75.5" text-anchor="middle" class="ks-green">works: http://lb/ shows the site</text>
<rect x="32" y="94" width="236" height="30" rx="4" class="k-green"/>
<text x="150.0" y="113.5" text-anchor="middle" class="ks-green">works: lb/api/orders = 42</text>
<rect x="32" y="132" width="236" height="30" rx="4" class="k-red"/>
<text x="150.0" y="151.5" text-anchor="middle" class="ks-red">refused: web1, web2 :80 directly</text>
<rect x="32" y="170" width="236" height="30" rx="4" class="k-red"/>
<text x="150.0" y="189.5" text-anchor="middle" class="ks-red">refused: api1 :8080 directly</text>
<rect x="32" y="208" width="236" height="30" rx="4" class="k-red"/>
<text x="150.0" y="227.5" text-anchor="middle" class="ks-red">refused: password SSH login</text>
<rect x="300" y="16" width="200" height="236" rx="10" class="k-blue"/>
<text x="400.0" y="40" text-anchor="middle" class="kb-blue">from web1, web2</text>
<rect x="312" y="56" width="176" height="30" rx="4" class="k-green"/>
<text x="400.0" y="75.5" text-anchor="middle" class="ks-green">works: api1:8080/orders</text>
<rect x="520" y="16" width="200" height="236" rx="10" class="k-teal"/>
<text x="620.0" y="40" text-anchor="middle" class="kb-teal">from lb</text>
<rect x="532" y="56" width="176" height="30" rx="4" class="k-red"/>
<text x="620.0" y="75.5" text-anchor="middle" class="ks-red">refused: api1:8080</text>
</g>
</svg>
<figcaption>Test what should work and what should fail, from where it matters.</figcaption>
</figure>

```yaml
- name: From outside (control)
  hosts: localhost
  gather_facts: false
  tasks:
    - name: The site works through the load balancer
      ansible.builtin.uri:
        url: http://lb/
        return_content: true
      register: site
      failed_when: "'Northwind Orders' not in site.content"

    - name: The API answers through the load balancer and a web server
      ansible.builtin.uri:
        url: http://lb/api/orders
        return_content: true
      register: api
      failed_when: api.json.orders != 42

    - name: The web servers can't be reached directly
      ansible.builtin.wait_for:
        host: "{{ item }}"
        port: 80
        timeout: 3
      loop: "{{ groups['web'] }}"
      register: direct
      ignore_errors: true

    ...

    - name: Verdict, from outside
      ansible.builtin.assert:
        that:
          - direct.results | selectattr('failed') | list | length == groups['web'] | length
          - api_direct is failed
          - password_login.rc != 0
        success_msg: "OK: only the load balancer is reachable, and SSH takes keys only"
```

The full file adds a check that the API refuses direct connections from control, that password logins are refused (`sshpass` with `PubkeyAuthentication=no`), that each web server **can** call the API, and that the load balancer **cannot**.

```
ansible-playbook verify.yml
```

```
"msg": "OK: only the load balancer is reachable, and SSH takes keys only"
...
localhost  : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2
```

Tests that something **fails**, a blocked port or a refused password, matter as much as tests that something works: a firewall that lets everything through would pass every positive test. The two *ignored* results are the expected failures, judged by the final `assert`.

## Break it, and let Ansible repair it

Someone opens web2's firewall, removes its SSH hardening and defaces its page:

```
ssh web2 'sudo iptables -P INPUT ACCEPT; sudo rm /etc/ssh/sshd_config.d/50-hardening.conf; echo hacked | sudo tee /var/www/html/index.html > /dev/null'
ansible-playbook verify.yml
```

```
fatal: [localhost]: FAILED! => {... "content": "hacked\n", ... "msg": "OK (7 bytes)", "status": 200, "url": "http://lb/"}
```

```
ansible-playbook site.yml --limit web2
```

```
changed: [web2]
changed: [web2]
RUNNING HANDLER [hardening : Restart ssh] **************************************
changed: [web2]
web2   : ok=20   changed=4    unreachable=0    failed=0 ...
```

```
ansible-playbook verify.yml
"msg": "OK: only the load balancer is reachable, and SSH takes keys only"
```

<figure class="diagram">
<svg viewBox="0 0 720 210" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The repair loop. Someone changes web2 by hand: firewall opened, SSH hardening removed, page defaced. verify.yml fails. site.yml --limit web2 restores the page, the SSH file, the firewall policy and restarts ssh through the handler. verify.yml passes again." xmlns="http://www.w3.org/2000/svg">
<style>
.a14g{ font-family:var(--sans),sans-serif; }
.a14g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14g .s{ fill:var(--text-dim); font-size:13px; }
.a14g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14g .c{ fill:var(--text-faint); font-size:12.5px; }
.a14g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14g-a); }
.a14g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14g-a); marker-start:url(#a14g-s); }
.a14g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14g-g); }
.a14g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14g-a); }
.a14g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14g-tri); }
.a14g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14g-tri); }
.a14g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14g-open); }
.a14g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14g-open); }
.a14g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14g-dia); }
.a14g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14g-odia); }
.a14g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14g-dia); marker-end:url(#a14g-open); }
.a14g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14g-solid); }
.a14g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14g-open); }
.a14g .fill{ fill:var(--text); stroke:none; }
.a14g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14g .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a14g .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a14g .ks-teal{ fill:#0F6E56; font-size:13px; }
.a14g .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a14g .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14g .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a14g .kb-teal, :root[data-theme="dark"] .a14g .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a14g .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a14g .kh-teal{ fill:#5DCAA5; }
.a14g .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a14g-teal); }
.a14g .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14g-teal); }
.a14g .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a14g .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a14g .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a14g .ks-amber{ fill:#854F0B; font-size:13px; }
.a14g .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a14g .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14g .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a14g .kb-amber, :root[data-theme="dark"] .a14g .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a14g .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a14g .kh-amber{ fill:#EF9F27; }
.a14g .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a14g-amber); }
.a14g .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14g-amber); }
.a14g .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a14g .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a14g .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a14g .ks-green{ fill:#3B6D11; font-size:13px; }
.a14g .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a14g .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14g .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a14g .kb-green, :root[data-theme="dark"] .a14g .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a14g .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a14g .kh-green{ fill:#97C459; }
.a14g .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a14g-green); }
.a14g .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14g-green); }
.a14g .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a14g .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a14g .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a14g .ks-red{ fill:#A32D2D; font-size:13px; }
.a14g .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a14g .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14g .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a14g .kb-red, :root[data-theme="dark"] .a14g .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a14g .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a14g .kh-red{ fill:#F09595; }
.a14g .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a14g-red); }
.a14g .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14g-red); }
.a14g .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14g-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a14g-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a14g-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a14g-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a14g">
<rect x="14" y="50" width="164" height="64" rx="7" class="k-red"/>
<text x="96.0" y="77.5" text-anchor="middle" class="kb-red">web2 edited</text>
<text x="96.0" y="95.5" text-anchor="middle" class="ks-red">by hand</text>
<path d="M180 82 L190 82" class="ar"/>
<rect x="192" y="50" width="164" height="64" rx="7" class="k-red"/>
<text x="274.0" y="77.5" text-anchor="middle" class="kb-red">verify.yml</text>
<text x="274.0" y="95.5" text-anchor="middle" class="ks-red">FAILED</text>
<path d="M358 82 L368 82" class="ar"/>
<rect x="370" y="50" width="164" height="64" rx="7" class="k-amber"/>
<text x="452.0" y="77.5" text-anchor="middle" class="kb-amber">site.yml</text>
<text x="452.0" y="95.5" text-anchor="middle" class="ks-amber">--limit web2</text>
<path d="M536 82 L546 82" class="ar"/>
<rect x="548" y="50" width="164" height="64" rx="7" class="k-green"/>
<text x="630.0" y="77.5" text-anchor="middle" class="kb-green">verify.yml</text>
<text x="630.0" y="95.5" text-anchor="middle" class="ks-green">OK</text>
<text x="360" y="160" text-anchor="middle" class="kh-teal">the playbooks are the truth; servers are brought back to it</text>
</g>
</svg>
<figcaption>Detect drift, repair it, prove the repair.</figcaption>
</figure>

`verify.yml` caught the change; `site.yml --limit web2` put back every piece (the page, the SSH file and, through its handler, the SSH restart, and the firewall policy); `verify.yml` passes again. The run with `--limit` works because the firewall role gathers the load balancer's facts itself. This is configuration management in one picture: **the playbooks are the truth, and servers are brought back to it.**

## Infrastructure as code, all the way

The last step is to stop running these commands by hand. Keep the project in Git, and let a **pipeline** run the same steps on every change, the way Chapter 13 of the *Concise Book on Software Engineering* builds one for application code:

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A pipeline for the infrastructure repository. A merge request triggers lint and syntax check, then site.yml in check mode with diff for review. After merge, site.yml runs on staging, then on production, followed by verify.yml. Every night, a check-mode run reports drift." xmlns="http://www.w3.org/2000/svg">
<style>
.a14h{ font-family:var(--sans),sans-serif; }
.a14h .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a14h .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a14h .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a14h .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a14h .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a14h .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a14h .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a14h .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a14h .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a14h .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a14h .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a14h .s{ fill:var(--text-dim); font-size:13px; }
.a14h .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a14h .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a14h .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a14h .c{ fill:var(--text-faint); font-size:12.5px; }
.a14h .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a14h .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a14h .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a14h .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14h-a); }
.a14h .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a14h-a); marker-start:url(#a14h-s); }
.a14h .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a14h-g); }
.a14h .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-a); }
.a14h .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a14h .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a14h .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a14h .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a14h .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a14h .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14h-tri); }
.a14h .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-tri); }
.a14h .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14h-open); }
.a14h .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-open); }
.a14h .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14h-dia); }
.a14h .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14h-odia); }
.a14h .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a14h-dia); marker-end:url(#a14h-open); }
.a14h .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a14h-solid); }
.a14h .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a14h-open); }
.a14h .fill{ fill:var(--text); stroke:none; }
.a14h .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a14h .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a14h .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a14h .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a14h .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a14h .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a14h .ks-blue{ fill:#185FA5; font-size:13px; }
.a14h .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a14h .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14h .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a14h .kb-blue, :root[data-theme="dark"] .a14h .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a14h .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a14h .kh-blue{ fill:#85B7EB; }
.a14h .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a14h-blue); }
.a14h .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-blue); }
.a14h .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a14h .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a14h .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a14h .ks-amber{ fill:#854F0B; font-size:13px; }
.a14h .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a14h .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14h .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a14h .kb-amber, :root[data-theme="dark"] .a14h .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a14h .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a14h .kh-amber{ fill:#EF9F27; }
.a14h .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a14h-amber); }
.a14h .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-amber); }
.a14h .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a14h .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a14h .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a14h .ks-purple{ fill:#534AB7; font-size:13px; }
.a14h .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a14h .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14h .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a14h .kb-purple, :root[data-theme="dark"] .a14h .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a14h .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a14h .kh-purple{ fill:#AFA9EC; }
.a14h .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a14h-purple); }
.a14h .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-purple); }
.a14h .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a14h .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a14h .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a14h .ks-green{ fill:#3B6D11; font-size:13px; }
.a14h .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a14h .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14h .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a14h .kb-green, :root[data-theme="dark"] .a14h .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a14h .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a14h .kh-green{ fill:#97C459; }
.a14h .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a14h-green); }
.a14h .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-green); }
.a14h .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a14h .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a14h .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a14h .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a14h .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a14h .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a14h .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a14h .kb-gray, :root[data-theme="dark"] .a14h .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a14h .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a14h .kh-gray{ fill:#B4B2A9; }
.a14h .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a14h-gray); }
.a14h .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a14h-gray); }
.a14h .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a14h-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a14h-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a14h-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a14h-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a14h-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a14h-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a14h-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a14h-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a14h-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a14h-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a14h-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a14h-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a14h-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a14h">
<rect x="14" y="40" width="126" height="60" rx="7" class="k-gray"/>
<text x="77.0" y="65.5" text-anchor="middle" class="kb-gray">merge request</text>
<text x="77.0" y="83.5" text-anchor="middle" class="ks-gray">Git</text>
<path d="M142 70 L152 70" class="ar"/>
<rect x="154" y="40" width="126" height="60" rx="7" class="k-amber"/>
<text x="217.0" y="65.5" text-anchor="middle" class="kb-amber">ansible-lint</text>
<text x="217.0" y="83.5" text-anchor="middle" class="km-amber">--syntax-check</text>
<path d="M282 70 L292 70" class="ar"/>
<rect x="294" y="40" width="126" height="60" rx="7" class="k-amber"/>
<text x="357.0" y="65.5" text-anchor="middle" class="kb-amber">site.yml</text>
<text x="357.0" y="83.5" text-anchor="middle" class="km-amber">--check --diff</text>
<path d="M422 70 L432 70" class="ar"/>
<rect x="434" y="40" width="126" height="60" rx="7" class="k-blue"/>
<text x="497.0" y="65.5" text-anchor="middle" class="kb-blue">apply</text>
<text x="497.0" y="83.5" text-anchor="middle" class="ks-blue">staging, prod</text>
<path d="M562 70 L572 70" class="ar"/>
<rect x="574" y="40" width="126" height="60" rx="7" class="k-green"/>
<text x="637.0" y="65.5" text-anchor="middle" class="kb-green">verify.yml</text>
<text x="637.0" y="83.5" text-anchor="middle" class="ks-green">must pass</text>
<rect x="120" y="140" width="480" height="50" rx="7" class="k-purple"/>
<text x="360.0" y="169.5" text-anchor="middle" class="ks-purple">every night: site.yml --check; any changed = drift alert</text>
<text x="360" y="214" text-anchor="middle" class="c">AWX, Ansible Automation Platform, Semaphore or any CI runner provide the schedule, credentials and logs</text>
</g>
</svg>
<figcaption>The same commands, run by a pipeline on every change.</figcaption>
</figure>

1. `ansible-lint` and `ansible-playbook --syntax-check`: style and syntax.
2. `ansible-playbook site.yml --check --diff`: what would change, attached to the merge request for review.
3. After approval and merge, `ansible-playbook site.yml`, against staging first, then production, with `serial` where it matters.
4. `ansible-playbook verify.yml`: the deployment isn't done until it passes.
5. Every night, `site.yml --check`: any `changed` is drift, and an alert.

Tools such as **AWX** (and its commercial version, Ansible Automation Platform), Semaphore, or a GitHub Actions or GitLab CI runner provide the scheduling, credentials, logs and access control around these runs. The playbooks don't change.

## What would make it production-ready?

| Gap | What to add |
|---|---|
| No encryption | TLS on the load balancer, certificates from Let's Encrypt (`community.crypto`) |
| Rules lost on restart | `iptables-persistent` or nftables configuration files |
| Secrets | admin password and API keys in a vault (Chapter 11) |
| One load balancer | a second one with a floating IP (keepalived) |
| No monitoring | a monitoring group checking `/health` and alerting (Prometheus, node exporter) |
| No central logs | ship logs to a log server (rsyslog, journald forwarding) |
| No backups | scheduled backups of the API's data, and a tested restore playbook |
| Manual runs | the pipeline above, with approvals |

## Hands on

1. Add web3 to the topology and the inventory. Which roles and firewall rules change when you run `site.yml`, and does `verify.yml` still pass without changes?
2. Make the firewall persistent: install `iptables-persistent` and save the rules with `iptables-save` in a handler.
3. Turn `verify.yml` into a role, and run both from one `ci.yml` with `import_playbook`.
4. Add a `monitoring` group with a server that checks every web server's `/health` every minute (`cron` and `curl`), with the firewall rules to match.
5. Put `site_title` and a password hash for opsadmin in a vault, and give the page a footer from it.

## Summary

- A production-style stack is **roles plus data**: a baseline on every server (common, hardening, firewall) and one role per job.
- **Least privilege** as data: each group's `firewall_allow` names hosts by group; the role turns them into rules with addresses from facts, `subelements` pairs ports with hosts, and `delegate_facts` makes it work with `--limit`.
- Order matters (allow before `DROP`); `validate` protects sshd and sudoers; handlers restart only what changed.
- A second run with `changed=0` proves idempotency; a **verification playbook** proves the result, including what must fail.
- Break, verify, repair, verify: the playbook is the truth. A pipeline runs it all on every change.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-14-final-project-a-secure-web-stack).

**1. Which hosts may connect to api1 on port 8080?**

- A. Anyone
- B. lb only
- C. control only
- D. web1 and web2 only

**2. Why does the firewall role set the DROP policy last?**

- A. iptables requires it
- B. Setting it first would cut Ansible's own SSH connection halfway through the run
- C. DROP must follow ACCEPT for speed
- D. It makes the role idempotent

**3. How does api1's firewall learn the web servers' addresses?**

- A. They are typed into the role
- B. From DNS
- C. From the load balancer
- D. From the web servers' facts, looked up for each host in firewall_allow

**4. Why does the firewall role gather some facts with delegate_to and delegate_facts: true?**

- A. With --limit, other hosts' facts would be missing; this gathers them and stores them as those hosts' facts
- B. To speed up the run
- C. To skip fact gathering on the current host
- D. To encrypt facts

**5. Why does the hardening role write its own file in sshd_config.d instead of editing sshd_config?**

- A. sshd_config is read-only
- B. To avoid validate
- C. It owns a whole file, leaving the package's file alone, and the first value set wins
- D. Drop-in files are faster

**6. What does the second run of site.yml reporting changed=0 show?**

- A. Nothing was configured
- B. The firewall is off
- C. Facts failed
- D. The whole stack is idempotent, so it can run every night to undo drift

**7. Why does verify.yml test that some connections fail?**

- A. To slow the run
- B. A firewall that allows everything would pass every positive test; blocked ports must be proved blocked
- C. Failures are easier to test
- D. To check DNS

**8. After someone tampers with web2, what restores it?**

- A. Rebuilding the server by hand
- B. Restarting the container
- C. site.yml --limit web2, then verify.yml to prove the repair
- D. Running verify.yml twice
