# Chapter 13: Network Automation with FRR

Northwind is growing beyond the shop's web servers. It has opened a roastery and a warehouse, and the three sites are linked by routers. Today the routers are configured by hand, one console at a time, with `configure terminal`. Every lesson of the previous chapters applies to networks as well: describe the desired state as **data**, generate configurations from **templates**, apply them **idempotently**, and **verify** the result. This chapter automates three routers running **FRRouting** (FRR), the open source routing suite, so that they learn their routes with **OSPF**.

> **Try it:** DockerLab, the routing lab in [`examples/ch13/ansible-13-frr-ospf.json`](https://github.com/khaledmahmudkm/ansiblehandson/tree/main/examples/ch13). Import it in DockerLab and Deploy. Topology: control `10.0.0.10`; routers r1, r2, r3 with management addresses `10.0.0.101` to `.103`. Start with `setup-keys r1 r2 r3`. The finished project is in `examples/ch13`.

## The network

<figure class="diagram">
<svg viewBox="0 0 720 410" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The routing lab. Control at 10.0.0.10 connects to a management switch, 10.0.0.0/24, which reaches each router&#x27;s eth0: r1 at 10.0.0.101, r2 at .102, r3 at .103. The routers form a ring: r1 eth1 to r2 eth1 on 10.0.12.0/24, r2 eth2 to r3 eth1 on 10.0.23.0/24, r3 eth2 to r1 eth2 on 10.0.13.0/24. Loopbacks are 1.1.1.1, 2.2.2.2 and 3.3.3.3." xmlns="http://www.w3.org/2000/svg">
<style>
.a13a{ font-family:var(--sans),sans-serif; }
.a13a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a13a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a13a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a13a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a13a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a13a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a13a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a13a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a13a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a13a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a13a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a13a .s{ fill:var(--text-dim); font-size:13px; }
.a13a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a13a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a13a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a13a .c{ fill:var(--text-faint); font-size:12.5px; }
.a13a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a13a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a13a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a13a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13a-a); }
.a13a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13a-a); marker-start:url(#a13a-s); }
.a13a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a13a-g); }
.a13a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a13a-a); }
.a13a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a13a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a13a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a13a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a13a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a13a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13a-tri); }
.a13a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13a-tri); }
.a13a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13a-open); }
.a13a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13a-open); }
.a13a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13a-dia); }
.a13a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13a-odia); }
.a13a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13a-dia); marker-end:url(#a13a-open); }
.a13a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13a-solid); }
.a13a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a13a-open); }
.a13a .fill{ fill:var(--text); stroke:none; }
.a13a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a13a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a13a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a13a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a13a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a13a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a13a .ks-blue{ fill:#185FA5; font-size:13px; }
.a13a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a13a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a13a .kb-blue, :root[data-theme="dark"] .a13a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a13a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a13a .kh-blue{ fill:#85B7EB; }
.a13a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a13a-blue); }
.a13a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13a-blue); }
.a13a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a13a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a13a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a13a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a13a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a13a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a13a .kb-teal, :root[data-theme="dark"] .a13a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a13a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a13a .kh-teal{ fill:#5DCAA5; }
.a13a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a13a-teal); }
.a13a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13a-teal); }
.a13a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a13a .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a13a .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a13a .ks-amber{ fill:#854F0B; font-size:13px; }
.a13a .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a13a .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13a .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a13a .kb-amber, :root[data-theme="dark"] .a13a .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a13a .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a13a .kh-amber{ fill:#EF9F27; }
.a13a .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a13a-amber); }
.a13a .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13a-amber); }
.a13a .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a13a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a13a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a13a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a13a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a13a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a13a .kb-gray, :root[data-theme="dark"] .a13a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a13a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a13a .kh-gray{ fill:#B4B2A9; }
.a13a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a13a-gray); }
.a13a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13a-gray); }
.a13a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a13a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a13a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a13a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a13a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a13a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a13a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a13a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a13a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a13a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a13a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a13a-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a13a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a13a">
<rect x="20" y="20" width="130" height="54" rx="7" class="dark"/>
<text x="85.0" y="43" text-anchor="middle" class="on">control</text>
<text x="85.0" y="61" text-anchor="middle" class="on" style="fill:var(--bg);font-weight:400">10.0.0.10</text>
<rect x="200" y="26" width="500" height="40" rx="20" class="k-gray"/>
<text x="450" y="51" text-anchor="middle" class="kb-gray">management switch  10.0.0.0/24  (eth0, no OSPF)</text>
<line x1="152" y1="46" x2="198" y2="46" class="lk"/>
<path d="M240 66 C 240 120, 150 120, 150 170" class="gd"/>
<path d="M570 66 L570 170" class="gd"/>
<path d="M680 66 C 690 200, 600 330, 392 340" class="gd"/>
<text x="158" y="120" text-anchor="start" class="c">eth0 .101</text>
<text x="562" y="120" text-anchor="end" class="c">eth0 .102</text>
<text x="660" y="250" text-anchor="start" class="c">eth0 .103</text>
<line x1="180" y1="200" x2="540" y2="200" class="ln-blue"/>
<line x1="545" y1="217" x2="385" y2="323" class="ln-blue"/>
<line x1="335" y1="323" x2="175" y2="217" class="ln-blue"/>
<text x="360" y="190" text-anchor="middle" class="km-blue">10.0.12.0/24</text>
<text x="200" y="190" text-anchor="middle" class="c">eth1 .1</text>
<text x="520" y="190" text-anchor="middle" class="c">eth1 .2</text>
<text x="498" y="290" text-anchor="start" class="km-blue">10.0.23.0/24</text>
<text x="222" y="290" text-anchor="end" class="km-blue">10.0.13.0/24</text>
<text x="586" y="246" text-anchor="start" class="c">eth2 .2</text>
<text x="410" y="360" text-anchor="start" class="c">eth1 .3</text>
<text x="132" y="246" text-anchor="end" class="c">eth2 .1</text>
<text x="310" y="360" text-anchor="end" class="c">eth2 .3</text>
<circle cx="150" cy="200" r="28" class="k-teal"/>
<text x="150" y="204.5" text-anchor="middle" class="kb-teal">r1</text>
<circle cx="570" cy="200" r="28" class="k-teal"/>
<text x="570" y="204.5" text-anchor="middle" class="kb-teal">r2</text>
<circle cx="360" cy="340" r="28" class="k-teal"/>
<text x="360" y="344.5" text-anchor="middle" class="kb-teal">r3</text>
<rect x="20" y="186" width="96" height="28" rx="14" class="k-amber"/>
<text x="68.0" y="204.5" text-anchor="middle" class="km-amber">lo 1.1.1.1</text>
<rect x="604" y="186" width="96" height="28" rx="14" class="k-amber"/>
<text x="652.0" y="204.5" text-anchor="middle" class="km-amber">lo 2.2.2.2</text>
<rect x="312" y="376" width="96" height="28" rx="14" class="k-amber"/>
<text x="360.0" y="394.5" text-anchor="middle" class="km-amber">lo 3.3.3.3</text>
</g>
</svg>
<figcaption>Three routers in a ring, managed out of band.</figcaption>
</figure>

