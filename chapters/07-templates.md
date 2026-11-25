# Chapter 7: Templates

Northwind's site configuration is growing: server aliases for `www.northwind.lab`, opening hours on the home page, a login banner that says what each server is, and a file on every server listing every other. Writing these files inline in `copy` tasks is already awkward, and each differs a little from host to host. **Templates** are files with blanks that Ansible fills in for each host, using the **Jinja2** language: variables, conditions, loops and filters. This chapter moves Northwind's files into templates.

> **Try it:** DockerLab, *Ansible lab 6, Templates*. Same topology. The finished project is in `examples/ch07`.

## How the template module works

The `template` module reads a `.j2` file **on the control node**, renders it with the target host's variables and facts, and writes the result to the host. Like `copy`, it changes the file only if the result differs, and reports a diff with `--diff`.

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Rendering a template. On control, site.conf.j2 contains listen {{ http_port }} and server_name {{ inventory_hostname }}. For web1, with http_port 80, the rendered file says listen 80 and server_name web1; for web2, with http_port 8080, it says listen 8080 and server_name web2. Each result is written to its host." xmlns="http://www.w3.org/2000/svg">
<style>
.a7a{ font-family:var(--sans),sans-serif; }
.a7a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a7a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a7a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a7a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a7a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a7a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a7a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a7a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a7a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a7a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a7a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a7a .s{ fill:var(--text-dim); font-size:13px; }
.a7a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a7a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a7a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a7a .c{ fill:var(--text-faint); font-size:12.5px; }
.a7a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a7a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a7a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a7a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7a-a); }
.a7a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7a-a); marker-start:url(#a7a-s); }
.a7a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a7a-g); }
.a7a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a7a-a); }
.a7a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a7a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a7a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a7a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a7a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a7a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7a-tri); }
.a7a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7a-tri); }
.a7a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7a-open); }
.a7a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7a-open); }
.a7a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7a-dia); }
.a7a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7a-odia); }
.a7a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7a-dia); marker-end:url(#a7a-open); }
.a7a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7a-solid); }
.a7a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a7a-open); }
.a7a .fill{ fill:var(--text); stroke:none; }
.a7a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a7a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a7a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a7a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a7a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a7a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a7a .ks-blue{ fill:#185FA5; font-size:13px; }
.a7a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a7a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a7a .kb-blue, :root[data-theme="dark"] .a7a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a7a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a7a .kh-blue{ fill:#85B7EB; }
.a7a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a7a-blue); }
.a7a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7a-blue); }
.a7a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a7a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a7a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a7a .ks-purple{ fill:#534AB7; font-size:13px; }
.a7a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a7a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a7a .kb-purple, :root[data-theme="dark"] .a7a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a7a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a7a .kh-purple{ fill:#AFA9EC; }
.a7a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a7a-purple); }
.a7a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7a-purple); }
.a7a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a7a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a7a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a7a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a7a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a7a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a7a .kb-gray, :root[data-theme="dark"] .a7a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a7a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a7a .kh-gray{ fill:#B4B2A9; }
.a7a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a7a-gray); }
.a7a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7a-gray); }
.a7a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a7a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a7a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a7a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a7a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a7a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a7a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a7a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a7a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a7a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a7a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a7a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a7a">
<rect x="10" y="20" width="340" height="250" rx="10" class="k-purple"/>
<text x="180" y="44" text-anchor="middle" class="kb-purple">on control</text>
<rect x="22" y="60" width="316" height="74" rx="6" class="k-gray"/>
<text x="32" y="80" text-anchor="start" class="kh-gray">templates/site.conf.j2</text>
<text x="34.0" y="102" text-anchor="start" class="m">listen {{ http_port }};</text>
<text x="34.0" y="120" text-anchor="start" class="m">server_name {{ inventory_hostname }};</text>
<rect x="22" y="160" width="150" height="92" rx="7" class="pnl"/>
<text x="97.0" y="201.5" text-anchor="middle" class="b">web1</text>
<text x="97.0" y="219.5" text-anchor="middle" class="m">http_port: 80</text>
<rect x="188" y="160" width="150" height="92" rx="7" class="pnl"/>
<text x="263.0" y="201.5" text-anchor="middle" class="b">web2</text>
<text x="263.0" y="219.5" text-anchor="middle" class="m">http_port: 8080</text>
<text x="180" y="150" text-anchor="middle" class="c">+ each host's variables and facts</text>
<rect x="400" y="30" width="310" height="100" rx="10" class="k-blue"/>
<text x="414" y="54" text-anchor="start" class="kb-blue">web1: /etc/nginx/sites-available/default</text>
<rect x="414" y="66" width="280" height="52" rx="6" class="pnl"/>
<text x="426.0" y="86" text-anchor="start" class="m">listen 80;</text>
<text x="426.0" y="104" text-anchor="start" class="m">server_name web1;</text>
<path d="M352 120 C 370 120, 360 90, 396 90" class="ar"/>
<rect x="400" y="155" width="310" height="100" rx="10" class="k-blue"/>
<text x="414" y="179" text-anchor="start" class="kb-blue">web2: /etc/nginx/sites-available/default</text>
<rect x="414" y="191" width="280" height="52" rx="6" class="pnl"/>
<text x="426.0" y="211" text-anchor="start" class="m">listen 8080;</text>
<text x="426.0" y="229" text-anchor="start" class="m">server_name web2;</text>
<path d="M352 200 C 370 200, 360 215, 396 215" class="ar"/>
</g>
</svg>
<figcaption>One template, a different file for every host.</figcaption>
</figure>

Templates conventionally live in a `templates/` folder beside the playbook (in a role, its own `templates/`, Chapter 10).

## Jinja2 in one page

A template is plain text with three kinds of markup:

