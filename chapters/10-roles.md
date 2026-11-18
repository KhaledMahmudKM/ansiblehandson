# Chapter 10: Roles and Collections

`site.yml` is now a hundred lines long. Its web play mixes package installs, templates, a sudo rule, nginx tuning, handlers and a health check; the templates sit in one shared folder; the variables have generic names like `site_name` that could clash with anything. Northwind's next projects, a load balancer and an API server, will need some of the same pieces. It is time to package each job as a **role**: a standard folder that holds everything for one job, reusable by any playbook with one line. This chapter splits `~/northwind` into three roles, `common`, `webserver` and `database`, and introduces **collections**, the way roles and modules are shared.

> **Try it:** DockerLab, *Ansible lab 9, Roles*. Same topology. The finished project is in `examples/ch10`.

## From one playbook to roles

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Refactoring a long site.yml into roles. The preflight play, the every-server play with banner and host list, the web play with packages, templates, tuning, handlers and a health check, and the database play become the roles common, webserver and database, and a short site.yml that applies them to groups." xmlns="http://www.w3.org/2000/svg">
<style>
.a10a{ font-family:var(--sans),sans-serif; }
.a10a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a10a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a10a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a10a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a10a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a10a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a10a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a10a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a10a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a10a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a10a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a10a .s{ fill:var(--text-dim); font-size:13px; }
.a10a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a10a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a10a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a10a .c{ fill:var(--text-faint); font-size:12.5px; }
.a10a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a10a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a10a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a10a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10a-a); }
.a10a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10a-a); marker-start:url(#a10a-s); }
.a10a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a10a-g); }
.a10a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-a); }
.a10a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a10a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a10a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a10a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a10a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a10a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10a-tri); }
.a10a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-tri); }
.a10a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10a-open); }
.a10a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-open); }
.a10a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10a-dia); }
.a10a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10a-odia); }
.a10a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10a-dia); marker-end:url(#a10a-open); }
.a10a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10a-solid); }
.a10a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a10a-open); }
.a10a .fill{ fill:var(--text); stroke:none; }
.a10a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a10a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a10a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a10a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a10a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a10a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a10a .ks-blue{ fill:#185FA5; font-size:13px; }
.a10a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a10a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a10a .kb-blue, :root[data-theme="dark"] .a10a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a10a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a10a .kh-blue{ fill:#85B7EB; }
.a10a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a10a-blue); }
.a10a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-blue); }
.a10a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a10a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a10a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a10a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a10a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a10a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a10a .kb-teal, :root[data-theme="dark"] .a10a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a10a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a10a .kh-teal{ fill:#5DCAA5; }
.a10a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a10a-teal); }
.a10a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-teal); }
.a10a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a10a .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a10a .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a10a .ks-amber{ fill:#854F0B; font-size:13px; }
.a10a .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a10a .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10a .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a10a .kb-amber, :root[data-theme="dark"] .a10a .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a10a .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a10a .kh-amber{ fill:#EF9F27; }
.a10a .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a10a-amber); }
.a10a .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-amber); }
.a10a .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a10a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a10a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a10a .ks-purple{ fill:#534AB7; font-size:13px; }
.a10a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a10a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a10a .kb-purple, :root[data-theme="dark"] .a10a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a10a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a10a .kh-purple{ fill:#AFA9EC; }
.a10a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a10a-purple); }
.a10a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-purple); }
.a10a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a10a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a10a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a10a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a10a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a10a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a10a .kb-gray, :root[data-theme="dark"] .a10a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a10a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a10a .kh-gray{ fill:#B4B2A9; }
.a10a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a10a-gray); }
.a10a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10a-gray); }
.a10a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a10a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a10a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a10a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a10a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a10a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a10a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a10a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a10a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a10a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a10a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a10a-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a10a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a10a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a10a">
<rect x="20" y="16" width="260" height="260" rx="10" class="k-gray"/>
<text x="150" y="40" text-anchor="middle" class="kb-gray">site.yml: 150 lines</text>
<rect x="36" y="54" width="228" height="34" rx="5" class="k-amber"/>
<text x="150.0" y="75.5" text-anchor="middle" class="ks-amber">preflight assert</text>
<rect x="36" y="96" width="228" height="34" rx="5" class="k-amber"/>
<text x="150.0" y="117.5" text-anchor="middle" class="ks-amber">banner, host list</text>
<rect x="36" y="138" width="228" height="34" rx="5" class="k-blue"/>
<text x="150.0" y="159.5" text-anchor="middle" class="ks-blue">nginx, templates, tuning</text>
<rect x="36" y="180" width="228" height="34" rx="5" class="k-blue"/>
<text x="150.0" y="201.5" text-anchor="middle" class="ks-blue">handlers, health check</text>
<rect x="36" y="222" width="228" height="34" rx="5" class="k-teal"/>
<text x="150.0" y="243.5" text-anchor="middle" class="ks-teal">sqlite, data folder</text>
<rect x="360" y="46" width="160" height="40" rx="6" class="k-amber"/>
<text x="440.0" y="70.5" text-anchor="middle" class="km-amber">roles/common</text>
<rect x="360" y="130" width="160" height="40" rx="6" class="k-blue"/>
<text x="440.0" y="154.5" text-anchor="middle" class="km-blue">roles/webserver</text>
<rect x="360" y="210" width="160" height="40" rx="6" class="k-teal"/>
<text x="440.0" y="234.5" text-anchor="middle" class="km-teal">roles/database</text>
<path d="M266 92 C 310 92, 320 66, 356 66" class="ar-amber"/>
<path d="M266 155 C 310 155, 320 150, 356 150" class="ar-blue"/>
<path d="M266 238 C 310 238, 320 230, 356 230" class="ar-teal"/>
<rect x="550" y="70" width="160" height="104" rx="6" class="k-purple"/>
<text x="562.0" y="90" text-anchor="start" class="m">site.yml:</text>
<text x="562.0" y="112" text-anchor="start" class="m">all:  common</text>
<text x="562.0" y="134" text-anchor="start" class="m">web:  webserver</text>
<text x="562.0" y="156" text-anchor="start" class="m">db:   database</text>
</g>
</svg>
<figcaption>The same work, packaged by job.</figcaption>
</figure>

