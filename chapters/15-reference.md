# Chapter 15: Quick Reference

A compact summary of the commands, keywords, modules and filters used in this book, with the chapter that explains each.

## The project layout

<figure class="diagram">
<svg viewBox="0 0 720 400" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A typical Ansible project layout: ansible.cfg, inventory, requirements.yml, site.yml and other playbooks at the top; group_vars and host_vars folders with per-group and per-host YAML, a group folder holding vars.yml and an encrypted vault.yml; and a roles folder with one folder per role, each with tasks, handlers, templates, files, defaults, vars and meta." xmlns="http://www.w3.org/2000/svg">
<style>
.a15a{ font-family:var(--sans),sans-serif; }
.a15a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a15a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a15a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a15a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a15a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a15a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a15a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a15a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a15a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a15a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a15a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a15a .s{ fill:var(--text-dim); font-size:13px; }
.a15a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a15a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .c{ fill:var(--text-faint); font-size:12.5px; }
.a15a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a15a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a15a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a15a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a15a-a); }
.a15a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a15a-a); marker-start:url(#a15a-s); }
.a15a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a15a-g); }
.a15a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-a); }
.a15a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a15a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a15a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a15a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a15a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a15a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a15a-tri); }
.a15a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-tri); }
.a15a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a15a-open); }
.a15a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-open); }
.a15a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a15a-dia); }
.a15a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a15a-odia); }
.a15a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a15a-dia); marker-end:url(#a15a-open); }
.a15a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a15a-solid); }
.a15a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a15a-open); }
.a15a .fill{ fill:var(--text); stroke:none; }
.a15a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a15a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a15a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a15a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a15a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a15a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a15a .ks-blue{ fill:#185FA5; font-size:13px; }
.a15a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a15a .kb-blue, :root[data-theme="dark"] .a15a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a15a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a15a .kh-blue{ fill:#85B7EB; }
.a15a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a15a-blue); }
.a15a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-blue); }
.a15a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a15a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a15a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a15a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a15a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a15a .kb-teal, :root[data-theme="dark"] .a15a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a15a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a15a .kh-teal{ fill:#5DCAA5; }
.a15a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a15a-teal); }
.a15a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-teal); }
.a15a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a15a .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a15a .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a15a .ks-amber{ fill:#854F0B; font-size:13px; }
.a15a .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15a .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a15a .kb-amber, :root[data-theme="dark"] .a15a .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a15a .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a15a .kh-amber{ fill:#EF9F27; }
.a15a .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a15a-amber); }
.a15a .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-amber); }
.a15a .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a15a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a15a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a15a .ks-purple{ fill:#534AB7; font-size:13px; }
.a15a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a15a .kb-purple, :root[data-theme="dark"] .a15a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a15a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a15a .kh-purple{ fill:#AFA9EC; }
.a15a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a15a-purple); }
.a15a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-purple); }
.a15a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a15a .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a15a .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a15a .ks-green{ fill:#3B6D11; font-size:13px; }
.a15a .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15a .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a15a .kb-green, :root[data-theme="dark"] .a15a .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a15a .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a15a .kh-green{ fill:#97C459; }
.a15a .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a15a-green); }
.a15a .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-green); }
.a15a .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a15a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a15a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a15a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a15a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a15a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a15a .kb-gray, :root[data-theme="dark"] .a15a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a15a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a15a .kh-gray{ fill:#B4B2A9; }
.a15a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a15a-gray); }
.a15a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15a-gray); }
.a15a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a15a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a15a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a15a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a15a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a15a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a15a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a15a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a15a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a15a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a15a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a15a-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a15a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a15a-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a15a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a15a">
<rect x="20" y="16" width="330" height="361" rx="6" class="pnl"/>
<text x="32.0" y="36" text-anchor="start" class="m">project/</text>
<text x="32.0" y="59" text-anchor="start" class="m">├── ansible.cfg</text>
<text x="32.0" y="82" text-anchor="start" class="m">├── inventory</text>
<text x="32.0" y="105" text-anchor="start" class="m">├── requirements.yml</text>
<text x="32.0" y="128" text-anchor="start" class="m">├── site.yml</text>
<text x="32.0" y="151" text-anchor="start" class="m">├── verify.yml</text>
<text x="32.0" y="174" text-anchor="start" class="m">├── group_vars/</text>
<text x="32.0" y="197" text-anchor="start" class="m">│   ├── all.yml</text>
<text x="32.0" y="220" text-anchor="start" class="m">│   └── web/</text>
<text x="32.0" y="243" text-anchor="start" class="m">│       ├── vars.yml</text>
<text x="32.0" y="266" text-anchor="start" class="m">│       └── vault.yml</text>
<text x="32.0" y="289" text-anchor="start" class="m">├── host_vars/</text>
<text x="32.0" y="312" text-anchor="start" class="m">│   └── web2.yml</text>
<text x="32.0" y="335" text-anchor="start" class="m">└── roles/</text>
<text x="62.4" y="358" text-anchor="start" class="m">└── webserver/  tasks/ handlers/ ...</text>
<line x1="352" y1="56" x2="384" y2="56" class="ln-gray"/>
<text x="392" y="60" text-anchor="start" class="kh-gray">settings: inventory, vault password file</text>
<line x1="352" y1="79" x2="384" y2="79" class="ln-teal"/>
<text x="392" y="83" text-anchor="start" class="kh-teal">hosts and groups</text>
<line x1="352" y1="102" x2="384" y2="102" class="ln-green"/>
<text x="392" y="106" text-anchor="start" class="kh-green">collections the project needs</text>
<line x1="352" y1="125" x2="384" y2="125" class="ln-purple"/>
<text x="392" y="129" text-anchor="start" class="kh-purple">the whole site: roles per group</text>
<line x1="352" y1="148" x2="384" y2="148" class="ln-green"/>
<text x="392" y="152" text-anchor="start" class="kh-green">checks the result</text>
<line x1="352" y1="240" x2="384" y2="240" class="ln-blue"/>
<text x="392" y="244" text-anchor="start" class="kh-blue">data per group: plain ...</text>
<line x1="352" y1="263" x2="384" y2="263" class="ln-purple"/>
<text x="392" y="267" text-anchor="start" class="kh-purple">... and encrypted (Chapter 11)</text>
<line x1="352" y1="286" x2="384" y2="286" class="ln-blue"/>
<text x="392" y="290" text-anchor="start" class="kh-blue">data for one host</text>
<line x1="352" y1="332" x2="384" y2="332" class="ln-amber"/>
<text x="392" y="336" text-anchor="start" class="kh-amber">one folder per job (Chapter 10)</text>
</g>
</svg>
<figcaption>Where everything goes in an Ansible project.</figcaption>
</figure>