<figure class="diagram">
<svg viewBox="0 0 720 200" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The three kinds of Jinja2 markup. Double braces print an expression, such as a variable through a filter. Brace-percent tags hold logic: if, for, set, with matching end tags. Brace-hash marks a comment that is not copied to the output." xmlns="http://www.w3.org/2000/svg">
<style>
.a7b{ font-family:var(--sans),sans-serif; }
.a7b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a7b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a7b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a7b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a7b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a7b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a7b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a7b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a7b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a7b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a7b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a7b .s{ fill:var(--text-dim); font-size:13px; }
.a7b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a7b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a7b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a7b .c{ fill:var(--text-faint); font-size:12.5px; }
.a7b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a7b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a7b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a7b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7b-a); }
.a7b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7b-a); marker-start:url(#a7b-s); }
.a7b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a7b-g); }
.a7b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a7b-a); }
.a7b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a7b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a7b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a7b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a7b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a7b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7b-tri); }
.a7b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7b-tri); }
.a7b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7b-open); }
.a7b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7b-open); }
.a7b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7b-dia); }
.a7b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7b-odia); }
.a7b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7b-dia); marker-end:url(#a7b-open); }
.a7b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7b-solid); }
.a7b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a7b-open); }
.a7b .fill{ fill:var(--text); stroke:none; }
.a7b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a7b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a7b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a7b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a7b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a7b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a7b .ks-blue{ fill:#185FA5; font-size:13px; }
.a7b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a7b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a7b .kb-blue, :root[data-theme="dark"] .a7b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a7b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a7b .kh-blue{ fill:#85B7EB; }
.a7b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a7b-blue); }
.a7b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7b-blue); }
.a7b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a7b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a7b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a7b .ks-purple{ fill:#534AB7; font-size:13px; }
.a7b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a7b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a7b .kb-purple, :root[data-theme="dark"] .a7b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a7b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a7b .kh-purple{ fill:#AFA9EC; }
.a7b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a7b-purple); }
.a7b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7b-purple); }
.a7b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a7b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a7b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a7b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a7b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a7b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a7b .kb-gray, :root[data-theme="dark"] .a7b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a7b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a7b .kh-gray{ fill:#B4B2A9; }
.a7b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a7b-gray); }
.a7b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7b-gray); }
.a7b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a7b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a7b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a7b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a7b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a7b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a7b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a7b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a7b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a7b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a7b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a7b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a7b">
<rect x="20" y="20" width="216" height="50" rx="6" class="k-blue"/>
<text x="128.0" y="49.5" text-anchor="middle" class="km-blue">{{ ... }}</text>
<text x="128" y="96" text-anchor="middle" class="kh-blue">print an expression</text>
<rect x="20" y="116" width="216" height="50" rx="6" class="pnl"/>
<text x="128.0" y="145.5" text-anchor="middle" class="m">{{ site_name | upper }}</text>
<rect x="252" y="20" width="216" height="50" rx="6" class="k-purple"/>
<text x="360.0" y="49.5" text-anchor="middle" class="km-purple">{% ... %}</text>
<text x="360" y="96" text-anchor="middle" class="kh-purple">logic: if, for, set</text>
<rect x="252" y="116" width="216" height="50" rx="6" class="pnl"/>
<text x="360.0" y="145.5" text-anchor="middle" class="m">{% for a in aliases %}</text>
<rect x="484" y="20" width="216" height="50" rx="6" class="k-gray"/>
<text x="592.0" y="49.5" text-anchor="middle" class="km-gray">{# ... #}</text>
<text x="592" y="96" text-anchor="middle" class="kh-gray">a comment, not output</text>
<rect x="484" y="116" width="216" height="50" rx="6" class="pnl"/>
<text x="592.0" y="145.5" text-anchor="middle" class="m">{# set by the LB team #}</text>
</g>
</svg>
<figcaption>Expressions, statements and comments.</figcaption>
</figure>

Filters transform values. A few of the many:

| Filter | Example | Result |
|---|---|---|
| `upper`, `lower`, `title` | `{{ 'web1' \| upper }}` | `WEB1` |
| `default` | `{{ port \| default(80) }}` | 80 if `port` is undefined |
| `join`, `split` | `{{ ['a','b'] \| join(', ') }}` | `a, b` |
| `length`, `first`, `last` | `{{ groups['web'] \| length }}` | `2` |
| `sort`, `unique`, `min`, `max` | `{{ [3,1,2] \| sort }}` | `[1, 2, 3]` |
| `int`, `float`, `round` | `{{ (7944 / 1024) \| round(1) }}` | `7.8` |
| `replace`, `regex_replace` | `{{ 'a-b' \| replace('-', '_') }}` | `a_b` |
| `to_nice_yaml`, `to_nice_json` | `{{ mydict \| to_nice_yaml }}` | data as text |
| `password_hash` | `{{ pw \| password_hash('sha512') }}` | a crypt hash (Chapter 11) |

Try them on control, where `localhost` needs no SSH:

```
ansible localhost -m debug -a 'msg="{{ \"web server\" | upper }} {{ [3,1,2] | sort }} {{ undefined_thing | default(\"fallback\") }}"'
```

## A login banner

Every server gets a message of the day saying what it is:

```
mkdir -p templates
cat > templates/motd.j2 <<'EOF'
==============================================================
 {{ inventory_hostname | upper }}: {{ company }} {{ role_description }}
 {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}, {{ ansible_facts['processor_vcpus'] }} CPUs, {{ (ansible_facts['memtotal_mb'] / 1024) | round(1) }} GB
{% if 'web' in group_names %}
 Web site: http://{{ inventory_hostname }}:{{ http_port }}/
{% endif %}
 Managed by Ansible: changes made by hand will be lost.
==============================================================
EOF
```

with a `role_description` in each group's variables (`a server` in `all.yml`, `web server` in `web.yml`, `database server` in `db.yml`). The `{% if %}` block keeps the *Web site* line only on web servers:

```
ssh web1 cat /etc/motd
```

```
==============================================================
 WEB1: Northwind web server
 Debian 13, 2 CPUs, 7.8 GB
 Web site: http://web1:80/
 Managed by Ansible: changes made by hand will be lost.
==============================================================
```

## The site from data

The web servers' settings move into `group_vars/web.yml`, including a list and a dictionary:

```yaml
role_description: web server
site_name: Northwind Coffee
http_port: 80
web_root: /var/www/html
site_aliases:
  - www.northwind.lab
  - northwind.lab
opening_hours:
  Monday to Friday: 7:00 to 19:00
  Saturday: 8:00 to 17:00
  Sunday: closed
```

The nginx configuration loops over the aliases; `ansible_managed` writes a standard warning that the file is managed by Ansible:

```
cat > templates/site.conf.j2 <<'EOF'
# {{ ansible_managed }}
server {
    listen {{ http_port }} default_server;
    server_name {{ inventory_hostname }}{% for alias in site_aliases %} {{ alias }}{% endfor %};
    root {{ web_root }};
    index index.html;
}
EOF
```

The home page loops over the opening hours, a dictionary, with `.items()`:

```
cat > templates/index.html.j2 <<'EOF'
<!DOCTYPE html>
<html><head><title>{{ site_name }}</title></head>
<body>
<h1>{{ site_name }}</h1>
<h2>Opening hours</h2>
<ul>
{% for day, hours in opening_hours.items() %}
  <li>{{ day }}: {{ hours }}</li>
{% endfor %}
</ul>
<p>Served by {{ inventory_hostname }} ({{ ansible_facts['eth0']['ipv4']['address'] }}).</p>
</body></html>
EOF
```

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A for loop over a dictionary. The variable opening_hours maps Monday to Friday to 7:00 to 19:00, Saturday to 8:00 to 17:00, and Sunday to closed. The loop for day, hours in opening_hours.items() produces one li line per entry in the rendered page." xmlns="http://www.w3.org/2000/svg">
<style>
.a7c{ font-family:var(--sans),sans-serif; }
.a7c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a7c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a7c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a7c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a7c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a7c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a7c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a7c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a7c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a7c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a7c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a7c .s{ fill:var(--text-dim); font-size:13px; }
.a7c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a7c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a7c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a7c .c{ fill:var(--text-faint); font-size:12.5px; }
.a7c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a7c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a7c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a7c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7c-a); }
.a7c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7c-a); marker-start:url(#a7c-s); }
.a7c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a7c-g); }
.a7c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a7c-a); }
.a7c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a7c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a7c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a7c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a7c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a7c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7c-tri); }
.a7c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7c-tri); }
.a7c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7c-open); }
.a7c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7c-open); }
.a7c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7c-dia); }
.a7c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7c-odia); }
.a7c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7c-dia); marker-end:url(#a7c-open); }
.a7c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7c-solid); }
.a7c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a7c-open); }
.a7c .fill{ fill:var(--text); stroke:none; }
.a7c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a7c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a7c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a7c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a7c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a7c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a7c .ks-blue{ fill:#185FA5; font-size:13px; }
.a7c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a7c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a7c .kb-blue, :root[data-theme="dark"] .a7c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a7c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a7c .kh-blue{ fill:#85B7EB; }
.a7c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a7c-blue); }
.a7c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7c-blue); }
.a7c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a7c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a7c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a7c .ks-amber{ fill:#854F0B; font-size:13px; }
.a7c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a7c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a7c .kb-amber, :root[data-theme="dark"] .a7c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a7c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a7c .kh-amber{ fill:#EF9F27; }
.a7c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a7c-amber); }
.a7c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7c-amber); }
.a7c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a7c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a7c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a7c .ks-purple{ fill:#534AB7; font-size:13px; }
.a7c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a7c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a7c .kb-purple, :root[data-theme="dark"] .a7c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a7c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a7c .kh-purple{ fill:#AFA9EC; }
.a7c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a7c-purple); }
.a7c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7c-purple); }
.a7c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a7c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a7c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a7c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a7c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a7c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a7c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a7c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a7c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a7c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a7c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a7c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a7c">
<rect x="20" y="20" width="300" height="110" rx="6" class="k-amber"/>
<text x="30" y="40" text-anchor="start" class="kh-amber">group_vars/web.yml</text>
<text x="32.0" y="62" text-anchor="start" class="m">opening_hours:</text>
<text x="47.2" y="80" text-anchor="start" class="m">Monday to Friday: 7:00 to 19:00</text>
<text x="47.2" y="98" text-anchor="start" class="m">Saturday: 8:00 to 17:00</text>
<text x="47.2" y="116" text-anchor="start" class="m">Sunday: closed</text>
<rect x="20" y="150" width="300" height="110" rx="6" class="k-purple"/>
<text x="30" y="170" text-anchor="start" class="kh-purple">index.html.j2</text>
<text x="32.0" y="192" text-anchor="start" class="m">{% for day, hours in</text>
<text x="77.6" y="210" text-anchor="start" class="m">opening_hours.items() %}</text>
<text x="47.2" y="228" text-anchor="start" class="m">&lt;li&gt;{{ day }}: {{ hours }}&lt;/li&gt;</text>
<text x="32.0" y="246" text-anchor="start" class="m">{% endfor %}</text>
<rect x="350" y="80" width="360" height="128" rx="6" class="k-blue"/>
<text x="360" y="100" text-anchor="start" class="kh-blue">rendered on web1</text>
<text x="362.0" y="122" text-anchor="start" class="m">&lt;ul&gt;</text>
<text x="377.2" y="140" text-anchor="start" class="m">&lt;li&gt;Monday to Friday: 7:00 to 19:00&lt;/li&gt;</text>
<text x="377.2" y="158" text-anchor="start" class="m">&lt;li&gt;Saturday: 8:00 to 17:00&lt;/li&gt;</text>
<text x="377.2" y="176" text-anchor="start" class="m">&lt;li&gt;Sunday: closed&lt;/li&gt;</text>
<text x="362.0" y="194" text-anchor="start" class="m">&lt;/ul&gt;</text>
<path d="M322 80 L346 130" class="ar-amber"/>
<path d="M322 200 L346 170" class="ar-purple"/>
</g>
</svg>
<figcaption>The data drives the repetition; the template stays the same.</figcaption>
</figure>