A role is a folder under `roles/` with a fixed layout. Ansible finds each part by its folder name, so a task in the role says `src: site.conf.j2` and Ansible looks in the role's own `templates/`:

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The webserver role&#x27;s folders: tasks/main.yml with the role&#x27;s tasks, handlers/main.yml with its handlers, templates holding site.conf.j2 and index.html.j2, defaults/main.yml with webserver_port and the other settings, and meta/main.yml with author, licence, platforms and dependencies." xmlns="http://www.w3.org/2000/svg">
<style>
.a10b{ font-family:var(--sans),sans-serif; }
.a10b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a10b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a10b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a10b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a10b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a10b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a10b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a10b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a10b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a10b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a10b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a10b .s{ fill:var(--text-dim); font-size:13px; }
.a10b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a10b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a10b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a10b .c{ fill:var(--text-faint); font-size:12.5px; }
.a10b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a10b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a10b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a10b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10b-a); }
.a10b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10b-a); marker-start:url(#a10b-s); }
.a10b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a10b-g); }
.a10b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-a); }
.a10b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a10b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a10b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a10b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a10b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a10b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10b-tri); }
.a10b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-tri); }
.a10b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10b-open); }
.a10b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-open); }
.a10b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10b-dia); }
.a10b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10b-odia); }
.a10b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10b-dia); marker-end:url(#a10b-open); }
.a10b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10b-solid); }
.a10b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a10b-open); }
.a10b .fill{ fill:var(--text); stroke:none; }
.a10b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a10b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a10b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a10b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a10b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a10b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a10b .ks-blue{ fill:#185FA5; font-size:13px; }
.a10b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a10b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a10b .kb-blue, :root[data-theme="dark"] .a10b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a10b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a10b .kh-blue{ fill:#85B7EB; }
.a10b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a10b-blue); }
.a10b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-blue); }
.a10b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a10b .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a10b .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a10b .ks-coral{ fill:#993C1D; font-size:13px; }
.a10b .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a10b .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10b .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a10b .kb-coral, :root[data-theme="dark"] .a10b .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a10b .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a10b .kh-coral{ fill:#F0997B; }
.a10b .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a10b-coral); }
.a10b .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-coral); }
.a10b .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a10b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a10b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a10b .ks-purple{ fill:#534AB7; font-size:13px; }
.a10b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a10b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a10b .kb-purple, :root[data-theme="dark"] .a10b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a10b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a10b .kh-purple{ fill:#AFA9EC; }
.a10b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a10b-purple); }
.a10b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-purple); }
.a10b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a10b .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a10b .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a10b .ks-green{ fill:#3B6D11; font-size:13px; }
.a10b .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a10b .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10b .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a10b .kb-green, :root[data-theme="dark"] .a10b .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a10b .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a10b .kh-green{ fill:#97C459; }
.a10b .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a10b-green); }
.a10b .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-green); }
.a10b .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a10b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a10b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a10b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a10b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a10b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a10b .kb-gray, :root[data-theme="dark"] .a10b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a10b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a10b .kh-gray{ fill:#B4B2A9; }
.a10b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a10b-gray); }
.a10b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10b-gray); }
.a10b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a10b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a10b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a10b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a10b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a10b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a10b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a10b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a10b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a10b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a10b-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a10b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a10b-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a10b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a10b">
<rect x="20" y="16" width="280" height="268" rx="6" class="pnl"/>
<text x="32.0" y="36" text-anchor="start" class="m">roles/webserver/</text>
<text x="32.0" y="57" text-anchor="start" class="m">├── tasks/</text>
<text x="32.0" y="78" text-anchor="start" class="m">│   └── main.yml</text>
<text x="32.0" y="99" text-anchor="start" class="m">├── handlers/</text>
<text x="32.0" y="120" text-anchor="start" class="m">│   └── main.yml</text>
<text x="32.0" y="141" text-anchor="start" class="m">├── templates/</text>
<text x="32.0" y="162" text-anchor="start" class="m">│   ├── site.conf.j2</text>
<text x="32.0" y="183" text-anchor="start" class="m">│   └── index.html.j2</text>
<text x="32.0" y="204" text-anchor="start" class="m">├── defaults/</text>
<text x="32.0" y="225" text-anchor="start" class="m">│   └── main.yml</text>
<text x="32.0" y="246" text-anchor="start" class="m">└── meta/</text>
<text x="62.4" y="267" text-anchor="start" class="m">└── main.yml</text>
<line x1="304" y1="66" x2="340" y2="66" class="ln-blue"/>
<text x="348" y="70" text-anchor="start" class="kh-blue">the work: install, configure, start, check</text>
<line x1="304" y1="108" x2="340" y2="108" class="ln-coral"/>
<text x="348" y="112" text-anchor="start" class="kh-coral">Reload nginx, Check the configuration</text>
<line x1="304" y1="171" x2="340" y2="171" class="ln-purple"/>
<text x="348" y="175" text-anchor="start" class="kh-purple">found by name: src: site.conf.j2</text>
<line x1="304" y1="213" x2="340" y2="213" class="ln-green"/>
<text x="348" y="217" text-anchor="start" class="kh-green">webserver_port: 80 ... (lowest priority)</text>
<line x1="304" y1="255" x2="340" y2="255" class="ln-gray"/>
<text x="348" y="259" text-anchor="start" class="kh-gray">galaxy_info, dependencies: []</text>
</g>
</svg>
<figcaption>Convention over configuration: each part has its place.</figcaption>
</figure>