## Commands

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The Ansible commands around a project: ansible for ad-hoc commands, ansible-playbook to run playbooks, ansible-inventory to inspect the inventory, ansible-config for settings, ansible-doc for documentation, ansible-galaxy for roles and collections, ansible-vault for secrets, and ansible-lint to check the project." xmlns="http://www.w3.org/2000/svg">
<style>
.a15b{ font-family:var(--sans),sans-serif; }
.a15b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a15b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a15b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a15b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a15b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a15b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a15b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a15b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a15b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a15b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a15b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a15b .s{ fill:var(--text-dim); font-size:13px; }
.a15b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a15b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a15b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a15b .c{ fill:var(--text-faint); font-size:12.5px; }
.a15b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a15b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a15b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a15b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a15b-a); }
.a15b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a15b-a); marker-start:url(#a15b-s); }
.a15b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a15b-g); }
.a15b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a15b-a); }
.a15b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a15b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a15b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a15b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a15b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a15b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a15b-tri); }
.a15b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a15b-tri); }
.a15b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a15b-open); }
.a15b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a15b-open); }
.a15b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a15b-dia); }
.a15b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a15b-odia); }
.a15b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a15b-dia); marker-end:url(#a15b-open); }
.a15b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a15b-solid); }
.a15b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a15b-open); }
.a15b .fill{ fill:var(--text); stroke:none; }
.a15b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a15b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a15b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a15b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a15b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a15b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a15b .ks-purple{ fill:#534AB7; font-size:13px; }
.a15b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a15b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a15b .kb-purple, :root[data-theme="dark"] .a15b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a15b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a15b .kh-purple{ fill:#AFA9EC; }
.a15b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a15b-purple); }
.a15b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15b-purple); }
.a15b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a15b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a15b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a15b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a15b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a15b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a15b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a15b .kb-gray, :root[data-theme="dark"] .a15b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a15b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a15b .kh-gray{ fill:#B4B2A9; }
.a15b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a15b-gray); }
.a15b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a15b-gray); }
.a15b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a15b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a15b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a15b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a15b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a15b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a15b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a15b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a15b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a15b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a15b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a15b">
<rect x="270" y="80" width="180" height="70" rx="7" class="k-gray"/>
<text x="360.0" y="110.5" text-anchor="middle" class="kb-gray">the project</text>
<text x="360.0" y="128.5" text-anchor="middle" class="km-gray">~/northwind</text>
<rect x="20" y="20" width="180" height="50" rx="7" class="k-purple"/>
<text x="110.0" y="41.0" text-anchor="middle" class="km-purple">ansible</text>
<text x="110.0" y="58.0" text-anchor="middle" class="ks-purple">ad-hoc</text>
<rect x="270" y="16" width="180" height="50" rx="7" class="k-purple"/>
<text x="360.0" y="37.0" text-anchor="middle" class="km-purple">ansible-playbook</text>
<text x="360.0" y="54.0" text-anchor="middle" class="ks-purple">run plays</text>
<rect x="520" y="20" width="180" height="50" rx="7" class="k-purple"/>
<text x="610.0" y="41.0" text-anchor="middle" class="km-purple">ansible-inventory</text>
<text x="610.0" y="58.0" text-anchor="middle" class="ks-purple">inspect hosts</text>
<rect x="20" y="90" width="180" height="50" rx="7" class="k-purple"/>
<text x="110.0" y="111.0" text-anchor="middle" class="km-purple">ansible-config</text>
<text x="110.0" y="128.0" text-anchor="middle" class="ks-purple">settings</text>
<rect x="520" y="90" width="180" height="50" rx="7" class="k-purple"/>
<text x="610.0" y="111.0" text-anchor="middle" class="km-purple">ansible-doc</text>
<text x="610.0" y="128.0" text-anchor="middle" class="ks-purple">module docs</text>
<rect x="20" y="160" width="180" height="50" rx="7" class="k-purple"/>
<text x="110.0" y="181.0" text-anchor="middle" class="km-purple">ansible-galaxy</text>
<text x="110.0" y="198.0" text-anchor="middle" class="ks-purple">roles, collections</text>
<rect x="270" y="166" width="180" height="50" rx="7" class="k-purple"/>
<text x="360.0" y="187.0" text-anchor="middle" class="km-purple">ansible-vault</text>
<text x="360.0" y="204.0" text-anchor="middle" class="ks-purple">secrets</text>
<rect x="520" y="160" width="180" height="50" rx="7" class="k-purple"/>
<text x="610.0" y="181.0" text-anchor="middle" class="km-purple">ansible-lint</text>
<text x="610.0" y="198.0" text-anchor="middle" class="ks-purple">good practice</text>
<line x1="202" y1="45" x2="266" y2="95" class="ln"/>
<line x1="360" y1="66" x2="360" y2="78" class="ln"/>
<line x1="518" y1="45" x2="454" y2="95" class="ln"/>
<line x1="202" y1="115" x2="266" y2="115" class="ln"/>
<line x1="518" y1="115" x2="454" y2="115" class="ln"/>
<line x1="202" y1="185" x2="266" y2="140" class="ln"/>
<line x1="360" y1="164" x2="360" y2="152" class="ln"/>
<line x1="518" y1="185" x2="454" y2="140" class="ln"/>
</g>
</svg>
<figcaption>Eight commands, one project.</figcaption>
</figure>