In `site.yml`, the two `copy` tasks become `template` tasks:

```yaml
    - name: Site configuration
      ansible.builtin.template:
        src: templates/site.conf.j2
        dest: /etc/nginx/sites-available/default
        mode: "0644"

    - name: Home page
      ansible.builtin.template:
        src: templates/index.html.j2
        dest: "{{ web_root }}/index.html"
        mode: "0644"
```

```
ansible-playbook site.yml
ssh web2 cat /etc/nginx/sites-available/default
curl -s http://web2
```

```
# Ansible managed
server {
    listen 80 default_server;
    server_name web2 www.northwind.lab northwind.lab;
    root /var/www/html;
    index index.html;
}
```

To add an alias or change the hours, you now edit **data**, not files, and run the playbook. The templates stay the same, and so does every other server's output.

## Whitespace

Template tags leave lines behind unless told otherwise. Ansible renders templates with Jinja2's **trim_blocks** on: the newline right after a `{% %}` tag is removed, so the `{% for %}` and `{% endfor %}` lines above leave nothing behind.

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Whitespace control. Without trim_blocks, each line holding only a for or endfor tag would leave an empty line in the output. Ansible enables trim_blocks, which removes the newline after a block tag, so the output has only the li lines. A minus sign inside a tag strips more whitespace." xmlns="http://www.w3.org/2000/svg">
<style>
.a7d{ font-family:var(--sans),sans-serif; }
.a7d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a7d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a7d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a7d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a7d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a7d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a7d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a7d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a7d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a7d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a7d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a7d .s{ fill:var(--text-dim); font-size:13px; }
.a7d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a7d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a7d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a7d .c{ fill:var(--text-faint); font-size:12.5px; }
.a7d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a7d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a7d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a7d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7d-a); }
.a7d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7d-a); marker-start:url(#a7d-s); }
.a7d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a7d-g); }
.a7d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a7d-a); }
.a7d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a7d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a7d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a7d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a7d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a7d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7d-tri); }
.a7d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7d-tri); }
.a7d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7d-open); }
.a7d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7d-open); }
.a7d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7d-dia); }
.a7d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7d-odia); }
.a7d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7d-dia); marker-end:url(#a7d-open); }
.a7d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7d-solid); }
.a7d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a7d-open); }
.a7d .fill{ fill:var(--text); stroke:none; }
.a7d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a7d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a7d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a7d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a7d .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a7d .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a7d .ks-teal{ fill:#0F6E56; font-size:13px; }
.a7d .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a7d .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7d .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a7d .kb-teal, :root[data-theme="dark"] .a7d .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a7d .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a7d .kh-teal{ fill:#5DCAA5; }
.a7d .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a7d-teal); }
.a7d .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7d-teal); }
.a7d .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a7d .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a7d .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a7d .ks-purple{ fill:#534AB7; font-size:13px; }
.a7d .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a7d .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7d .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a7d .kb-purple, :root[data-theme="dark"] .a7d .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a7d .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a7d .kh-purple{ fill:#AFA9EC; }
.a7d .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a7d-purple); }
.a7d .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7d-purple); }
.a7d .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a7d .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a7d .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a7d .ks-red{ fill:#A32D2D; font-size:13px; }
.a7d .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a7d .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7d .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a7d .kb-red, :root[data-theme="dark"] .a7d .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a7d .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a7d .kh-red{ fill:#F09595; }
.a7d .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a7d-red); }
.a7d .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7d-red); }
.a7d .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a7d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a7d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a7d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a7d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a7d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a7d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a7d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a7d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a7d-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a7d-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a7d-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a7d">
<rect x="20" y="20" width="210" height="128" rx="6" class="k-purple"/>
<text x="30" y="40" text-anchor="start" class="kh-purple">template</text>
<text x="32.0" y="62" text-anchor="start" class="m">&lt;ul&gt;</text>
<text x="32.0" y="80" text-anchor="start" class="m">{% for d in days %}</text>
<text x="32.0" y="98" text-anchor="start" class="m">&lt;li&gt;{{ d }}&lt;/li&gt;</text>
<text x="32.0" y="116" text-anchor="start" class="m">{% endfor %}</text>
<text x="32.0" y="134" text-anchor="start" class="m">&lt;/ul&gt;</text>
<rect x="260" y="20" width="200" height="146" rx="6" class="k-red"/>
<text x="270" y="40" text-anchor="start" class="kh-red">plain Jinja2</text>
<text x="272.0" y="62" text-anchor="start" class="m">&lt;ul&gt;</text>
<text x="272.0" y="80" text-anchor="start" class="c">(empty line)</text>
<text x="272.0" y="98" text-anchor="start" class="m">&lt;li&gt;Mon&lt;/li&gt;</text>
<text x="272.0" y="116" text-anchor="start" class="c">(empty line)</text>
<text x="272.0" y="134" text-anchor="start" class="m">&lt;li&gt;Tue&lt;/li&gt;</text>
<text x="272.0" y="152" text-anchor="start" class="m">...</text>
<rect x="490" y="20" width="210" height="110" rx="6" class="k-teal"/>
<text x="500" y="40" text-anchor="start" class="kh-teal">Ansible: trim_blocks</text>
<text x="502.0" y="62" text-anchor="start" class="m">&lt;ul&gt;</text>
<text x="502.0" y="80" text-anchor="start" class="m">&lt;li&gt;Mon&lt;/li&gt;</text>
<text x="502.0" y="98" text-anchor="start" class="m">&lt;li&gt;Tue&lt;/li&gt;</text>
<text x="502.0" y="116" text-anchor="start" class="m">&lt;/ul&gt;</text>
<text x="360" y="200" text-anchor="middle" class="c">{%- strips before the tag, -%} strips after it</text>
</g>
</svg>
<figcaption>trim_blocks removes the line break after a block tag.</figcaption>
</figure>