| Folder | Holds |
|---|---|
| `tasks/main.yml` | the role's tasks: its work |
| `handlers/main.yml` | its handlers |
| `templates/`, `files/` | templates and files its tasks use, found by name alone |
| `defaults/main.yml` | variables with the **lowest** priority: the settings users of the role are meant to change |
| `vars/main.yml` | variables with **high** priority: the role's internal constants |
| `meta/main.yml` | author, licence, supported platforms, and roles it depends on |

`ansible-galaxy role init roles/webserver` creates the whole skeleton; delete the folders you don't use.

## The webserver role

The web play's tasks move to `roles/webserver/tasks/main.yml`, unchanged except for their variable names:

```yaml
---
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    update_cache: true
    cache_valid_time: 3600
  tags: packages

- name: Site configuration
  ansible.builtin.template:
    src: site.conf.j2
    dest: /etc/nginx/sites-available/default
    mode: "0644"
  notify: Reload nginx
  tags: config

- name: Hide the nginx version in error pages
  ansible.builtin.lineinfile:
    path: /etc/nginx/nginx.conf
    regexp: '^\s*#?\s*server_tokens'
    line: '        server_tokens off;'
  notify: Reload nginx
  tags: config

- name: Home page
  ansible.builtin.template:
    src: index.html.j2
    dest: "{{ webserver_root }}/index.html"
    mode: "0644"
  tags: content

- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started

- name: Apply any configuration change now, before testing
  ansible.builtin.meta: flush_handlers

- name: Wait until the site answers
  ansible.builtin.uri:
    url: "http://{{ inventory_hostname }}:{{ webserver_port }}/"
  register: webserver_page
  until: webserver_page.status == 200
  retries: 5
  delay: 2
  delegate_to: localhost
  become: false
```

The two handlers go to `roles/webserver/handlers/main.yml`, the two templates to `roles/webserver/templates/`, and the role's settings, with safe values, to `roles/webserver/defaults/main.yml`:

```yaml
---
webserver_port: 80
webserver_root: /var/www/html
webserver_site_name: Welcome
webserver_aliases: []
webserver_opening_hours: {}
```

Every variable now starts with the role's name. Roles from different authors end up in one project; prefixes make sure `port` in one never overwrites `port` in another. `ansible-lint` enforces it (`var-naming[no-role-prefix]`).

## The common and database roles

**common** is the baseline every server gets: the preflight check, a few packages, the login banner and the fleet's host list.

```yaml
---
- name: Refuse hosts this role doesn't support
  ansible.builtin.assert:
    that:
      - ansible_facts['os_family'] == 'Debian'
      - ansible_facts['memtotal_mb'] >= 256
    fail_msg: "{{ inventory_hostname }} is not a supported host"
    quiet: true

- name: Baseline packages
  ansible.builtin.apt:
    name: "{{ common_packages }}"
    update_cache: true
    cache_valid_time: 3600

- name: Login message
  ansible.builtin.template:
    src: motd.j2
    dest: /etc/motd
    mode: "0644"

- name: Every server knows every other
  ansible.builtin.template:
    src: lab-hosts.j2
    dest: /etc/lab-hosts
    mode: "0644"
```

with `common_company: Northwind`, `common_role_description: server` and `common_packages: [curl, tree, less]` as defaults. **database** installs SQLite and makes the data folder, from `database_data_dir` and `database_owner`.

## site.yml, now