| Router | Management (eth0) | Link | Link | Loopback, router ID |
|---|---|---|---|---|
| r1 | 10.0.0.101 | eth1 10.0.12.1/24 to r2 | eth2 10.0.13.1/24 to r3 | 1.1.1.1/32 |
| r2 | 10.0.0.102 | eth1 10.0.12.2/24 to r1 | eth2 10.0.23.2/24 to r3 | 2.2.2.2/32 |
| r3 | 10.0.0.103 | eth1 10.0.23.3/24 to r2 | eth2 10.0.13.3/24 to r1 | 3.3.3.3/32 |

Two design choices matter for automation:

- **Out-of-band management.** Each router's `eth0` connects to a separate management network, shared with control. Ansible reaches the routers there, so a mistake in the routing configuration cannot cut off Ansible's own access. OSPF must never run on `eth0`.
- **Loopbacks.** Each router has a `/32` address on its loopback interface, which stays up whatever happens to the links. It doubles as the OSPF **router ID** and stands for the site's network; the test at the end pings from loopback to loopback.

The routers are the same `ansible-node` image as the servers, with DockerLab's *router* option turning on IP forwarding. Ansible installs FRR itself, so the whole router configuration, software included, is code.

## Ways to automate a network

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Three ways Ansible automates network devices. Linux routers such as FRR: SSH and Python, templates and services. Vendor CLIs such as Cisco IOS, Arista EOS, Junos: network_cli connection, commands sent over SSH, resource modules such as ios_interfaces. APIs: NETCONF, RESTCONF, gNMI or a controller&#x27;s REST API, with structured data. All follow data, template or resource module, apply, verify." xmlns="http://www.w3.org/2000/svg">
<style>
.a13b{ font-family:var(--sans),sans-serif; }
.a13b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a13b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a13b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a13b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a13b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a13b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a13b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a13b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a13b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a13b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a13b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a13b .s{ fill:var(--text-dim); font-size:13px; }
.a13b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a13b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a13b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a13b .c{ fill:var(--text-faint); font-size:12.5px; }
.a13b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a13b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a13b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a13b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13b-a); }
.a13b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13b-a); marker-start:url(#a13b-s); }
.a13b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a13b-g); }
.a13b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a13b-a); }
.a13b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a13b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a13b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a13b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a13b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a13b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13b-tri); }
.a13b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13b-tri); }
.a13b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13b-open); }
.a13b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13b-open); }
.a13b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13b-dia); }
.a13b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13b-odia); }
.a13b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13b-dia); marker-end:url(#a13b-open); }
.a13b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13b-solid); }
.a13b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a13b-open); }
.a13b .fill{ fill:var(--text); stroke:none; }
.a13b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a13b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a13b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a13b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a13b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a13b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a13b .ks-blue{ fill:#185FA5; font-size:13px; }
.a13b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a13b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a13b .kb-blue, :root[data-theme="dark"] .a13b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a13b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a13b .kh-blue{ fill:#85B7EB; }
.a13b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a13b-blue); }
.a13b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13b-blue); }
.a13b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a13b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a13b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a13b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a13b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a13b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a13b .kb-teal, :root[data-theme="dark"] .a13b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a13b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a13b .kh-teal{ fill:#5DCAA5; }
.a13b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a13b-teal); }
.a13b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13b-teal); }
.a13b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a13b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a13b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a13b .ks-purple{ fill:#534AB7; font-size:13px; }
.a13b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a13b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a13b .kb-purple, :root[data-theme="dark"] .a13b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a13b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a13b .kh-purple{ fill:#AFA9EC; }
.a13b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a13b-purple); }
.a13b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13b-purple); }
.a13b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a13b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a13b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a13b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a13b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a13b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a13b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a13b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a13b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a13b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a13b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a13b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a13b">
<rect x="20" y="16" width="216" height="170" rx="10" class="k-teal"/>
<text x="128" y="40" text-anchor="middle" class="kb-teal">Linux routers</text>
<text x="128" y="58" text-anchor="middle" class="ks-teal">FRR, VyOS, SONiC</text>
<rect x="34" y="72" width="188" height="28" rx="4" class="pnl"/>
<text x="128.0" y="90.5" text-anchor="middle" class="m">SSH + Python</text>
<rect x="34" y="108" width="188" height="28" rx="4" class="pnl"/>
<text x="128.0" y="126.5" text-anchor="middle" class="m">template frr.conf</text>
<rect x="34" y="144" width="188" height="28" rx="4" class="pnl"/>
<text x="128.0" y="162.5" text-anchor="middle" class="m">service frr</text>
<rect x="252" y="16" width="216" height="170" rx="10" class="k-blue"/>
<text x="360" y="40" text-anchor="middle" class="kb-blue">vendor CLI</text>
<text x="360" y="58" text-anchor="middle" class="ks-blue">IOS, EOS, Junos</text>
<rect x="266" y="72" width="188" height="28" rx="4" class="pnl"/>
<text x="360.0" y="90.5" text-anchor="middle" class="m">network_cli</text>
<rect x="266" y="108" width="188" height="28" rx="4" class="pnl"/>
<text x="360.0" y="126.5" text-anchor="middle" class="m">send commands</text>
<rect x="266" y="144" width="188" height="28" rx="4" class="pnl"/>
<text x="360.0" y="162.5" text-anchor="middle" class="m">resource modules</text>
<rect x="484" y="16" width="216" height="170" rx="10" class="k-purple"/>
<text x="592" y="40" text-anchor="middle" class="kb-purple">APIs</text>
<text x="592" y="58" text-anchor="middle" class="ks-purple">NETCONF, RESTCONF, gNMI</text>
<rect x="498" y="72" width="188" height="28" rx="4" class="pnl"/>
<text x="592.0" y="90.5" text-anchor="middle" class="m">structured data</text>
<rect x="498" y="108" width="188" height="28" rx="4" class="pnl"/>
<text x="592.0" y="126.5" text-anchor="middle" class="m">netconf_config</text>
<rect x="498" y="144" width="188" height="28" rx="4" class="pnl"/>
<text x="592.0" y="162.5" text-anchor="middle" class="m">controller REST</text>
<text x="360" y="214" text-anchor="middle" class="kh-teal">the same method everywhere: data  -&gt;  template or resource module  -&gt;  apply  -&gt;  verify</text>
<text x="128" y="238" text-anchor="middle" class="c">this chapter</text>
</g>
</svg>
<figcaption>How Ansible reaches the device changes; the method doesn't.</figcaption>
</figure>