For finer control, a minus sign strips whitespace on that side of a tag: `{%- ... %}` before it, `{% ... -%}` after it. Whitespace hardly matters in HTML, but it matters in YAML, Python and some configuration files. When in doubt, render the template and look at the result with `--check --diff`.

## Files about the whole fleet

A template can loop over the inventory. This one writes, on **every** server, a list of every server:

```
cat > templates/lab-hosts.j2 <<'EOF'
# {{ ansible_managed }}
# Every Northwind server, from the Ansible inventory
{% for host in groups['all'] | sort %}
{{ hostvars[host]['ansible_facts']['eth0']['ipv4']['address'] }}  {{ host }}.northwind.lab  {{ host }}
{% endfor %}
EOF
```

```
# Ansible managed
# Every Northwind server, from the Ansible inventory
10.0.0.21  db1.northwind.lab  db1
10.0.0.11  web1.northwind.lab  web1
10.0.0.12  web2.northwind.lab  web2
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Building a file about the whole fleet. The template loops over groups all and reads each host&#x27;s eth0 address from hostvars. Every host, web1, web2 and db1, receives the same list of all three hosts and their addresses. Adding a host to the inventory adds a line everywhere." xmlns="http://www.w3.org/2000/svg">
<style>
.a7e{ font-family:var(--sans),sans-serif; }
.a7e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a7e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a7e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a7e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a7e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a7e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a7e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a7e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a7e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a7e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a7e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a7e .s{ fill:var(--text-dim); font-size:13px; }
.a7e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a7e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a7e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a7e .c{ fill:var(--text-faint); font-size:12.5px; }
.a7e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a7e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a7e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a7e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7e-a); }
.a7e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7e-a); marker-start:url(#a7e-s); }
.a7e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a7e-g); }
.a7e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a7e-a); }
.a7e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a7e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a7e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a7e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a7e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a7e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7e-tri); }
.a7e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7e-tri); }
.a7e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7e-open); }
.a7e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7e-open); }
.a7e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7e-dia); }
.a7e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7e-odia); }
.a7e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7e-dia); marker-end:url(#a7e-open); }
.a7e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7e-solid); }
.a7e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a7e-open); }
.a7e .fill{ fill:var(--text); stroke:none; }
.a7e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a7e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a7e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a7e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a7e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a7e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a7e .ks-blue{ fill:#185FA5; font-size:13px; }
.a7e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a7e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a7e .kb-blue, :root[data-theme="dark"] .a7e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a7e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a7e .kh-blue{ fill:#85B7EB; }
.a7e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a7e-blue); }
.a7e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7e-blue); }
.a7e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a7e .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a7e .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a7e .ks-amber{ fill:#854F0B; font-size:13px; }
.a7e .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a7e .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7e .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a7e .kb-amber, :root[data-theme="dark"] .a7e .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a7e .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a7e .kh-amber{ fill:#EF9F27; }
.a7e .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a7e-amber); }
.a7e .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7e-amber); }
.a7e .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a7e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a7e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a7e .ks-purple{ fill:#534AB7; font-size:13px; }
.a7e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a7e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a7e .kb-purple, :root[data-theme="dark"] .a7e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a7e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a7e .kh-purple{ fill:#AFA9EC; }
.a7e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a7e-purple); }
.a7e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7e-purple); }
.a7e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a7e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a7e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a7e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a7e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a7e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a7e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a7e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a7e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a7e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a7e-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a7e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a7e">
<rect x="10" y="20" width="380" height="110" rx="6" class="k-purple"/>
<text x="20" y="40" text-anchor="start" class="kh-purple">lab-hosts.j2</text>
<text x="22.0" y="62" text-anchor="start" class="m">{% for host in groups['all'] | sort %}</text>
<text x="22.0" y="80" text-anchor="start" class="m">{{ hostvars[host]['ansible_facts']</text>
<text x="60.0" y="98" text-anchor="start" class="m">['eth0']['ipv4']['address'] }}  {{ host }}</text>
<text x="22.0" y="116" text-anchor="start" class="m">{% endfor %}</text>
<rect x="10" y="170" width="380" height="60" rx="7" class="k-amber"/>
<text x="200.0" y="195.5" text-anchor="middle" class="kb-amber">needs every host's facts in this run</text>
<text x="200.0" y="213.5" text-anchor="middle" class="ks-amber">--limit web1 alone fails: no facts for db1</text>
<rect x="430" y="16" width="280" height="68" rx="8" class="k-blue"/>
<text x="444" y="34" text-anchor="start" class="kb-blue">web1: /etc/lab-hosts</text>
<text x="444" y="54" text-anchor="start" class="km-blue">10.0.0.21 db1   10.0.0.11 web1</text>
<text x="444" y="72" text-anchor="start" class="km-blue">10.0.0.12 web2</text>
<path d="M392 80 L426 50" class="ar-purple"/>
<rect x="430" y="94" width="280" height="68" rx="8" class="k-blue"/>
<text x="444" y="112" text-anchor="start" class="kb-blue">web2: /etc/lab-hosts</text>
<text x="444" y="132" text-anchor="start" class="km-blue">10.0.0.21 db1   10.0.0.11 web1</text>
<text x="444" y="150" text-anchor="start" class="km-blue">10.0.0.12 web2</text>
<path d="M392 80 L426 128" class="ar-purple"/>
<rect x="430" y="172" width="280" height="68" rx="8" class="k-blue"/>
<text x="444" y="190" text-anchor="start" class="kb-blue">db1: /etc/lab-hosts</text>
<text x="444" y="210" text-anchor="start" class="km-blue">10.0.0.21 db1   10.0.0.11 web1</text>
<text x="444" y="228" text-anchor="start" class="km-blue">10.0.0.12 web2</text>
<path d="M392 80 L426 206" class="ar-purple"/>
</g>
</svg>
<figcaption>Every host gets the same picture of the fleet.</figcaption>
</figure>

Add a server to the inventory, run the playbook, and every server knows it. The file goes to `/etc/lab-hosts` rather than `/etc/hosts`, which DockerLab rewrites at every start.

There is a catch. The template reads **every** host's facts through `hostvars`, so all hosts must have been gathered in this run. With `--limit web1`, db1's facts are missing and the task fails with *object of type 'dict' has no attribute 'eth0'*. Chapter 14's firewall role shows the fix: gather the missing facts with `delegate_to` and `delegate_facts`. Fact caching (`fact_caching = jsonfile` in `ansible.cfg`) is another.

## A report on control

Templates can also write files on the control node itself. This playbook gathers every host's facts, then writes one Markdown report:

```
cat > templates/report.md.j2 <<'EOF'
# Northwind fleet report, {{ now(fmt='%Y-%m-%d %H:%M') }}