```yaml
---
- name: Every server
  hosts: all
  become: true
  roles:
    - common

- name: Web servers
  hosts: web
  become: true
  roles:
    - webserver

- name: Database servers
  hosts: db
  become: true
  roles:
    - database
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="site.yml maps roles to inventory groups. Play 1 applies common to all hosts: web1, web2 and db1. Play 2 applies webserver to the web group: web1 and web2. Play 3 applies database to the db group: db1." xmlns="http://www.w3.org/2000/svg">
<style>
.a10c{ font-family:var(--sans),sans-serif; }
.a10c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a10c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a10c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a10c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a10c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a10c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a10c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a10c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a10c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a10c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a10c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a10c .s{ fill:var(--text-dim); font-size:13px; }
.a10c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a10c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a10c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a10c .c{ fill:var(--text-faint); font-size:12.5px; }
.a10c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a10c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a10c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a10c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10c-a); }
.a10c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10c-a); marker-start:url(#a10c-s); }
.a10c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a10c-g); }
.a10c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a10c-a); }
.a10c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a10c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a10c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a10c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a10c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a10c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10c-tri); }
.a10c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10c-tri); }
.a10c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10c-open); }
.a10c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10c-open); }
.a10c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10c-dia); }
.a10c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10c-odia); }
.a10c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10c-dia); marker-end:url(#a10c-open); }
.a10c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10c-solid); }
.a10c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a10c-open); }
.a10c .fill{ fill:var(--text); stroke:none; }
.a10c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a10c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a10c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a10c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a10c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a10c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a10c .ks-blue{ fill:#185FA5; font-size:13px; }
.a10c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a10c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a10c .kb-blue, :root[data-theme="dark"] .a10c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a10c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a10c .kh-blue{ fill:#85B7EB; }
.a10c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a10c-blue); }
.a10c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10c-blue); }
.a10c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a10c .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a10c .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a10c .ks-teal{ fill:#0F6E56; font-size:13px; }
.a10c .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a10c .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10c .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a10c .kb-teal, :root[data-theme="dark"] .a10c .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a10c .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a10c .kh-teal{ fill:#5DCAA5; }
.a10c .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a10c-teal); }
.a10c .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10c-teal); }
.a10c .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a10c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a10c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a10c .ks-amber{ fill:#854F0B; font-size:13px; }
.a10c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a10c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a10c .kb-amber, :root[data-theme="dark"] .a10c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a10c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a10c .kh-amber{ fill:#EF9F27; }
.a10c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a10c-amber); }
.a10c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10c-amber); }
.a10c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
</style>
<defs><marker id="a10c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a10c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a10c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a10c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a10c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a10c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a10c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a10c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a10c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a10c-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a10c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker></defs>
<g class="a10c">
<text x="470" y="30" text-anchor="middle" class="b">web1</text>
<text x="560" y="30" text-anchor="middle" class="b">web2</text>
<text x="650" y="30" text-anchor="middle" class="b">db1</text>
<rect x="20" y="50" width="300" height="48" rx="6" class="k-amber"/>
<text x="170.0" y="78.5" text-anchor="middle" class="km-amber">hosts: all   roles: [common]</text>
<rect x="430" y="56" width="80" height="36" rx="5" class="k-amber"/>
<text x="470.0" y="78.5" text-anchor="middle" class="ks-amber">common</text>
<rect x="520" y="56" width="80" height="36" rx="5" class="k-amber"/>
<text x="560.0" y="78.5" text-anchor="middle" class="ks-amber">common</text>
<rect x="610" y="56" width="80" height="36" rx="5" class="k-amber"/>
<text x="650.0" y="78.5" text-anchor="middle" class="ks-amber">common</text>
<path d="M322 74 L426 74" class="ar-amber"/>
<rect x="20" y="114" width="300" height="48" rx="6" class="k-blue"/>
<text x="170.0" y="142.5" text-anchor="middle" class="km-blue">hosts: web   roles: [webserver]</text>
<rect x="430" y="120" width="80" height="36" rx="5" class="k-blue"/>
<text x="470.0" y="142.5" text-anchor="middle" class="ks-blue">webserver</text>
<rect x="520" y="120" width="80" height="36" rx="5" class="k-blue"/>
<text x="560.0" y="142.5" text-anchor="middle" class="ks-blue">webserver</text>
<path d="M322 138 L426 138" class="ar-blue"/>
<rect x="20" y="178" width="300" height="48" rx="6" class="k-teal"/>
<text x="170.0" y="206.5" text-anchor="middle" class="km-teal">hosts: db   roles: [database]</text>
<rect x="610" y="184" width="80" height="36" rx="5" class="k-teal"/>
<text x="650.0" y="206.5" text-anchor="middle" class="ks-teal">database</text>
<path d="M322 202 L606 202" class="ar-teal"/>
<text x="360" y="240" text-anchor="middle" class="c">every server: baseline; each group: its own job on top</text>
</g>
</svg>
<figcaption>A site.yml of roles reads like a description of the infrastructure.</figcaption>
</figure>

`site.yml` now reads like a description of the infrastructure: every server gets *common*, web servers also get *webserver*, database servers *database*. Task names in the output carry their role's name, `TASK [webserver : Site configuration]`. This layout, `site.yml` plus roles plus `group_vars`, is how most Ansible projects are organized.

```
northwind/
├── ansible.cfg
├── inventory
├── site.yml
├── users.yml
├── group_vars/  all.yml  web.yml  db.yml  production.yml  staging.yml
├── host_vars/   web2.yml
└── roles/
    ├── common/     tasks/  templates/  defaults/  meta/
    ├── webserver/  tasks/  templates/  defaults/  handlers/  meta/
    └── database/   tasks/  defaults/  meta/
```

## Data lives outside the roles

The roles hold sensible defaults; Northwind's own values stay in `group_vars` and `host_vars`, where they were:

```yaml
# group_vars/web.yml
common_role_description: web server
webserver_site_name: Northwind Coffee
webserver_aliases:
  - www.northwind.lab
  - northwind.lab
webserver_opening_hours:
  Monday to Friday: 7:00 to 19:00
  Saturday: 8:00 to 17:00
  Sunday: closed