Network devices differ from servers in how Ansible talks to them:

| Approach | How | Typical devices |
|---|---|---|
| **Linux routers** | SSH and Python, like any server: install packages, template configuration files, start services | FRR, VyOS, Cumulus, SONiC, white-box switches |
| **CLI over SSH** | `ansible_connection: ansible.netcommon.network_cli`; modules send commands and parse the output; no Python on the device | Cisco IOS and NX-OS, Arista EOS, Juniper Junos |
| **APIs** | NETCONF, RESTCONF, gNMI, or a controller's REST API, with structured data | modern routers and switches, SD-WAN controllers |

Each vendor collection (`cisco.ios`, `arista.eos`, `junipernetworks.junos` ...) offers **resource modules**, such as `ios_interfaces` and `eos_ospfv2`, that take structured data and work out the commands, idempotently. The FRR routers here are Linux machines, so this chapter uses the first approach, with the tools you already know. The method is the same for all three: **data, template or resource module, apply, verify**.

## FRR on Linux

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="FRR&#x27;s architecture on a Linux router. vtysh talks to the daemons; ospfd and staticd compute routes and hand them to zebra, which installs them in the Linux kernel&#x27;s routing table and manages addresses. Ansible manages the files /etc/frr/daemons and /etc/frr/frr.conf and the frr service, and reads state with vtysh show commands in JSON." xmlns="http://www.w3.org/2000/svg">
<style>
.a13c{ font-family:var(--sans),sans-serif; }
.a13c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a13c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a13c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a13c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a13c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a13c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a13c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a13c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a13c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a13c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a13c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a13c .s{ fill:var(--text-dim); font-size:13px; }
.a13c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a13c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .c{ fill:var(--text-faint); font-size:12.5px; }
.a13c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a13c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a13c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a13c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13c-a); }
.a13c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13c-a); marker-start:url(#a13c-s); }
.a13c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a13c-g); }
.a13c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-a); }
.a13c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a13c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a13c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a13c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a13c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a13c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13c-tri); }
.a13c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-tri); }
.a13c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13c-open); }
.a13c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-open); }
.a13c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13c-dia); }
.a13c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13c-odia); }
.a13c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13c-dia); marker-end:url(#a13c-open); }
.a13c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13c-solid); }
.a13c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a13c-open); }
.a13c .fill{ fill:var(--text); stroke:none; }
.a13c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a13c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a13c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a13c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a13c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a13c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a13c .ks-blue{ fill:#185FA5; font-size:13px; }
.a13c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a13c .kb-blue, :root[data-theme="dark"] .a13c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a13c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a13c .kh-blue{ fill:#85B7EB; }
.a13c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a13c-blue); }
.a13c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-blue); }
.a13c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a13c .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a13c .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a13c .ks-teal{ fill:#0F6E56; font-size:13px; }
.a13c .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13c .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a13c .kb-teal, :root[data-theme="dark"] .a13c .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a13c .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a13c .kh-teal{ fill:#5DCAA5; }
.a13c .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a13c-teal); }
.a13c .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-teal); }
.a13c .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a13c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a13c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a13c .ks-amber{ fill:#854F0B; font-size:13px; }
.a13c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a13c .kb-amber, :root[data-theme="dark"] .a13c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a13c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a13c .kh-amber{ fill:#EF9F27; }
.a13c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a13c-amber); }
.a13c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-amber); }
.a13c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a13c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a13c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a13c .ks-purple{ fill:#534AB7; font-size:13px; }
.a13c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a13c .kb-purple, :root[data-theme="dark"] .a13c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a13c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a13c .kh-purple{ fill:#AFA9EC; }
.a13c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a13c-purple); }
.a13c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-purple); }
.a13c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a13c .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a13c .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a13c .ks-green{ fill:#3B6D11; font-size:13px; }
.a13c .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13c .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a13c .kb-green, :root[data-theme="dark"] .a13c .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a13c .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a13c .kh-green{ fill:#97C459; }
.a13c .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a13c-green); }
.a13c .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-green); }
.a13c .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a13c .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a13c .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a13c .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a13c .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a13c .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13c .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a13c .kb-gray, :root[data-theme="dark"] .a13c .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a13c .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a13c .kh-gray{ fill:#B4B2A9; }
.a13c .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a13c-gray); }
.a13c .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13c-gray); }
.a13c .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a13c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a13c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a13c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a13c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a13c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a13c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a13c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a13c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a13c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a13c-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a13c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a13c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a13c-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a13c-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a13c">
<rect x="20" y="16" width="470" height="238" rx="10" class="k-gray"/>
<text x="255" y="40" text-anchor="middle" class="kb-gray">router r1 (Linux)</text>
<rect x="40" y="56" width="130" height="44" rx="7" class="k-purple"/>
<text x="105.0" y="82.5" text-anchor="middle" class="km-purple">vtysh</text>
<rect x="200" y="56" width="120" height="44" rx="7" class="k-teal"/>
<text x="260.0" y="82.5" text-anchor="middle" class="km-teal">ospfd</text>
<rect x="340" y="56" width="130" height="44" rx="7" class="k-teal"/>
<text x="405.0" y="82.5" text-anchor="middle" class="km-teal">staticd, bgpd</text>
<rect x="200" y="130" width="270" height="44" rx="7" class="k-blue"/>
<text x="335.0" y="156.5" text-anchor="middle" class="km-blue">zebra</text>
<rect x="40" y="200" width="430" height="40" rx="7" class="pnl"/>
<text x="255.0" y="224.5" text-anchor="middle" class="s">Linux kernel: interfaces, routing table</text>
<path d="M172 78 L196 78" class="ar-purple"/>
<path d="M260 102 L300 126" class="ar-teal"/>
<path d="M405 102 L370 126" class="ar-teal"/>
<path d="M335 176 L335 196" class="ar-blue"/>
<rect x="520" y="30" width="180" height="56" rx="7" class="k-amber"/>
<text x="610.0" y="53.5" text-anchor="middle" class="km-amber">/etc/frr/daemons</text>
<text x="610.0" y="71.5" text-anchor="middle" class="ks-amber">lineinfile</text>
<rect x="520" y="100" width="180" height="56" rx="7" class="k-amber"/>
<text x="610.0" y="123.5" text-anchor="middle" class="km-amber">/etc/frr/frr.conf</text>
<text x="610.0" y="141.5" text-anchor="middle" class="ks-amber">template + validate</text>
<rect x="520" y="170" width="180" height="56" rx="7" class="k-green"/>
<text x="610.0" y="193.5" text-anchor="middle" class="km-green">vtysh show ... json</text>
<text x="610.0" y="211.5" text-anchor="middle" class="ks-green">verify reads state</text>
<path d="M516 58 L472 70" class="ar-amber"/>
<path d="M516 128 L472 150" class="ar-amber"/>
</g>
</svg>
<figcaption>Daemons per protocol, zebra to the kernel, two files for Ansible.</figcaption>
</figure>