| Host | Groups | System | CPUs | Memory | Address |
|---|---|---|---|---|---|
{% for host in groups['all'] | sort %}
{% set f = hostvars[host]['ansible_facts'] %}
| {{ host }} | {{ hostvars[host]['group_names'] | join(', ') }} | {{ f['distribution'] }} {{ f['distribution_version'] }} | {{ f['processor_vcpus'] }} | {{ f['memtotal_mb'] }} MB | {{ f['eth0']['ipv4']['address'] }} |
{% endfor %}

{{ groups['all'] | length }} hosts.
EOF
cat > report.yml <<'EOF'
---
- name: Report on the fleet
  hosts: all
  tasks:
    - name: Write the report on control
      ansible.builtin.template:
        src: templates/report.md.j2
        dest: "{{ playbook_dir }}/fleet-report.md"
        mode: "0644"
      delegate_to: localhost
      run_once: true
EOF
ansible-playbook report.yml
cat fleet-report.md
```

```
# Northwind fleet report, 2026-11-18 04:46

| Host | Groups | System | CPUs | Memory | Address |
|---|---|---|---|---|---|
| db1 | db, linux, production | Debian 13 | 2 | 7944 MB | 10.0.0.21 |
| web1 | linux, production, web | Debian 13 | 2 | 7944 MB | 10.0.0.11 |
| web2 | linux, staging, web | Debian 13 | 2 | 7944 MB | 10.0.0.12 |