```

```yaml
# host_vars/web2.yml
webserver_site_name: Northwind Coffee (staging)
webserver_port: 8080
```

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The value of webserver_site_name for each host. The role default is Welcome. group_vars/web.yml sets Northwind Coffee, which web1 uses. host_vars/web2.yml sets Northwind Coffee (staging), which web2 uses. The role itself never changes." xmlns="http://www.w3.org/2000/svg">
<style>
.a10d{ font-family:var(--sans),sans-serif; }
.a10d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a10d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a10d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a10d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a10d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a10d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a10d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a10d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a10d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a10d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a10d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a10d .s{ fill:var(--text-dim); font-size:13px; }
.a10d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a10d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a10d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a10d .c{ fill:var(--text-faint); font-size:12.5px; }
.a10d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a10d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a10d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a10d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10d-a); }
.a10d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10d-a); marker-start:url(#a10d-s); }
.a10d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a10d-g); }
.a10d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a10d-a); }
.a10d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a10d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a10d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a10d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a10d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a10d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10d-tri); }
.a10d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10d-tri); }
.a10d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10d-open); }
.a10d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10d-open); }
.a10d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10d-dia); }
.a10d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10d-odia); }
.a10d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10d-dia); marker-end:url(#a10d-open); }
.a10d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10d-solid); }
.a10d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a10d-open); }
.a10d .fill{ fill:var(--text); stroke:none; }
.a10d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a10d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a10d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a10d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a10d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a10d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a10d .ks-blue{ fill:#185FA5; font-size:13px; }
.a10d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a10d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a10d .kb-blue, :root[data-theme="dark"] .a10d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a10d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a10d .kh-blue{ fill:#85B7EB; }
.a10d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a10d-blue); }
.a10d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10d-blue); }
.a10d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a10d .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a10d .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a10d .ks-coral{ fill:#993C1D; font-size:13px; }
.a10d .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a10d .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10d .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a10d .kb-coral, :root[data-theme="dark"] .a10d .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a10d .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a10d .kh-coral{ fill:#F0997B; }
.a10d .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a10d-coral); }
.a10d .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10d-coral); }
.a10d .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a10d .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a10d .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a10d .ks-purple{ fill:#534AB7; font-size:13px; }
.a10d .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a10d .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10d .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a10d .kb-purple, :root[data-theme="dark"] .a10d .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a10d .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a10d .kh-purple{ fill:#AFA9EC; }
.a10d .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a10d-purple); }
.a10d .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10d-purple); }
.a10d .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a10d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a10d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a10d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a10d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a10d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a10d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a10d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a10d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a10d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a10d-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a10d-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a10d">
<rect x="20" y="150" width="200" height="60" rx="7" class="k-purple"/>
<text x="120.0" y="175.5" text-anchor="middle" class="kb-purple">role defaults</text>
<text x="120.0" y="193.5" text-anchor="middle" class="km-purple">Welcome</text>
<rect x="250" y="90" width="210" height="60" rx="7" class="k-blue"/>
<text x="355.0" y="115.5" text-anchor="middle" class="km-blue">group_vars/web.yml</text>
<text x="355.0" y="133.5" text-anchor="middle" class="ks-blue">Northwind Coffee</text>
<rect x="490" y="30" width="210" height="60" rx="7" class="k-coral"/>
<text x="595.0" y="55.5" text-anchor="middle" class="km-coral">host_vars/web2.yml</text>
<text x="595.0" y="73.5" text-anchor="middle" class="ks-coral">Northwind Coffee (staging)</text>
<path d="M222 180 L246 130" class="ag"/>
<path d="M462 120 L486 72" class="ag"/>
<text x="355" y="190" text-anchor="middle" class="kh-blue">web1 uses this</text>
<text x="595" y="120" text-anchor="middle" class="kh-coral">web2 uses this</text>
<text x="120" y="230" text-anchor="middle" class="c"></text>
</g>
</svg>
<figcaption>Role defaults are the floor; inventory data builds on them.</figcaption>
</figure>

Role **defaults** have the lowest priority of all variables, so anything in the inventory overrides them. That is what makes a role reusable: the same *webserver* role could serve a second company's site tomorrow, with a different `group_vars/web.yml` and no change to the role.

One trap: a role's defaults are loaded only in the plays that **use** the role. The banner template in *common* mentions the web port, but *webserver*'s defaults aren't loaded in the *Every server* play. The template therefore writes `{{ webserver_port | default(80) }}`, so it works on web1, which has no `webserver_port` of its own.

## Using roles: three ways

```yaml
- name: A play can pass variables to a role
  hosts: web1
  become: true
  roles:
    - role: webserver
      vars:
        webserver_site_name: Closed for maintenance

- name: Or use a role among other tasks
  hosts: web2
  become: true
  tasks:
    - name: Before
      ansible.builtin.debug:
        msg: "about to apply the role on {{ inventory_hostname }}"

    - name: The role, as one task
      ansible.builtin.include_role:
        name: webserver
      vars:
        webserver_site_name: Back soon