| Command | Purpose |
|---|---|
| `ansible PATTERN -m MODULE -a ARGS` | ad-hoc command (2, 3) |
| `ansible-playbook site.yml` | run a playbook (5) |
| `ansible-playbook site.yml -C -D` | dry run with diffs (3, 5) |
| `ansible-playbook site.yml -l web2` | limit to some hosts (5) |
| `ansible-playbook site.yml -t config` / `--skip-tags packages` | run part of a playbook (9) |
| `ansible-playbook site.yml -e var=value` / `-e @file.yml` | extra variables (6) |
| `ansible-playbook site.yml --syntax-check` / `--list-tasks` / `--list-hosts` | look before running (5) |
| `ansible-playbook site.yml --start-at-task NAME` / `--step` | start in the middle, ask per task (5) |
| `ansible-inventory --graph` / `--host H` / `--list` | inspect the inventory (4) |
| `ansible-config dump --only-changed` | settings that differ, and their source (4) |
| `ansible-doc MODULE` / `-s MODULE` / `-l` | module documentation (3) |
| `ansible-galaxy role init roles/NAME` | role skeleton (10) |
| `ansible-galaxy collection install -r requirements.yml` | install collections (10) |
| `ansible-vault create/encrypt/view/edit/rekey FILE` | encrypted files (11) |
| `ansible-vault encrypt_string --stdin-name VAR` | one encrypted value (11) |
| `ansible-lint` | check good practice (10) |

Common options: `-i INVENTORY`, `-u USER`, `-b` (become), `-K` (ask become password), `-f N` (forks), `-v` to `-vvvv`, `--ask-vault-pass`, `--vault-password-file F`, `--force-handlers`.

## Host patterns

| Pattern | Hosts |
|---|---|
| `all`, `*` | every host |
| `web` | group web |
| `web:db` | in web or db |
| `web:&production` | in web and production |
| `all:!db` | all except db |
| `web*`, `~web[0-9]+` | wildcard, regular expression |
| `--limit web1` | narrows any pattern |