FRR is a set of daemons. **zebra** talks to the Linux kernel, installing routes and addresses; each protocol has its own daemon (**ospfd**, **bgpd**, **staticd** ...); `vtysh` is the shell that talks to all of them, with a Cisco-like command language. Two files matter to Ansible:

- `/etc/frr/daemons`: which daemons to start (`ospfd=yes`).
- `/etc/frr/frr.conf`: the whole configuration, in the same language you would type in `vtysh`.

## The data

The project is a separate folder, `~/northwind-net`, with a YAML inventory, because routers carry structured data:

```yaml
# inventory.yml
all:
  children:
    routers:
      hosts:
        r1:
          ansible_host: 10.0.0.101
        r2:
          ansible_host: 10.0.0.102
        r3:
          ansible_host: 10.0.0.103
```

```yaml
# group_vars/routers.yml
---
ospf_area: 0
ospf_mgmt_interface: eth0      # never run OSPF on the management network
frr_daemons: [ospfd]
```

```yaml
# host_vars/r1.yml
---
router_id: 1.1.1.1
ospf_interfaces:
  - { name: eth1, address: 10.0.12.1/24, peer: r2 }
  - { name: eth2, address: 10.0.13.1/24, peer: r3 }
```

r2 and r3 have the same shape with their own values. Everything that makes r1 different from r2 is in these few lines. In larger networks this data comes from a **source of truth** such as NetBox, through a dynamic inventory (Chapter 4).

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="From data to configuration. group_vars/routers.yml holds ospf_area 0; host_vars/r1.yml holds router_id 1.1.1.1 and the list of OSPF interfaces eth1 and eth2 with their peers. The template frr.conf.j2 combines them into r1&#x27;s frr.conf with interface lo, interface eth1, interface eth2 and router ospf." xmlns="http://www.w3.org/2000/svg">
<style>
.a13d{ font-family:var(--sans),sans-serif; }
.a13d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a13d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a13d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a13d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a13d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a13d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a13d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a13d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a13d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a13d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a13d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a13d .s{ fill:var(--text-dim); font-size:13px; }
.a13d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a13d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a13d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a13d .c{ fill:var(--text-faint); font-size:12.5px; }
.a13d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a13d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a13d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a13d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13d-a); }
.a13d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13d-a); marker-start:url(#a13d-s); }
.a13d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a13d-g); }
.a13d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a13d-a); }
.a13d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a13d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a13d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a13d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a13d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a13d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13d-tri); }
.a13d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13d-tri); }
.a13d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13d-open); }
.a13d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13d-open); }
.a13d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13d-dia); }
.a13d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13d-odia); }
.a13d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13d-dia); marker-end:url(#a13d-open); }
.a13d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13d-solid); }
.a13d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a13d-open); }
.a13d .fill{ fill:var(--text); stroke:none; }
.a13d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a13d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a13d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a13d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a13d .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a13d .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a13d .ks-teal{ fill:#0F6E56; font-size:13px; }
.a13d .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a13d .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13d .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a13d .kb-teal, :root[data-theme="dark"] .a13d .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a13d .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a13d .kh-teal{ fill:#5DCAA5; }
.a13d .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a13d-teal); }
.a13d .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13d-teal); }
.a13d .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a13d .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a13d .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a13d .ks-amber{ fill:#854F0B; font-size:13px; }
.a13d .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a13d .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13d .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a13d .kb-amber, :root[data-theme="dark"] .a13d .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a13d .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a13d .kh-amber{ fill:#EF9F27; }
.a13d .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a13d-amber); }
.a13d .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13d-amber); }
.a13d .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a13d .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a13d .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a13d .ks-purple{ fill:#534AB7; font-size:13px; }
.a13d .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a13d .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13d .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a13d .kb-purple, :root[data-theme="dark"] .a13d .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a13d .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a13d .kh-purple{ fill:#AFA9EC; }
.a13d .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a13d-purple); }
.a13d .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13d-purple); }
.a13d .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a13d .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a13d .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a13d .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a13d .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a13d .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13d .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a13d .kb-gray, :root[data-theme="dark"] .a13d .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a13d .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a13d .kh-gray{ fill:#B4B2A9; }
.a13d .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a13d-gray); }
.a13d .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13d-gray); }
.a13d .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a13d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a13d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a13d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a13d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a13d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a13d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a13d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a13d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a13d-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a13d-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a13d-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a13d-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a13d">
<rect x="20" y="16" width="250" height="74" rx="6" class="k-gray"/>
<text x="30" y="36" text-anchor="start" class="kh-gray">group_vars/routers.yml</text>
<text x="32.0" y="58" text-anchor="start" class="m">ospf_area: 0</text>
<text x="32.0" y="76" text-anchor="start" class="m">frr_daemons: [ospfd]</text>
<rect x="20" y="110" width="250" height="110" rx="6" class="k-amber"/>
<text x="30" y="130" text-anchor="start" class="kh-amber">host_vars/r1.yml</text>
<text x="32.0" y="152" text-anchor="start" class="m">router_id: 1.1.1.1</text>
<text x="32.0" y="170" text-anchor="start" class="m">ospf_interfaces:</text>
<text x="47.2" y="188" text-anchor="start" class="m">- {name: eth1, peer: r2}</text>
<text x="47.2" y="206" text-anchor="start" class="m">- {name: eth2, peer: r3}</text>
<rect x="300" y="100" width="120" height="60" rx="7" class="k-purple"/>
<text x="360.0" y="125.5" text-anchor="middle" class="km-purple">frr.conf.j2</text>
<text x="360.0" y="143.5" text-anchor="middle" class="ks-purple">template</text>
<path d="M272 50 L316 98" class="ag"/>
<path d="M272 160 L296 140" class="ar-amber"/>
<rect x="450" y="16" width="260" height="190" rx="6" class="k-teal"/>
<text x="460" y="36" text-anchor="start" class="kh-teal">r1: /etc/frr/frr.conf</text>
<text x="462.0" y="58" text-anchor="start" class="m">interface lo</text>
<text x="469.6" y="77" text-anchor="start" class="m">ip address 1.1.1.1/32</text>
<text x="462.0" y="96" text-anchor="start" class="m">interface eth1</text>
<text x="469.6" y="115" text-anchor="start" class="m">description link to r2</text>
<text x="469.6" y="134" text-anchor="start" class="m">ip ospf area 0</text>
<text x="462.0" y="153" text-anchor="start" class="m">interface eth2  ...</text>
<text x="462.0" y="172" text-anchor="start" class="m">router ospf</text>
<text x="469.6" y="191" text-anchor="start" class="m">ospf router-id 1.1.1.1</text>
<path d="M422 130 L446 130" class="ar-teal"/>
</g>
</svg>
<figcaption>A few lines of data per router; the template does the rest.</figcaption>
</figure>