```

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Three ways to use a role. roles: in a play runs the role before the play&#x27;s tasks. import_role is static: its tasks are inserted when the playbook is read. include_role is dynamic: decided when the task runs, so it can be looped or conditional." xmlns="http://www.w3.org/2000/svg">
<style>
.a10e{ font-family:var(--sans),sans-serif; }
.a10e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a10e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a10e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a10e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a10e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a10e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a10e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a10e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a10e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a10e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a10e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a10e .s{ fill:var(--text-dim); font-size:13px; }
.a10e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a10e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a10e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a10e .c{ fill:var(--text-faint); font-size:12.5px; }
.a10e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a10e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a10e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a10e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10e-a); }
.a10e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10e-a); marker-start:url(#a10e-s); }
.a10e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a10e-g); }
.a10e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a10e-a); }
.a10e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a10e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a10e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a10e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a10e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a10e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10e-tri); }
.a10e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10e-tri); }
.a10e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10e-open); }
.a10e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10e-open); }
.a10e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10e-dia); }
.a10e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10e-odia); }
.a10e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10e-dia); marker-end:url(#a10e-open); }
.a10e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10e-solid); }
.a10e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a10e-open); }
.a10e .fill{ fill:var(--text); stroke:none; }
.a10e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a10e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a10e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a10e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a10e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a10e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a10e .ks-blue{ fill:#185FA5; font-size:13px; }
.a10e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a10e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a10e .kb-blue, :root[data-theme="dark"] .a10e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a10e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a10e .kh-blue{ fill:#85B7EB; }
.a10e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a10e-blue); }
.a10e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10e-blue); }
.a10e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a10e .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a10e .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a10e .ks-teal{ fill:#0F6E56; font-size:13px; }
.a10e .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a10e .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10e .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a10e .kb-teal, :root[data-theme="dark"] .a10e .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a10e .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a10e .kh-teal{ fill:#5DCAA5; }
.a10e .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a10e-teal); }
.a10e .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10e-teal); }
.a10e .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a10e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a10e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a10e .ks-purple{ fill:#534AB7; font-size:13px; }
.a10e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a10e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a10e .kb-purple, :root[data-theme="dark"] .a10e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a10e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a10e .kh-purple{ fill:#AFA9EC; }
.a10e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a10e-purple); }
.a10e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10e-purple); }
.a10e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a10e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a10e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a10e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a10e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a10e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a10e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a10e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a10e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a10e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a10e-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a10e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a10e">
<rect x="20" y="30" width="216" height="50" rx="6" class="k-purple"/>
<text x="128.0" y="59.5" text-anchor="middle" class="km-purple">roles:</text>
<rect x="20" y="100" width="216" height="80" rx="7" class="pnl"/>
<text x="128.0" y="135.5" text-anchor="middle" class="s">play level, before tasks:</text>
<text x="128.0" y="153.5" text-anchor="middle" class="c">the usual case</text>
<path d="M128 82 L128 96" class="ar-purple"/>
<rect x="252" y="30" width="216" height="50" rx="6" class="k-blue"/>
<text x="360.0" y="59.5" text-anchor="middle" class="km-blue">import_role</text>
<rect x="252" y="100" width="216" height="80" rx="7" class="pnl"/>
<text x="360.0" y="135.5" text-anchor="middle" class="s">static, read in advance</text>
<text x="360.0" y="153.5" text-anchor="middle" class="c">like pasting its tasks</text>
<path d="M360 82 L360 96" class="ar-blue"/>
<rect x="484" y="30" width="216" height="50" rx="6" class="k-teal"/>
<text x="592.0" y="59.5" text-anchor="middle" class="km-teal">include_role</text>
<rect x="484" y="100" width="216" height="80" rx="7" class="pnl"/>
<text x="592.0" y="135.5" text-anchor="middle" class="s">dynamic, at run time</text>
<text x="592.0" y="153.5" text-anchor="middle" class="c">in loops, under when</text>
<path d="M592 82 L592 96" class="ar-teal"/>
<text x="360" y="214" text-anchor="middle" class="m">pre_tasks:  -&gt;  roles:  -&gt;  tasks:  -&gt;  post_tasks:   (the order inside a play)</text>
</g>
</svg>
<figcaption>Static or dynamic, play-level or task-level.</figcaption>
</figure>

| | When it's read | Can be in a loop or under `when` | Use it for |
|---|---|---|---|
| `roles:` | when the playbook is loaded | `when` applies to every task | the usual case: the play's job |
| `import_role` | when the playbook is loaded (static) | `when` copied onto each task | a role at a fixed place among tasks |
| `include_role` | when the task runs (dynamic) | yes, decided at run time | a role chosen by a variable, in a loop |

`roles:` run before the play's `tasks:`. Use `pre_tasks:` for anything that must come before them (Chapter 12).

## Role dependencies

A role can list roles it needs in `meta/main.yml`; they run first:

```yaml
dependencies:
  - role: common