3 hosts.
```

<figure class="diagram">
<svg viewBox="0 0 720 244" style="width:100%;height:auto;max-width:720px" role="img" aria-label="delegate_to and run_once. The play targets web1, web2 and db1 and gathers their facts. The report task is delegated to localhost, the control node, and runs once instead of three times, writing fleet-report.md on control from all hosts&#x27; facts." xmlns="http://www.w3.org/2000/svg">
<style>
.a7f{ font-family:var(--sans),sans-serif; }
.a7f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a7f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a7f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a7f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a7f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a7f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a7f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a7f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a7f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a7f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a7f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a7f .s{ fill:var(--text-dim); font-size:13px; }
.a7f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a7f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a7f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a7f .c{ fill:var(--text-faint); font-size:12.5px; }
.a7f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a7f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a7f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a7f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7f-a); }
.a7f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7f-a); marker-start:url(#a7f-s); }
.a7f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a7f-g); }
.a7f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a7f-a); }
.a7f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a7f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a7f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a7f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a7f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a7f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7f-tri); }
.a7f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7f-tri); }
.a7f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7f-open); }
.a7f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7f-open); }
.a7f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7f-dia); }
.a7f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7f-odia); }
.a7f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7f-dia); marker-end:url(#a7f-open); }
.a7f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7f-solid); }
.a7f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a7f-open); }
.a7f .fill{ fill:var(--text); stroke:none; }
.a7f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a7f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a7f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a7f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a7f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a7f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a7f .ks-blue{ fill:#185FA5; font-size:13px; }
.a7f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a7f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a7f .kb-blue, :root[data-theme="dark"] .a7f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a7f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a7f .kh-blue{ fill:#85B7EB; }
.a7f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a7f-blue); }
.a7f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7f-blue); }
.a7f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a7f .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a7f .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a7f .ks-purple{ fill:#534AB7; font-size:13px; }
.a7f .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a7f .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7f .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a7f .kb-purple, :root[data-theme="dark"] .a7f .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a7f .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a7f .kh-purple{ fill:#AFA9EC; }
.a7f .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a7f-purple); }
.a7f .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7f-purple); }
.a7f .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a7f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a7f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a7f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a7f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a7f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a7f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a7f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a7f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a7f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a7f-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a7f">
<rect x="20" y="16" width="150" height="54" rx="7" class="k-blue"/>
<line x1="30" y1="25" x2="46" y2="25" class="ln-blue"/>
<line x1="30" y1="30" x2="46" y2="30" class="ln-blue"/>
<text x="103.0" y="38" text-anchor="middle" class="kb-blue">web1</text>
<text x="95.0" y="57" text-anchor="middle" class="ks-blue">facts gathered</text>
<rect x="20" y="82" width="150" height="54" rx="7" class="k-blue"/>
<line x1="30" y1="91" x2="46" y2="91" class="ln-blue"/>
<line x1="30" y1="96" x2="46" y2="96" class="ln-blue"/>
<text x="103.0" y="104" text-anchor="middle" class="kb-blue">web2</text>
<text x="95.0" y="123" text-anchor="middle" class="ks-blue">facts gathered</text>
<rect x="20" y="148" width="150" height="54" rx="7" class="k-blue"/>
<line x1="30" y1="157" x2="46" y2="157" class="ln-blue"/>
<line x1="30" y1="162" x2="46" y2="162" class="ln-blue"/>
<text x="103.0" y="170" text-anchor="middle" class="kb-blue">db1</text>
<text x="95.0" y="189" text-anchor="middle" class="ks-blue">facts gathered</text>
<rect x="250" y="70" width="190" height="70" rx="7" class="k-purple"/>
<text x="345.0" y="90.5" text-anchor="middle" class="kb-purple">template task</text>
<text x="345.0" y="109.5" text-anchor="middle" class="km-purple">delegate_to: localhost</text>
<text x="345.0" y="128.5" text-anchor="middle" class="km-purple">run_once: true</text>
<path d="M172 43 L246 105" class="ag"/>
<path d="M172 109 L246 105" class="ag"/>
<path d="M172 175 L246 105" class="ag"/>
<rect x="500" y="60" width="200" height="54" rx="7" class="dark"/>
<text x="600.0" y="83" text-anchor="middle" class="on">control</text>
<text x="600.0" y="101" text-anchor="middle" class="on" style="fill:var(--bg);font-weight:400">writes fleet-report.md</text>
<path d="M442 105 L496 90" class="ar-purple"/>
<text x="360" y="232" text-anchor="middle" class="c">runs on control, once, with the variables of the first host in the play</text>
</g>
</svg>
<figcaption>Delegation moves a task to another machine; run_once runs it once.</figcaption>
</figure>

`delegate_to: localhost` runs the task on control instead of on the host, but **with the host's variables**. `run_once: true` runs it once for the whole play instead of once per host. `{% set %}` gives a long expression a short name inside the template.

## Validate before replacing

A broken configuration file can take a service down, or lock everyone out. `template` and `copy` can check the new file **before** it replaces the old one, with `validate`; `%s` is the new file's temporary name:

```yaml
    - name: A sudo rule for the deploy group, only if visudo accepts it
      ansible.builtin.copy:
        content: "%deploy ALL=(root) NOPASSWD: /usr/sbin/service nginx *\n"
        dest: /etc/sudoers.d/deploy
        mode: "0440"
        validate: /usr/sbin/visudo -cf %s