## Play keywords

| Keyword | Meaning | Chapter |
|---|---|---|
| `name`, `hosts` | the play's name; a host pattern | 5 |
| `become`, `become_user` | run as root (or another user) | 2, 5 |
| `gather_facts` | collect facts first (default true) | 6 |
| `vars`, `vars_files`, `vars_prompt` | play variables | 6 |
| `pre_tasks`, `roles`, `tasks`, `post_tasks`, `handlers` | the play's parts, in running order | 9, 10, 12 |
| `serial`, `max_fail_percentage`, `any_errors_fatal` | batches and failure limits | 12 |
| `tags` | tags for every task in the play | 9 |
| `order`, `strategy`, `throttle` | host order, execution strategy, parallelism | 12 |

## Task keywords

| Keyword | Meaning | Chapter |
|---|---|---|
| `name` | what the task achieves | 5 |
| `when` | condition, without braces | 8 |
| `loop`, `loop_control` (`label`, `index_var`) | repeat over a list | 8 |
| `register` | keep the result | 6 |
| `until`, `retries`, `delay` | retry until a condition holds | 8 |
| `changed_when`, `failed_when` | define changed and failed | 6, 8 |
| `ignore_errors` | carry on after a failure | 8 |
| `notify` | trigger a handler when changed | 9 |
| `listen` (handlers) | answer a notification topic | 9 |
| `block`, `rescue`, `always` | group tasks; handle failures | 8 |
| `delegate_to`, `delegate_facts`, `run_once` | run elsewhere, once | 7, 12, 14 |
| `become`, `no_log`, `diff`, `check_mode`, `tags` | per-task controls | 9, 11 |

## Modules used in this book

| Module | Does | Chapter |
|---|---|---|
| `ping`, `setup`, `debug`, `assert`, `fail` | test, facts, print, check, stop | 2, 6, 8 |
| `command`, `shell` | run programs | 3 |
| `apt`, `package` | software | 3 |
| `copy`, `template`, `file`, `lineinfile`, `stat`, `fetch` | files | 3, 7 |
| `user`, `group`, `ansible.posix.authorized_key` | accounts and keys | 3, 8, 14 |
| `service` | daemons | 3, 9 |
| `uri`, `get_url`, `wait_for` | web and readiness | 8, 12 |
| `iptables` | host firewall | 14 |
| `set_fact`, `meta`, `include_role`, `import_role` | control | 6, 9, 10 |

All built-in modules have a full name `ansible.builtin.NAME`; use it in playbooks.

## Variable precedence (simplified, weakest first)

1. role `defaults/`
2. inventory: `group_vars/all`, other groups, `host_vars`
3. facts
4. play `vars`, `vars_files`
5. role `vars/`, task `vars`, `register`, `set_fact`
6. extra vars, `-e`: always win

## Jinja2

| Syntax | Meaning |
|---|---|
| `{{ expr }}` | print an expression |
| `{% if %} ... {% elif %} ... {% else %} ... {% endif %}` | condition |
| `{% for x in list %} ... {% endfor %}` | loop; `.items()` for dictionaries |
| `{% set name = expr %}` | local variable |
| `{# comment #}` | not in the output |
| `{%- ... -%}` | strip whitespace |

Filters used in this book: `default`, `upper`, `join`, `split`, `length`, `sort`, `unique`, `first`, `int`, `round`, `trim`, `replace`, `dict2items`, `subelements`, `map`, `select`, `selectattr`, `flatten`, `difference`, `from_json`, `to_nice_yaml`, `password_hash`. Tests: `defined`, `failed`, `succeeded`, `changed`, `eq`, `search`.

## Magic variables

`inventory_hostname`, `group_names`, `groups`, `hostvars`, `ansible_play_hosts`, `playbook_dir`, `ansible_facts`, `ansible_check_mode`, `ansible_failed_task` (in `rescue`), `ansible_managed` (in templates).

## Result states

| State | Meaning |
|---|---|
| ok | already as wanted |
| changed | a change was made |
| failed | the task did not reach its goal; the host stops |
| skipped | `when` was false |
| unreachable | no connection |
| rescued | a `rescue` handled a failure |
| ignored | failed, with `ignore_errors` |

## DockerLab notes

- Control: Debian 13, ansible-core 2.19 with the `ansible` collections, `ansible-lint`, `setup-keys HOST...`.
- Managed nodes: Debian 13, Python 3, SSH, user `student` with password-less sudo; no systemd, so `service` uses init scripts.
- A node's lab address is `ansible_facts['eth0']['ipv4']['address']`; `default_ipv4` is the internet connection, `nat0`.
- Redeploy resets everything, control included: keep the project in Git, or copy it out with *Files*.