```

Dependencies help when a role truly cannot work without another. Use them sparingly: they hide ordering inside roles. Northwind keeps *common* in its own play for every host, because the host list template needs every host's facts, and a dependency would apply *common* only to the web servers in the web play. A role listed twice in one play runs only once, unless its parameters differ.

## Collections

Roles share **tasks**; **collections** share everything else as well: modules, roles, plugins, filters, and playbooks, published under a two-part name, `namespace.collection`.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Collections. Ansible Galaxy and other servers publish collections named namespace.collection, such as community.general and ansible.posix. Each contains modules, roles, plugins and filters. ansible-galaxy collection install, driven by requirements.yml, puts them on the control node, where playbooks use them by full name, such as community.general.ufw." xmlns="http://www.w3.org/2000/svg">
<style>
.a10f{ font-family:var(--sans),sans-serif; }
.a10f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a10f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a10f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a10f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a10f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a10f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a10f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a10f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a10f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a10f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a10f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a10f .s{ fill:var(--text-dim); font-size:13px; }
.a10f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a10f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a10f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a10f .c{ fill:var(--text-faint); font-size:12.5px; }
.a10f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a10f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a10f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a10f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10f-a); }
.a10f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10f-a); marker-start:url(#a10f-s); }
.a10f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a10f-g); }
.a10f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a10f-a); }
.a10f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a10f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a10f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a10f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a10f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a10f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10f-tri); }
.a10f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10f-tri); }
.a10f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10f-open); }
.a10f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10f-open); }
.a10f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10f-dia); }
.a10f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10f-odia); }
.a10f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10f-dia); marker-end:url(#a10f-open); }
.a10f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10f-solid); }
.a10f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a10f-open); }
.a10f .fill{ fill:var(--text); stroke:none; }
.a10f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a10f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a10f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a10f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a10f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a10f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a10f .ks-blue{ fill:#185FA5; font-size:13px; }
.a10f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a10f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a10f .kb-blue, :root[data-theme="dark"] .a10f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a10f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a10f .kh-blue{ fill:#85B7EB; }
.a10f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a10f-blue); }
.a10f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10f-blue); }
.a10f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a10f .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a10f .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a10f .ks-purple{ fill:#534AB7; font-size:13px; }
.a10f .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a10f .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10f .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a10f .kb-purple, :root[data-theme="dark"] .a10f .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a10f .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a10f .kh-purple{ fill:#AFA9EC; }
.a10f .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a10f-purple); }
.a10f .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10f-purple); }
.a10f .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a10f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a10f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a10f .ks-green{ fill:#3B6D11; font-size:13px; }
.a10f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a10f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a10f .kb-green, :root[data-theme="dark"] .a10f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a10f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a10f .kh-green{ fill:#97C459; }
.a10f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a10f-green); }
.a10f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10f-green); }
.a10f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a10f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a10f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a10f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a10f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a10f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a10f .kb-gray, :root[data-theme="dark"] .a10f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a10f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a10f .kh-gray{ fill:#B4B2A9; }
.a10f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a10f-gray); }
.a10f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10f-gray); }
.a10f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a10f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a10f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a10f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a10f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a10f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a10f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a10f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a10f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a10f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a10f-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a10f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a10f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a10f">
<rect x="20" y="90" width="160" height="70" rx="7" class="k-gray"/>
<text x="100.0" y="120.5" text-anchor="middle" class="kb-gray">Ansible Galaxy</text>
<text x="100.0" y="138.5" text-anchor="middle" class="ks-gray">or a private hub</text>
<rect x="220" y="16" width="260" height="226" rx="10" class="k-green"/>
<text x="350" y="40" text-anchor="middle" class="kb-green">community.general</text>
<rect x="236" y="56" width="228" height="36" rx="5" class="pnl"/>
<text x="350.0" y="78.5" text-anchor="middle" class="s">modules: ufw, timezone, ini_file ...</text>
<rect x="236" y="100" width="228" height="36" rx="5" class="pnl"/>
<text x="350.0" y="122.5" text-anchor="middle" class="s">roles: packaged jobs</text>
<rect x="236" y="144" width="228" height="36" rx="5" class="pnl"/>
<text x="350.0" y="166.5" text-anchor="middle" class="s">plugins: inventory, lookup, callback</text>
<rect x="236" y="188" width="228" height="36" rx="5" class="pnl"/>
<text x="350.0" y="210.5" text-anchor="middle" class="s">filters: json_query, dict_kv ...</text>
<path d="M182 125 L216 125" class="ag"/>
<rect x="500" y="30" width="210" height="70" rx="7" class="k-purple"/>
<text x="605.0" y="50.5" text-anchor="middle" class="km-purple">requirements.yml</text>
<text x="605.0" y="69.5" text-anchor="middle" class="km-purple">ansible-galaxy</text>
<text x="605.0" y="88.5" text-anchor="middle" class="km-purple">collection install</text>
<rect x="500" y="130" width="210" height="88" rx="6" class="k-blue"/>
<text x="512.0" y="150" text-anchor="start" class="m">- name: Allow HTTP</text>
<text x="527.2" y="168" text-anchor="start" class="m">community.general.ufw:</text>
<text x="542.4" y="186" text-anchor="start" class="m">rule: allow</text>
<text x="542.4" y="204" text-anchor="start" class="m">port: "80"</text>
<path d="M482 120 L496 150" class="ar-green"/>
</g>
</svg>
<figcaption>Collections are how modules, roles and plugins are packaged and shared.</figcaption>
</figure>

```
ansible-galaxy collection list | grep -E '^(community\.general|ansible\.posix) '
ansible-doc -l community.general | wc -l
```

The DockerLab control node already has the **ansible** package's collections: `community.general` alone has hundreds of modules (`ufw`, `timezone`, `ini_file`, `npm` ...), and `ansible.posix` has `authorized_key`, `sysctl` and `firewalld`. A project lists the collections it relies on in `requirements.yml`, so anyone can install them:

```yaml
---
collections:
  - name: community.general
  - name: ansible.posix