```

Break it on purpose, by removing the colon after `NOPASSWD`, and run the playbook:

```
fatal: [web1]: FAILED! => {"changed": false, "exit_status": 1, "msg": "failed to validate",
  "stderr": ".../.source:1:54: syntax error\n%deploy ALL=(root) NOPASSWD /usr/sbin/service nginx *\n ..."}
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="How validate protects a file. The module writes the new content to a temporary file on the host and runs the validate command on it, such as visudo -cf. If validation passes, the temporary file replaces the real one and the task reports changed. If it fails, the real file is untouched and the task fails with the validator&#x27;s message." xmlns="http://www.w3.org/2000/svg">
<style>
.a7g{ font-family:var(--sans),sans-serif; }
.a7g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a7g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a7g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a7g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a7g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a7g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a7g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a7g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a7g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a7g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a7g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a7g .s{ fill:var(--text-dim); font-size:13px; }
.a7g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a7g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a7g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a7g .c{ fill:var(--text-faint); font-size:12.5px; }
.a7g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a7g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a7g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a7g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7g-a); }
.a7g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a7g-a); marker-start:url(#a7g-s); }
.a7g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a7g-g); }
.a7g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-a); }
.a7g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a7g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a7g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a7g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a7g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a7g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7g-tri); }
.a7g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-tri); }
.a7g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7g-open); }
.a7g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-open); }
.a7g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7g-dia); }
.a7g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7g-odia); }
.a7g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a7g-dia); marker-end:url(#a7g-open); }
.a7g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a7g-solid); }
.a7g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a7g-open); }
.a7g .fill{ fill:var(--text); stroke:none; }
.a7g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a7g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a7g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a7g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a7g .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a7g .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a7g .ks-amber{ fill:#854F0B; font-size:13px; }
.a7g .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a7g .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7g .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a7g .kb-amber, :root[data-theme="dark"] .a7g .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a7g .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a7g .kh-amber{ fill:#EF9F27; }
.a7g .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a7g-amber); }
.a7g .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-amber); }
.a7g .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a7g .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a7g .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a7g .ks-purple{ fill:#534AB7; font-size:13px; }
.a7g .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a7g .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7g .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a7g .kb-purple, :root[data-theme="dark"] .a7g .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a7g .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a7g .kh-purple{ fill:#AFA9EC; }
.a7g .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a7g-purple); }
.a7g .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-purple); }
.a7g .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a7g .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a7g .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a7g .ks-green{ fill:#3B6D11; font-size:13px; }
.a7g .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a7g .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7g .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a7g .kb-green, :root[data-theme="dark"] .a7g .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a7g .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a7g .kh-green{ fill:#97C459; }
.a7g .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a7g-green); }
.a7g .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-green); }
.a7g .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a7g .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a7g .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a7g .ks-red{ fill:#A32D2D; font-size:13px; }
.a7g .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a7g .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7g .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a7g .kb-red, :root[data-theme="dark"] .a7g .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a7g .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a7g .kh-red{ fill:#F09595; }
.a7g .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a7g-red); }
.a7g .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-red); }
.a7g .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a7g .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a7g .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a7g .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a7g .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a7g .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a7g .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a7g .kb-gray, :root[data-theme="dark"] .a7g .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a7g .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a7g .kh-gray{ fill:#B4B2A9; }
.a7g .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a7g-gray); }
.a7g .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a7g-gray); }
.a7g .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a7g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a7g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a7g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a7g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a7g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a7g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a7g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a7g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a7g-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a7g-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a7g-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a7g-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a7g-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a7g">
<rect x="20" y="90" width="150" height="60" rx="7" class="k-purple"/>
<text x="95.0" y="115.5" text-anchor="middle" class="kb-purple">new content</text>
<text x="95.0" y="133.5" text-anchor="middle" class="ks-purple">from copy/template</text>
<rect x="200" y="90" width="150" height="60" rx="7" class="k-gray"/>
<text x="275.0" y="115.5" text-anchor="middle" class="kb-gray">temporary file</text>
<text x="275.0" y="133.5" text-anchor="middle" class="ks-gray">on the host</text>
<path d="M172 120 L196 120" class="ar"/>
<path d="M420 84 L456 120 L420 156 L384 120 z" class="k-amber"/>
<text x="420" y="124.5" text-anchor="middle" class="kb-amber">valid?</text>
<path d="M352 120 L382 120" class="ar"/>
<text x="420" y="190" text-anchor="middle" class="m">visudo -cf %s</text>
<rect x="520" y="20" width="180" height="80" rx="7" class="k-green"/>
<text x="610.0" y="55.5" text-anchor="middle" class="kb-green">replace the real file</text>
<text x="610.0" y="73.5" text-anchor="middle" class="ks-green">changed</text>
<rect x="520" y="140" width="180" height="90" rx="7" class="k-red"/>
<text x="610.0" y="171.5" text-anchor="middle" class="kb-red">leave the real file</text>
<text x="610.0" y="189.5" text-anchor="middle" class="ks-red">task failed:</text>
<text x="610.0" y="207.5" text-anchor="middle" class="km-red">failed to validate</text>
<path d="M440 92 L516 60" class="ar-green"/>
<text x="470" y="66" text-anchor="middle" class="kh-green">yes</text>
<path d="M440 148 L516 184" class="ar-red"/>
<text x="492" y="160" text-anchor="middle" class="kh-red">no</text>
</g>
</svg>
<figcaption>A broken file never reaches its real place.</figcaption>
</figure>