## The template

```
! {{ ansible_managed }}
frr defaults traditional
hostname {{ inventory_hostname }}
log syslog informational
service integrated-vtysh-config
!
interface lo
 ip address {{ router_id }}/32
 ip ospf area {{ ospf_area }}
exit
!
{% for i in ospf_interfaces %}
interface {{ i.name }}
 description link to {{ i.peer }}
 ip ospf area {{ ospf_area }}
 ip ospf network point-to-point
exit
!
{% endfor %}
router ospf
 ospf router-id {{ router_id }}
 passive-interface lo
exit
!
```

OSPF is enabled **per interface** (`ip ospf area 0`) on the loopback and the listed links only, so `eth0` is never touched. `ip ospf network point-to-point` suits a link with exactly two routers: no designated-router election, and faster adjacencies. The loopback is `passive`: its address is advertised, but no hellos are sent there.

## The playbook

```yaml
---
- name: FRR routers running OSPF
  hosts: routers
  become: true

  tasks:
    - name: Install FRR
      ansible.builtin.apt:
        name: frr
        update_cache: true
        cache_valid_time: 3600

    - name: Turn on the routing daemons this network uses
      ansible.builtin.lineinfile:
        path: /etc/frr/daemons
        regexp: "^{{ item }}="
        line: "{{ item }}=yes"
      loop: "{{ frr_daemons }}"
      notify: Restart FRR

    - name: Routing configuration, checked by FRR before use
      ansible.builtin.template:
        src: templates/frr.conf.j2
        dest: /etc/frr/frr.conf
        owner: frr
        group: frr
        mode: "0640"
        validate: /usr/bin/vtysh --dryrun --inputfile %s
      notify: Restart FRR

    - name: Start FRR
      ansible.builtin.service:
        name: frr
        state: started

  handlers:
    - name: Restart FRR
      ansible.builtin.service:
        name: frr
        state: restarted
```

```
cd ~/northwind-net
setup-keys r1 r2 r3
ansible-playbook routers.yml
ansible-playbook routers.yml
```

```
r1   : ok=6    changed=5    unreachable=0    failed=0 ...
...
r1   : ok=5    changed=0    unreachable=0    failed=0 ...
```

Everything from earlier chapters is here: `lineinfile` for one setting in a package's file, a template with `validate` (FRR parses the new configuration with `--dryrun` and rejects a typo before it replaces the running one), a handler so FRR restarts only when something changed, and a second run with `changed=0`. Restarting FRR briefly drops the router's adjacencies; on production routers, `frr-reload.py` applies only the differences, and a rolling `serial: 1` changes one router at a time, as in Chapter 12.

## Neighbours: how OSPF converges

After the restart, each router sends **hello** packets on its OSPF interfaces, finds its neighbours, exchanges its link-state database with them, and computes the shortest paths. A neighbour relationship climbs through several states to **Full**:

<figure class="diagram">
<svg viewBox="0 0 720 200" style="width:100%;height:auto;max-width:720px" role="img" aria-label="OSPF neighbour states from first contact to full adjacency: Down, Init after receiving a hello, 2-Way once each sees the other, ExStart and Exchange to swap database descriptions, Loading to request missing link-state advertisements, and Full when the databases match. Routes are computed once neighbours are Full." xmlns="http://www.w3.org/2000/svg">
<style>
.a13e{ font-family:var(--sans),sans-serif; }
.a13e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a13e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a13e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a13e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a13e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a13e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a13e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a13e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a13e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a13e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a13e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a13e .s{ fill:var(--text-dim); font-size:13px; }
.a13e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a13e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a13e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a13e .c{ fill:var(--text-faint); font-size:12.5px; }
.a13e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a13e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a13e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a13e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13e-a); }
.a13e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13e-a); marker-start:url(#a13e-s); }
.a13e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a13e-g); }
.a13e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a13e-a); }
.a13e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a13e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a13e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a13e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a13e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a13e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13e-tri); }
.a13e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13e-tri); }
.a13e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13e-open); }
.a13e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13e-open); }
.a13e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13e-dia); }
.a13e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13e-odia); }
.a13e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13e-dia); marker-end:url(#a13e-open); }
.a13e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13e-solid); }
.a13e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a13e-open); }
.a13e .fill{ fill:var(--text); stroke:none; }
.a13e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a13e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a13e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a13e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a13e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a13e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a13e .ks-blue{ fill:#185FA5; font-size:13px; }
.a13e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a13e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a13e .kb-blue, :root[data-theme="dark"] .a13e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a13e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a13e .kh-blue{ fill:#85B7EB; }
.a13e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a13e-blue); }
.a13e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13e-blue); }
.a13e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a13e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a13e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a13e .ks-purple{ fill:#534AB7; font-size:13px; }
.a13e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a13e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a13e .kb-purple, :root[data-theme="dark"] .a13e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a13e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a13e .kh-purple{ fill:#AFA9EC; }
.a13e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a13e-purple); }
.a13e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13e-purple); }
.a13e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a13e .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a13e .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a13e .ks-green{ fill:#3B6D11; font-size:13px; }
.a13e .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a13e .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13e .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a13e .kb-green, :root[data-theme="dark"] .a13e .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a13e .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a13e .kh-green{ fill:#97C459; }
.a13e .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a13e-green); }
.a13e .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13e-green); }
.a13e .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a13e .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a13e .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a13e .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a13e .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a13e .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13e .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a13e .kb-gray, :root[data-theme="dark"] .a13e .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a13e .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a13e .kh-gray{ fill:#B4B2A9; }
.a13e .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a13e-gray); }
.a13e .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13e-gray); }
.a13e .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a13e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a13e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a13e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a13e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a13e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a13e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a13e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a13e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a13e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a13e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a13e-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a13e-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a13e">
<rect x="14" y="50" width="90" height="44" rx="7" class="k-gray"/>
<text x="59.0" y="76.5" text-anchor="middle" class="kb-gray">Down</text>
<path d="M105 72 L113 72" class="ar"/>
<rect x="114" y="50" width="90" height="44" rx="7" class="k-gray"/>
<text x="159.0" y="76.5" text-anchor="middle" class="kb-gray">Init</text>
<path d="M205 72 L213 72" class="ar"/>
<rect x="214" y="50" width="90" height="44" rx="7" class="k-blue"/>
<text x="259.0" y="76.5" text-anchor="middle" class="kb-blue">2-Way</text>
<path d="M305 72 L313 72" class="ar"/>
<rect x="314" y="50" width="90" height="44" rx="7" class="k-blue"/>
<text x="359.0" y="76.5" text-anchor="middle" class="kb-blue">ExStart</text>
<path d="M405 72 L413 72" class="ar"/>
<rect x="414" y="50" width="90" height="44" rx="7" class="k-purple"/>
<text x="459.0" y="76.5" text-anchor="middle" class="kb-purple">Exchange</text>
<path d="M505 72 L513 72" class="ar"/>
<rect x="514" y="50" width="90" height="44" rx="7" class="k-purple"/>
<text x="559.0" y="76.5" text-anchor="middle" class="kb-purple">Loading</text>
<path d="M605 72 L613 72" class="ar"/>
<rect x="614" y="50" width="90" height="44" rx="7" class="k-green"/>
<text x="659.0" y="76.5" text-anchor="middle" class="kb-green">Full</text>
<text x="159" y="120" text-anchor="middle" class="c">hello received</text>
<text x="259" y="120" text-anchor="middle" class="c">sees itself</text>
<text x="459" y="120" text-anchor="middle" class="c">databases swapped</text>
<text x="659" y="120" text-anchor="middle" class="c">databases match</text>
<text x="360" y="166" text-anchor="middle" class="kh-green">verify.yml waits until every link's neighbour is Full</text>
</g>
</svg>
<figcaption>Convergence takes time: verification must wait for Full.</figcaption>
</figure>