```

```
ansible-galaxy collection install -r requirements.yml
```

**Ansible Galaxy** (galaxy.ansible.com) hosts thousands of community roles and collections. Read a role before you use it: it runs as root on your servers.

## Checking with ansible-lint

`ansible-lint` checks playbooks and roles against good practice: full module names, named tasks, quoted modes, prefixed role variables, idempotent commands, YAML style, and more, grouped into **profiles** from `min` to `production`.

```
ansible-lint
```

```
Passed: 0 failure(s), 0 warning(s) in 26 files processed of 28 encountered.
Last profile that met the validation criteria was 'production'.
```

<figure class="diagram">
<svg viewBox="0 0 720 190" style="width:100%;height:auto;max-width:720px" role="img" aria-label="ansible-lint in the workflow: edit roles and playbooks, run ansible-lint locally, fix findings such as name missing or no-changed-when, commit, and let CI run ansible-lint again. Profiles range from min through basic, moderate, safety and shared to production." xmlns="http://www.w3.org/2000/svg">
<style>
.a10g{ font-family:var(--sans),sans-serif; }
.a10g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a10g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a10g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a10g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a10g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a10g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a10g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a10g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a10g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a10g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a10g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a10g .s{ fill:var(--text-dim); font-size:13px; }
.a10g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a10g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a10g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a10g .c{ fill:var(--text-faint); font-size:12.5px; }
.a10g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a10g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a10g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a10g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10g-a); }
.a10g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a10g-a); marker-start:url(#a10g-s); }
.a10g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a10g-g); }
.a10g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-a); }
.a10g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a10g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a10g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a10g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a10g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a10g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10g-tri); }
.a10g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-tri); }
.a10g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10g-open); }
.a10g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-open); }
.a10g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10g-dia); }
.a10g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10g-odia); }
.a10g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a10g-dia); marker-end:url(#a10g-open); }
.a10g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a10g-solid); }
.a10g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a10g-open); }
.a10g .fill{ fill:var(--text); stroke:none; }
.a10g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a10g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a10g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a10g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a10g .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a10g .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a10g .ks-blue{ fill:#185FA5; font-size:13px; }
.a10g .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a10g .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10g .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a10g .kb-blue, :root[data-theme="dark"] .a10g .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a10g .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a10g .kh-blue{ fill:#85B7EB; }
.a10g .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a10g-blue); }
.a10g .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-blue); }
.a10g .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a10g .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a10g .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a10g .ks-amber{ fill:#854F0B; font-size:13px; }
.a10g .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a10g .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10g .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a10g .kb-amber, :root[data-theme="dark"] .a10g .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a10g .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a10g .kh-amber{ fill:#EF9F27; }
.a10g .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a10g-amber); }
.a10g .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-amber); }
.a10g .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a10g .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a10g .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a10g .ks-coral{ fill:#993C1D; font-size:13px; }
.a10g .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a10g .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10g .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a10g .kb-coral, :root[data-theme="dark"] .a10g .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a10g .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a10g .kh-coral{ fill:#F0997B; }
.a10g .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a10g-coral); }
.a10g .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-coral); }
.a10g .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a10g .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a10g .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a10g .ks-green{ fill:#3B6D11; font-size:13px; }
.a10g .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a10g .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10g .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a10g .kb-green, :root[data-theme="dark"] .a10g .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a10g .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a10g .kh-green{ fill:#97C459; }
.a10g .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a10g-green); }
.a10g .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-green); }
.a10g .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a10g .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a10g .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a10g .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a10g .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a10g .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a10g .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a10g .kb-gray, :root[data-theme="dark"] .a10g .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a10g .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a10g .kh-gray{ fill:#B4B2A9; }
.a10g .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a10g-gray); }
.a10g .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a10g-gray); }
.a10g .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a10g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a10g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a10g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a10g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a10g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a10g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a10g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a10g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a10g-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a10g-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a10g-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a10g-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a10g-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a10g">
<rect x="20" y="30" width="126" height="56" rx="7" class="k-blue"/>
<text x="83.0" y="53.5" text-anchor="middle" class="kb-blue">edit</text>
<text x="83.0" y="71.5" text-anchor="middle" class="ks-blue">roles, playbooks</text>
<path d="M148 58 L158 58" class="ar"/>
<rect x="160" y="30" width="126" height="56" rx="7" class="k-amber"/>
<text x="223.0" y="53.5" text-anchor="middle" class="kb-amber">ansible-lint</text>
<text x="223.0" y="71.5" text-anchor="middle" class="ks-amber">locally</text>
<path d="M288 58 L298 58" class="ar"/>
<rect x="300" y="30" width="126" height="56" rx="7" class="k-coral"/>
<text x="363.0" y="53.5" text-anchor="middle" class="kb-coral">fix findings</text>
<text x="363.0" y="71.5" text-anchor="middle" class="ks-coral">or skip by rule</text>
<path d="M428 58 L438 58" class="ar"/>
<rect x="440" y="30" width="126" height="56" rx="7" class="k-gray"/>
<text x="503.0" y="53.5" text-anchor="middle" class="kb-gray">commit</text>
<text x="503.0" y="71.5" text-anchor="middle" class="ks-gray">Git</text>
<path d="M568 58 L578 58" class="ar"/>
<rect x="580" y="30" width="126" height="56" rx="7" class="k-green"/>
<text x="643.0" y="53.5" text-anchor="middle" class="kb-green">CI runs it</text>
<text x="643.0" y="71.5" text-anchor="middle" class="ks-green">on every change</text>
<rect x="20" y="120" width="105" height="32" rx="5" class="pnl"/>
<text x="72.5" y="140.5" text-anchor="middle" class="m">min</text>
<rect x="135" y="120" width="105" height="32" rx="5" class="pnl"/>
<text x="187.5" y="140.5" text-anchor="middle" class="m">basic</text>
<rect x="250" y="120" width="105" height="32" rx="5" class="pnl"/>
<text x="302.5" y="140.5" text-anchor="middle" class="m">moderate</text>
<rect x="365" y="120" width="105" height="32" rx="5" class="pnl"/>
<text x="417.5" y="140.5" text-anchor="middle" class="m">safety</text>
<rect x="480" y="120" width="105" height="32" rx="5" class="pnl"/>
<text x="532.5" y="140.5" text-anchor="middle" class="m">shared</text>
<rect x="595" y="120" width="105" height="32" rx="5" class="k-green"/>
<text x="647.5" y="140.5" text-anchor="middle" class="km-green">production</text>
<text x="360" y="176" text-anchor="middle" class="c">profiles, from least to most demanding</text>
</g>
</svg>
<figcaption>Lint early, lint often.</figcaption>
</figure>

Northwind's project passes the strictest profile, because every chapter followed its rules: full names, named tasks, quoted modes, `changed_when` on read-only commands, prefixed variables. Run it before every commit; Chapter 14 runs it in a pipeline. When it complains, read the rule's name (`name[missing]`, `no-changed-when`, `risky-file-permissions` ...) and fix what you agree with; a rule you consciously reject can be skipped in `.ansible-lint`.

## Hands on

1. Add `web1` to a new play that applies *webserver* with `webserver_site_name: Closed for maintenance`, run it, then run `site.yml` to restore the site.
2. Add a `webserver_extra_sites` list and a template that writes one `server` block per site.
3. Put the project in Git: `git init`, `git add .`, `git commit -m "Northwind roles"`. Change a default and look at `git diff`.
4. `ansible-galaxy role install geerlingguy.nginx -p /tmp/roles` (needs the internet): how is a widely used role organized?

## Summary

- A **role** packages one job: `tasks/`, `handlers/`, `templates/`, `files/`, `defaults/`, `vars/`, `meta/`, each found by name.
- **Prefix** role variables with the role's name; put user-facing settings in `defaults/`, the lowest priority, so `group_vars` and `host_vars` override them.
- `site.yml` assigns roles to groups; data stays outside the roles.
- Use roles with `roles:`, `import_role` (static) or `include_role` (dynamic); dependencies in `meta/main.yml` run first.
- **Collections** (`namespace.collection`) ship modules, roles and plugins; list them in `requirements.yml`.
- `ansible-lint` checks a project against good practice; aim for the `production` profile.