The broken rule never reached `/etc/sudoers.d`, where it would have broken `sudo` for everyone, including Ansible. Use `validate` for every file in which a mistake could lock you out: sudoers rules (`visudo -cf %s`), SSH settings (`sshd -t -f %s`, Chapter 14), FRR's routing configuration (`vtysh --dryrun --inputfile %s`, Chapter 13). nginx's site files can't be checked alone, so Chapter 9 runs `nginx -t` in a handler instead.

## Hands on

1. Give web2 a different `server_name` without changing the template.
2. Add `{{ opening_hours | to_nice_yaml }}` to a template and look at the result.
3. Make `report.md.j2` mark hosts with less than 1 GB of memory in **bold**.
4. Write a template for `/etc/ssh/sshd_config.d/northwind.conf` setting `MaxAuthTries` from a variable, validated with `/usr/sbin/sshd -t -f %s`.

## Summary

- `template` renders a Jinja2 file **on control** with each host's variables and facts, and writes the result to the host, idempotently.
- `{{ }}` prints expressions, `{% %}` holds logic (`if`, `for`, `set`), `{# #}` holds comments; filters transform values.
- Keep **data** in variables and **form** in templates: change the data, not the files.
- Ansible's templates use `trim_blocks`; `{%-` and `-%}` strip more whitespace.
- Templates can loop over `groups` and read `hostvars` to build files about the whole fleet, as long as every host's facts were gathered.
- `delegate_to: localhost` with `run_once` writes a file on control; `validate` checks a file before it replaces the old one.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-7-templates).

**1. Where does the template module read its .j2 file?**

- A. On the control node
- B. On the managed host
- C. From a web server
- D. From the inventory

**2. What is the difference between {{ }} and {% %}?**

- A. None
- B. {{ }} is for comments
- C. {{ }} prints an expression; {% %} holds logic such as if, for and set
- D. {% %} prints values

**3. How do you loop over a dictionary such as opening_hours in a template?**

- A. {% for d in opening_hours %}
- B. {{ opening_hours | loop }}
- C. {% each opening_hours %}
- D. {% for day, hours in opening_hours.items() %}

**4. What does Jinja2's trim_blocks, enabled by Ansible, do?**

- A. Removes comments
- B. Removes the newline right after a {% %} block tag
- C. Trims spaces from variables
- D. Shortens long lines

**5. Why does a template reading hostvars[host]['ansible_facts'] for all hosts fail with --limit web1?**

- A. --limit disables templates
- B. hostvars is only available in roles
- C. The other hosts' facts were not gathered in this run
- D. Templates can't read facts

**6. What do delegate_to: localhost and run_once: true do together?**

- A. Run the task once, on control, with the first host's variables
- B. Run the task on every host twice
- C. Skip the task
- D. Run on all hosts in parallel

**7. What does validate: /usr/sbin/visudo -cf %s do on a copy task?**

- A. Checks the new file before it replaces the old one; on failure the real file is untouched
- B. Formats the file
- C. Encrypts the file
- D. Restarts sudo

**8. Where should the opening hours of the site live?**

- A. Hard-coded in the template
- B. In the inventory file
- C. As data in group_vars, with the template only describing the form
- D. In ansible.cfg