Convergence takes a few seconds to tens of seconds. A verification playbook must therefore **wait** for it, not check once.

## Verify, from structured data

Most `vtysh` *show* commands can answer in **JSON**, which Ansible parses with `from_json`. The verification playbook waits until every router has a Full neighbour on each of its links, then pings every other router's loopback **from its own loopback**, which proves routes work in both directions:

```yaml
---
- name: Is the network converged?
  hosts: routers
  become: true
  gather_facts: false

  tasks:
    - name: Ask FRR for the OSPF neighbours, as JSON
      ansible.builtin.command: vtysh -c 'show ip ospf neighbor json'
      register: ospf
      changed_when: false
      until: >-
        (ospf.stdout | from_json).neighbors | dict2items
        | map(attribute='value') | flatten
        | selectattr('converged', 'defined') | selectattr('converged', 'eq', 'Full')
        | list | length == ospf_interfaces | length
      retries: 12
      delay: 5

    - name: Every router reaches every other router's loopback
      ansible.builtin.command: "ping -c 1 -W 2 -I {{ router_id }} {{ hostvars[item]['router_id'] }}"
      loop: "{{ groups['routers'] | difference([inventory_hostname]) }}"
      register: ping
      until: ping.rc == 0
      retries: 5
      delay: 2
      changed_when: false

    - name: Show the routes OSPF learned
      ansible.builtin.command: vtysh -c 'show ip route ospf'
      register: routes
      changed_when: false

    - name: Report
      ansible.builtin.debug:
        msg: "{{ routes.stdout_lines | select('search', '^O>') | list }}"
```

<figure class="diagram">
<svg viewBox="0 0 720 280" style="width:100%;height:auto;max-width:720px" role="img" aria-label="How verify.yml decides. vtysh show ip ospf neighbor json returns neighbors keyed by router ID, each a list of entries with a converged field. The filter chain from_json, dict2items, map value, flatten, selectattr converged eq Full, length gives the number of Full neighbours, compared with the number of ospf_interfaces in the router&#x27;s data: 2 equals 2. Then loopback pings to every other router." xmlns="http://www.w3.org/2000/svg">
<style>
.a13f{ font-family:var(--sans),sans-serif; }
.a13f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a13f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a13f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a13f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a13f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a13f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a13f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a13f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a13f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a13f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a13f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a13f .s{ fill:var(--text-dim); font-size:13px; }
.a13f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a13f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a13f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a13f .c{ fill:var(--text-faint); font-size:12.5px; }
.a13f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a13f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a13f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a13f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13f-a); }
.a13f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13f-a); marker-start:url(#a13f-s); }
.a13f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a13f-g); }
.a13f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-a); }
.a13f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a13f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a13f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a13f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a13f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a13f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13f-tri); }
.a13f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-tri); }
.a13f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13f-open); }
.a13f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-open); }
.a13f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13f-dia); }
.a13f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13f-odia); }
.a13f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13f-dia); marker-end:url(#a13f-open); }
.a13f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13f-solid); }
.a13f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a13f-open); }
.a13f .fill{ fill:var(--text); stroke:none; }
.a13f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a13f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a13f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a13f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a13f .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a13f .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a13f .ks-teal{ fill:#0F6E56; font-size:13px; }
.a13f .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a13f .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13f .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a13f .kb-teal, :root[data-theme="dark"] .a13f .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a13f .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a13f .kh-teal{ fill:#5DCAA5; }
.a13f .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a13f-teal); }
.a13f .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-teal); }
.a13f .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a13f .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a13f .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a13f .ks-amber{ fill:#854F0B; font-size:13px; }
.a13f .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a13f .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13f .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a13f .kb-amber, :root[data-theme="dark"] .a13f .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a13f .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a13f .kh-amber{ fill:#EF9F27; }
.a13f .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a13f-amber); }
.a13f .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-amber); }
.a13f .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a13f .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a13f .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a13f .ks-purple{ fill:#534AB7; font-size:13px; }
.a13f .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a13f .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13f .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a13f .kb-purple, :root[data-theme="dark"] .a13f .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a13f .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a13f .kh-purple{ fill:#AFA9EC; }
.a13f .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a13f-purple); }
.a13f .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-purple); }
.a13f .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a13f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a13f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a13f .ks-green{ fill:#3B6D11; font-size:13px; }
.a13f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a13f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a13f .kb-green, :root[data-theme="dark"] .a13f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a13f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a13f .kh-green{ fill:#97C459; }
.a13f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a13f-green); }
.a13f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-green); }
.a13f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a13f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a13f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a13f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a13f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a13f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a13f .kb-gray, :root[data-theme="dark"] .a13f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a13f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a13f .kh-gray{ fill:#B4B2A9; }
.a13f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a13f-gray); }
.a13f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13f-gray); }
.a13f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a13f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a13f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a13f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a13f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a13f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a13f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a13f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a13f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a13f-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a13f-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a13f-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a13f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a13f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a13f">
<rect x="20" y="16" width="300" height="152" rx="6" class="k-gray"/>
<text x="30" y="36" text-anchor="start" class="kh-gray">show ip ospf neighbor json</text>
<text x="32.0" y="58" text-anchor="start" class="m">{"neighbors": {</text>
<text x="47.2" y="77" text-anchor="start" class="m">"2.2.2.2": [</text>
<text x="62.4" y="96" text-anchor="start" class="m">{"converged": "Full", ...}],</text>
<text x="47.2" y="115" text-anchor="start" class="m">"3.3.3.3": [</text>
<text x="62.4" y="134" text-anchor="start" class="m">{"converged": "Full", ...}]</text>
<text x="32.0" y="153" text-anchor="start" class="m">}}</text>
<rect x="360" y="16" width="200" height="32" rx="5" class="k-purple"/>
<text x="460.0" y="36.5" text-anchor="middle" class="km-purple">from_json</text>
<path d="M460 49 L460 55" class="ar-purple"/>
<rect x="360" y="56" width="200" height="32" rx="5" class="k-purple"/>
<text x="460.0" y="76.5" text-anchor="middle" class="km-purple">dict2items | map(value)</text>
<path d="M460 89 L460 95" class="ar-purple"/>
<rect x="360" y="96" width="200" height="32" rx="5" class="k-purple"/>
<text x="460.0" y="116.5" text-anchor="middle" class="km-purple">flatten</text>
<path d="M460 129 L460 135" class="ar-purple"/>
<rect x="360" y="136" width="200" height="32" rx="5" class="k-purple"/>
<text x="460.0" y="156.5" text-anchor="middle" class="km-purple">selectattr Full</text>
<path d="M460 169 L460 175" class="ar-purple"/>
<rect x="360" y="176" width="200" height="32" rx="5" class="k-green"/>
<text x="460.0" y="196.5" text-anchor="middle" class="km-green">length = 2</text>
<rect x="590" y="160" width="120" height="44" rx="7" class="k-amber"/>
<text x="650.0" y="178.0" text-anchor="middle" class="km-amber">ospf_interfaces</text>
<text x="650.0" y="195.0" text-anchor="middle" class="km-amber">| length = 2</text>
<text x="575" y="188" text-anchor="middle" class="h">=</text>
<rect x="20" y="224" width="300" height="44" rx="7" class="k-teal"/>
<text x="170.0" y="250.5" text-anchor="middle" class="km-teal">then: ping -I 1.1.1.1 2.2.2.2, 3.3.3.3</text>
<path d="M322 66 L356 32" class="ag"/>
</g>
</svg>
<figcaption>The expected state comes from the same data that built the configuration.</figcaption>
</figure>

```
ansible-playbook verify.yml
```

```
ok: [r1] => {
    "msg": [
        "O>* 2.2.2.2/32 [110/10] via 10.0.12.2, eth1, weight 1, 00:00:06",
        "O>* 3.3.3.3/32 [110/10] via 10.0.13.3, eth2, weight 1, 00:00:11",
        "O>* 10.0.23.0/24 [110/20] via 10.0.12.2, eth1, weight 1, 00:00:06"
    ]
}
...
r1   : ok=4    changed=0    unreachable=0    failed=0 ...
r2   : ok=4    changed=0    unreachable=0    failed=0 ...
r3   : ok=4    changed=0    unreachable=0    failed=0 ...
```

The `until` expression reads like a pipeline: parse the JSON; FRR lists neighbours under `neighbors`, keyed by router ID, each with a list of entries; take the values, flatten them, keep entries whose `converged` state is `Full`, count them, and compare with the number of links in this router's own data. The **expected** state comes from the same data that built the configuration, so the test grows with the network. `hostvars[item]['router_id']` reads the other routers' loopbacks from the inventory, without facts.

## Break a link, watch it heal

```
ansible r1 -b -a 'ip link set eth1 down'
ansible r1 -b -a "vtysh -c 'show ip route 2.2.2.2'"
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Failover. With the r1 to r2 link up, r1 reaches 2.2.2.2 directly via eth1 at cost 10. With eth1 down, OSPF recomputes and r1 reaches 2.2.2.2 via r3, through eth2, at cost 20." xmlns="http://www.w3.org/2000/svg">
<style>
.a13g{ font-family:var(--sans),sans-serif; }
.a13g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a13g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a13g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a13g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a13g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a13g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a13g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a13g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a13g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a13g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a13g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a13g .s{ fill:var(--text-dim); font-size:13px; }
.a13g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a13g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a13g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a13g .c{ fill:var(--text-faint); font-size:12.5px; }
.a13g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a13g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a13g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a13g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13g-a); }
.a13g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a13g-a); marker-start:url(#a13g-s); }
.a13g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a13g-g); }
.a13g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-a); }
.a13g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a13g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a13g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a13g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a13g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a13g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13g-tri); }
.a13g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-tri); }
.a13g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13g-open); }
.a13g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-open); }
.a13g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13g-dia); }
.a13g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13g-odia); }
.a13g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a13g-dia); marker-end:url(#a13g-open); }
.a13g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a13g-solid); }
.a13g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a13g-open); }
.a13g .fill{ fill:var(--text); stroke:none; }
.a13g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a13g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a13g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a13g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a13g .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a13g .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a13g .ks-blue{ fill:#185FA5; font-size:13px; }
.a13g .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a13g .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13g .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a13g .kb-blue, :root[data-theme="dark"] .a13g .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a13g .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a13g .kh-blue{ fill:#85B7EB; }
.a13g .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a13g-blue); }
.a13g .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-blue); }
.a13g .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a13g .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a13g .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a13g .ks-teal{ fill:#0F6E56; font-size:13px; }
.a13g .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a13g .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13g .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a13g .kb-teal, :root[data-theme="dark"] .a13g .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a13g .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a13g .kh-teal{ fill:#5DCAA5; }
.a13g .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a13g-teal); }
.a13g .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-teal); }
.a13g .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a13g .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a13g .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a13g .ks-coral{ fill:#993C1D; font-size:13px; }
.a13g .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a13g .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13g .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a13g .kb-coral, :root[data-theme="dark"] .a13g .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a13g .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a13g .kh-coral{ fill:#F0997B; }
.a13g .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a13g-coral); }
.a13g .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-coral); }
.a13g .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a13g .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a13g .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a13g .ks-green{ fill:#3B6D11; font-size:13px; }
.a13g .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a13g .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13g .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a13g .kb-green, :root[data-theme="dark"] .a13g .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a13g .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a13g .kh-green{ fill:#97C459; }
.a13g .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a13g-green); }
.a13g .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-green); }
.a13g .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a13g .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a13g .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a13g .ks-red{ fill:#A32D2D; font-size:13px; }
.a13g .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a13g .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a13g .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a13g .kb-red, :root[data-theme="dark"] .a13g .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a13g .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a13g .kh-red{ fill:#F09595; }
.a13g .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a13g-red); }
.a13g .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a13g-red); }
.a13g .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a13g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a13g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a13g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a13g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a13g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a13g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a13g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a13g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a13g-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a13g-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a13g-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a13g-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a13g-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a13g">
<text x="180" y="24" text-anchor="middle" class="b">link up</text>
<line x1="96" y1="90" x2="264" y2="90" class="ln-blue"/>
<line x1="271" y1="107" x2="199" y2="173" class="ln-blue"/>
<line x1="161" y1="173" x2="89" y2="107" class="ln-blue"/>
<path d="M100 76 L260 76" class="ar-green"/>
<text x="180" y="66" text-anchor="middle" class="kh-green">O&gt;* 2.2.2.2 via eth1, cost 10</text>
<circle cx="70" cy="90" r="24" class="k-teal"/>
<text x="70" y="94.5" text-anchor="middle" class="kb-teal">r1</text>
<circle cx="290" cy="90" r="24" class="k-teal"/>
<text x="290" y="94.5" text-anchor="middle" class="kb-teal">r2</text>
<circle cx="180" cy="190" r="24" class="k-teal"/>
<text x="180" y="194.5" text-anchor="middle" class="kb-teal">r3</text>
<text x="540" y="24" text-anchor="middle" class="b">eth1 down</text>
<line x1="456" y1="90" x2="624" y2="90" class="x"/>
<line x1="631" y1="107" x2="559" y2="173" class="ln-blue"/>
<line x1="521" y1="173" x2="449" y2="107" class="ln-blue"/>
<text x="540" y="84" text-anchor="middle" class="kh-red">X</text>
<path d="M446 116 L520 176" class="ar-coral"/>
<path d="M564 176 L632 116" class="ar-coral"/>
<text x="540" y="240" text-anchor="middle" class="kh-coral">O&gt;* 2.2.2.2 via r3 (eth2), cost 20</text>
<circle cx="430" cy="90" r="24" class="k-teal"/>
<text x="430" y="94.5" text-anchor="middle" class="kb-teal">r1</text>
<circle cx="650" cy="90" r="24" class="k-teal"/>
<text x="650" y="94.5" text-anchor="middle" class="kb-teal">r2</text>
<circle cx="540" cy="190" r="24" class="k-teal"/>
<text x="540" y="194.5" text-anchor="middle" class="kb-teal">r3</text>
</g>
</svg>
<figcaption>OSPF routes around a failed link; verify.yml reports the lost neighbour.</figcaption>
</figure>

Within OSPF's dead interval (40 seconds by default; less with point-to-point tuning), r1 loses its neighbour on eth1 and recomputes: the route to 2.2.2.2 now goes via r3, at a higher cost. `verify.yml` now fails r1's neighbour count, which is exactly what monitoring should report. Bring the link back with `ip link set eth1 up` and run `verify.yml` again.

## Network automation, done safely

- **Never lose the management path.** Manage devices out of band, and keep management interfaces out of routing protocols and firewall changes.
- **Validate before applying**: `vtysh --dryrun` here; `check_mode` and `--diff` with resource modules.
- **Change one device at a time** (`serial: 1`) and verify between devices, as in Chapter 12.
- **Back up before changing**: `vtysh -c 'show running-config'` registered and saved on control, or the vendor's `*_config` module with `backup: true`.
- **Keep the data in a source of truth**, and generate everything else from it.

## Hands on

1. Add a site LAN to each router: a `dummy` interface `lan0` with `192.168.N.1/24`, advertised by OSPF as passive. Which parts of the data, template and verification change?
2. Make the r1 to r2 link expensive (`ip ospf cost 100` from a per-interface `cost` value). Where does traffic from r1 to r2 go now?
3. Add a fourth router: what do you add to the inventory and `host_vars`, and does `verify.yml` need any change?
4. Save each router's running configuration to `backups/ROUTER.conf` on control, with a timestamp.
5. Replace OSPF with BGP: one AS per router, eBGP sessions on the links. Which data would you add?

## Summary

- Network automation applies the same method as server automation: **data, templates, idempotent apply, verify**.
- Linux routers such as FRR are managed like servers; vendor devices use `network_cli` or APIs with **resource modules**.
- FRR's `daemons` file selects daemons; `frr.conf` holds the configuration; `vtysh --dryrun` validates it.
- Keep management **out of band** and out of the routing protocol.
- Verify from **structured output** (`show ... json` with `from_json`), wait for convergence with `until`, and test end to end with loopback pings.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-13-network-automation-with-frr).

**1. Why do the routers' eth0 interfaces sit on a separate management network?**

- A. So a routing mistake cannot cut off Ansible's own access: management is out of band
- B. For faster OSPF
- C. eth0 can't carry routes
- D. To save addresses

**2. Why does each router have a /32 address on its loopback?**

- A. Loopbacks are required by SSH
- B. It replaces eth0
- C. It stays up whatever happens to the links, and doubles as the OSPF router ID
- D. It is the management address

**3. How does Ansible manage the FRR routers in this chapter?**

- A. Through network_cli and vendor modules
- B. Like servers: SSH and Python, apt, lineinfile, a template and the service module
- C. Through NETCONF
- D. Through a controller's REST API

**4. Which two files does Ansible manage for FRR?**

- A. /etc/hosts and /etc/resolv.conf
- B. sshd_config and sudoers
- C. /etc/network/interfaces only
- D. /etc/frr/daemons and /etc/frr/frr.conf

**5. What does validate: /usr/bin/vtysh --dryrun --inputfile %s protect against?**

- A. Slow convergence
- B. A typo in frr.conf replacing the running configuration
- C. Lost SSH keys
- D. Duplicate router IDs

**6. Why does verify.yml use until with retries for the OSPF neighbour check?**

- A. vtysh is slow to start
- B. JSON parsing can fail
- C. To retry SSH
- D. OSPF needs time to converge; neighbours must reach Full

**7. Where does verify.yml get the number of neighbours each router should have?**

- A. From the router's own data: the length of ospf_interfaces
- B. From a hard-coded number
- C. From the spine
- D. From DNS

**8. r1's eth1 link to r2 goes down. What does OSPF do?**

- A. Nothing until it comes back
- B. Drops all routes
- C. Recomputes routes: r1 reaches 2.2.2.2 via r3 at a higher cost
- D. Restarts FRR
