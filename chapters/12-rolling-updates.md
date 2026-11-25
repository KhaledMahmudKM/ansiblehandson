# Chapter 12: Rolling Updates

The shop is live. *Northwind Orders* now runs on three web servers behind a **load balancer**, and customers place orders at all hours. A new version must go out without the site going down, and a bad version must never reach every server. The answer is a **rolling update**: one server at a time, taken out of the load balancer, updated, checked, and put back. If a server fails its check, the rollout stops before the failure spreads. This chapter builds the load balancer from the inventory and writes the rollout playbook.

> **Try it:** DockerLab, *Ansible lab 11, Rolling updates*. Topology: control `10.0.0.10`, lb `10.0.0.20`, web1 `10.0.0.11`, web2 `10.0.0.12`, web3 `10.0.0.13`. Start with `setup-keys lb web1 web2 web3`. The finished project is in `examples/ch12`.

## The new topology

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Ansible lab 11&#x27;s topology. Customers send requests to the load balancer lb at 10.0.0.20, which shares them among web1, web2 and web3 at 10.0.0.11 to .13. The control node at 10.0.0.10 manages all four over SSH." xmlns="http://www.w3.org/2000/svg">
<style>
.a12a{ font-family:var(--sans),sans-serif; }
.a12a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a12a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a12a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a12a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a12a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a12a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a12a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a12a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a12a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a12a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a12a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a12a .s{ fill:var(--text-dim); font-size:13px; }
.a12a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a12a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a12a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a12a .c{ fill:var(--text-faint); font-size:12.5px; }
.a12a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a12a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a12a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a12a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12a-a); }
.a12a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12a-a); marker-start:url(#a12a-s); }
.a12a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a12a-g); }
.a12a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a12a-a); }
.a12a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a12a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a12a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a12a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a12a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a12a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12a-tri); }
.a12a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12a-tri); }
.a12a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12a-open); }
.a12a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12a-open); }
.a12a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12a-dia); }
.a12a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12a-odia); }
.a12a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12a-dia); marker-end:url(#a12a-open); }
.a12a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12a-solid); }
.a12a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a12a-open); }
.a12a .fill{ fill:var(--text); stroke:none; }
.a12a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a12a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a12a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a12a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a12a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a12a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a12a .ks-blue{ fill:#185FA5; font-size:13px; }
.a12a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a12a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a12a .kb-blue, :root[data-theme="dark"] .a12a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a12a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a12a .kh-blue{ fill:#85B7EB; }
.a12a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a12a-blue); }
.a12a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12a-blue); }
.a12a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a12a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a12a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a12a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a12a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a12a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a12a .kb-teal, :root[data-theme="dark"] .a12a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a12a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a12a .kh-teal{ fill:#5DCAA5; }
.a12a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a12a-teal); }
.a12a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12a-teal); }
.a12a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a12a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a12a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a12a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a12a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a12a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a12a .kb-gray, :root[data-theme="dark"] .a12a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a12a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a12a .kh-gray{ fill:#B4B2A9; }
.a12a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a12a-gray); }
.a12a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12a-gray); }
.a12a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a12a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a12a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a12a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a12a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a12a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a12a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a12a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a12a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a12a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a12a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a12a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a12a">
<rect x="20" y="20" width="150" height="56" rx="7" class="k-gray"/>
<text x="95.0" y="43.5" text-anchor="middle" class="kb-gray">customers</text>
<text x="95.0" y="61.5" text-anchor="middle" class="km-gray">http://lb/</text>
<rect x="285" y="20" width="150" height="54" rx="7" class="k-teal"/>
<line x1="295" y1="29" x2="311" y2="29" class="ln-teal"/>
<line x1="295" y1="34" x2="311" y2="34" class="ln-teal"/>
<text x="368.0" y="42" text-anchor="middle" class="kb-teal">lb</text>
<text x="360.0" y="61" text-anchor="middle" class="ks-teal">10.0.0.20</text>
<path d="M172 48 L280 48" class="ar-teal"/>
<rect x="110" y="170" width="150" height="54" rx="7" class="k-blue"/>
<line x1="120" y1="179" x2="136" y2="179" class="ln-blue"/>
<line x1="120" y1="184" x2="136" y2="184" class="ln-blue"/>
<text x="193.0" y="192" text-anchor="middle" class="kb-blue">web1</text>
<text x="185.0" y="211" text-anchor="middle" class="ks-blue">10.0.0.11</text>
<path d="M360 76 L185 166" class="ar-teal"/>
<rect x="300" y="170" width="150" height="54" rx="7" class="k-blue"/>
<line x1="310" y1="179" x2="326" y2="179" class="ln-blue"/>
<line x1="310" y1="184" x2="326" y2="184" class="ln-blue"/>
<text x="383.0" y="192" text-anchor="middle" class="kb-blue">web2</text>
<text x="375.0" y="211" text-anchor="middle" class="ks-blue">10.0.0.12</text>
<path d="M360 76 L375 166" class="ar-teal"/>
<rect x="490" y="170" width="150" height="54" rx="7" class="k-blue"/>
<line x1="500" y1="179" x2="516" y2="179" class="ln-blue"/>
<line x1="500" y1="184" x2="516" y2="184" class="ln-blue"/>
<text x="573.0" y="192" text-anchor="middle" class="kb-blue">web3</text>
<text x="565.0" y="211" text-anchor="middle" class="ks-blue">10.0.0.13</text>
<path d="M360 76 L565 166" class="ar-teal"/>
<rect x="560" y="20" width="140" height="54" rx="7" class="dark"/>
<text x="630.0" y="43" text-anchor="middle" class="on">control</text>
<text x="630.0" y="61" text-anchor="middle" class="on" style="fill:var(--bg);font-weight:400">10.0.0.10</text>
<path d="M630 76 C 640 120, 640 140, 640 160" class="ad"/>
<text x="360" y="262" text-anchor="middle" class="c">round robin: each request to the next backend in turn</text>
<text x="630" y="112" text-anchor="start" class="a">SSH</text>
</g>
</svg>
<figcaption>Three web servers behind one load balancer.</figcaption>
</figure>

The inventory changes; the roles from Chapters 10 and 11 stay:

```
[loadbalancers]
lb

[web]
web1
web2
web3
```

`group_vars/web/vars.yml` now names the application and its version:

```yaml
common_role_description: web server
webserver_site_name: Northwind Orders
webserver_app_version: "1.0"
...
```

The webserver role gains two small things: the home page shows `webserver_app_version`, and the site answers a **health check** at `/health`, a URL that a load balancer or a test can ask "are you working?":

```
    location = /health { return 200 "OK {{ inventory_hostname }}\n"; }
```

(The group is called *loadbalancers*, not *lb*: a group and a host with the same name confuse Ansible, which warns about it.)

## A load balancer role, built from the inventory

nginx can act as a load balancer: an `upstream` block lists the **backends**, and `proxy_pass` sends each request to one of them in turn. The **loadbalancer** role writes the list from the inventory, as **one small file per backend**, so that a single server can later be taken out by changing only its own file:

```yaml
---
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    update_cache: true
    cache_valid_time: 3600

- name: Folder for the backends
  ansible.builtin.file:
    path: /etc/nginx/backends
    state: directory
    mode: "0755"

- name: One file per web server in the inventory (created once, then left to the rollout)
  ansible.builtin.copy:
    dest: /etc/nginx/backends/{{ item }}.conf
    content: "server {{ item }}:{{ loadbalancer_backend_port }};\n"
    force: false
    mode: "0644"
  loop: "{{ groups[loadbalancer_group] }}"
  notify: Reload nginx

- name: The balancer's site
  ansible.builtin.template:
    src: lb.conf.j2
    dest: /etc/nginx/sites-available/default
    mode: "0644"
  notify: Reload nginx

- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started
```

```
# roles/loadbalancer/templates/lb.conf.j2
# {{ ansible_managed }}
upstream app {
    include /etc/nginx/backends/*.conf;
}
server {
    listen 80 default_server;
    location / {
        proxy_pass http://app;
        add_header X-Served-By $upstream_addr always;
    }
}
```

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The load balancer&#x27;s configuration. lb.conf has an upstream block app that includes every file in /etc/nginx/backends. There is one file per web server: web1.conf with server web1:80, web2.conf with server web2:80 down (drained during a rollout), web3.conf with server web3:80. Requests go only to backends not marked down." xmlns="http://www.w3.org/2000/svg">
<style>
.a12b{ font-family:var(--sans),sans-serif; }
.a12b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a12b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a12b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a12b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a12b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a12b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a12b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a12b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a12b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a12b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a12b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a12b .s{ fill:var(--text-dim); font-size:13px; }
.a12b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a12b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a12b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a12b .c{ fill:var(--text-faint); font-size:12.5px; }
.a12b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a12b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a12b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a12b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12b-a); }
.a12b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12b-a); marker-start:url(#a12b-s); }
.a12b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a12b-g); }
.a12b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a12b-a); }
.a12b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a12b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a12b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a12b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a12b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a12b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12b-tri); }
.a12b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12b-tri); }
.a12b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12b-open); }
.a12b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12b-open); }
.a12b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12b-dia); }
.a12b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12b-odia); }
.a12b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12b-dia); marker-end:url(#a12b-open); }
.a12b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12b-solid); }
.a12b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a12b-open); }
.a12b .fill{ fill:var(--text); stroke:none; }
.a12b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a12b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a12b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a12b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a12b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a12b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a12b .ks-blue{ fill:#185FA5; font-size:13px; }
.a12b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a12b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a12b .kb-blue, :root[data-theme="dark"] .a12b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a12b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a12b .kh-blue{ fill:#85B7EB; }
.a12b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a12b-blue); }
.a12b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12b-blue); }
.a12b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a12b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a12b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a12b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a12b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a12b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a12b .kb-teal, :root[data-theme="dark"] .a12b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a12b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a12b .kh-teal{ fill:#5DCAA5; }
.a12b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a12b-teal); }
.a12b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12b-teal); }
.a12b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a12b .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a12b .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a12b .ks-red{ fill:#A32D2D; font-size:13px; }
.a12b .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a12b .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12b .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a12b .kb-red, :root[data-theme="dark"] .a12b .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a12b .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a12b .kh-red{ fill:#F09595; }
.a12b .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a12b-red); }
.a12b .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12b-red); }
.a12b .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a12b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a12b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a12b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a12b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a12b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a12b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a12b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a12b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a12b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a12b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a12b-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a12b">
<rect x="10" y="20" width="350" height="171" rx="6" class="k-teal"/>
<text x="20" y="40" text-anchor="start" class="kh-teal">sites-available/default</text>
<text x="22.0" y="62" text-anchor="start" class="m">upstream app {</text>
<text x="52.4" y="81" text-anchor="start" class="m">include /etc/nginx/backends/*.conf;</text>
<text x="22.0" y="100" text-anchor="start" class="m">}</text>
<text x="22.0" y="119" text-anchor="start" class="m">server {</text>
<text x="52.4" y="138" text-anchor="start" class="m">listen 80 default_server;</text>
<text x="52.4" y="157" text-anchor="start" class="m">location / { proxy_pass http://app; }</text>
<text x="22.0" y="176" text-anchor="start" class="m">}</text>
<rect x="400" y="20" width="310" height="60" rx="7" class="k-blue"/>
<text x="555.0" y="45.5" text-anchor="middle" class="kb-blue">web1.conf</text>
<text x="555.0" y="63.5" text-anchor="middle" class="km-blue">server web1:80;</text>
<path d="M362 60 L396 50" class="ag"/>
<rect x="400" y="96" width="310" height="60" rx="7" class="k-red"/>
<text x="555.0" y="121.5" text-anchor="middle" class="kb-red">web2.conf</text>
<text x="555.0" y="139.5" text-anchor="middle" class="km-red">server web2:80 down;</text>
<path d="M362 60 L396 126" class="ag"/>
<rect x="400" y="172" width="310" height="60" rx="7" class="k-blue"/>
<text x="555.0" y="197.5" text-anchor="middle" class="kb-blue">web3.conf</text>
<text x="555.0" y="215.5" text-anchor="middle" class="km-blue">server web3:80;</text>
<path d="M362 60 L396 202" class="ag"/>
<text x="540" y="252" text-anchor="middle" class="c">drain one server by changing one file, then reload</text>
</g>
</svg>
<figcaption>One file per backend: the rollout can take out exactly one server.</figcaption>
</figure>

`force: false` creates each backend file but never overwrites it: from now on, the rollout owns those files. `site.yml` gains a play:

```yaml
- name: Load balancer
  hosts: loadbalancers
  become: true
  roles:
    - loadbalancer
```

```
ansible-playbook site.yml
for i in 1 2 3 4 5 6; do curl -s http://lb/ | grep -o 'Served by web.'; done
```

```
Served by web1
Served by web2
Served by web3
Served by web1
...
```

Requests go round the three servers in turn. Add a `web4` to the inventory and the next run of `site.yml` adds it to the balancer: nobody edits the load balancer's configuration by hand.

## Why not just run site.yml?

Running the normal playbook with a new version would update all three servers **at the same time**: during the change, every server is half old, half new, and if the new version is broken, every server is broken. Two play settings change that:

- `serial`: run the **whole play** on a batch of hosts, then the next batch.
- `max_fail_percentage`: stop the whole run when more than this share of a batch fails.

<figure class="diagram">
<svg viewBox="0 0 720 310" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Three ways to run the same play on three web servers. Without serial, all three are updated at once, so all are out together. With serial 1, one at a time: web1, then web2, then web3, two always serving. With serial [1, &#x27;100%&#x27;], web1 alone as a canary, then web2 and web3 together." xmlns="http://www.w3.org/2000/svg">
<style>
.a12c{ font-family:var(--sans),sans-serif; }
.a12c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a12c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a12c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a12c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a12c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a12c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a12c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a12c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a12c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a12c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a12c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a12c .s{ fill:var(--text-dim); font-size:13px; }
.a12c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a12c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a12c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a12c .c{ fill:var(--text-faint); font-size:12.5px; }
.a12c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a12c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a12c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a12c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12c-a); }
.a12c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12c-a); marker-start:url(#a12c-s); }
.a12c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a12c-g); }
.a12c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a12c-a); }
.a12c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a12c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a12c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a12c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a12c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a12c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12c-tri); }
.a12c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12c-tri); }
.a12c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12c-open); }
.a12c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12c-open); }
.a12c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12c-dia); }
.a12c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12c-odia); }
.a12c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12c-dia); marker-end:url(#a12c-open); }
.a12c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12c-solid); }
.a12c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a12c-open); }
.a12c .fill{ fill:var(--text); stroke:none; }
.a12c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a12c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a12c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a12c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a12c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a12c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a12c .ks-blue{ fill:#185FA5; font-size:13px; }
.a12c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a12c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a12c .kb-blue, :root[data-theme="dark"] .a12c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a12c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a12c .kh-blue{ fill:#85B7EB; }
.a12c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a12c-blue); }
.a12c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12c-blue); }
.a12c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a12c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a12c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a12c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a12c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a12c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a12c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a12c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a12c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a12c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker></defs>
<g class="a12c">
<text x="20" y="46" text-anchor="start" class="ma">no serial</text>
<rect x="200" y="16" width="70" height="22" rx="4" class="k-blue"/>
<text x="235.0" y="31.5" text-anchor="middle" class="kb-blue">web1</text>
<rect x="200" y="42" width="70" height="22" rx="4" class="k-blue"/>
<text x="235.0" y="57.5" text-anchor="middle" class="kb-blue">web2</text>
<rect x="200" y="68" width="70" height="22" rx="4" class="k-blue"/>
<text x="235.0" y="83.5" text-anchor="middle" class="kb-blue">web3</text>
<text x="480" y="46" text-anchor="start" class="s">all at once: if it's bad, all are bad</text>
<text x="20" y="146" text-anchor="start" class="ma">serial: 1</text>
<rect x="200" y="120" width="70" height="22" rx="4" class="k-blue"/>
<text x="235.0" y="135.5" text-anchor="middle" class="kb-blue">web1</text>
<path d="M272 132 L286 132" class="ag"/>
<rect x="290" y="120" width="70" height="22" rx="4" class="k-blue"/>
<text x="325.0" y="135.5" text-anchor="middle" class="kb-blue">web2</text>
<path d="M362 132 L376 132" class="ag"/>
<rect x="380" y="120" width="70" height="22" rx="4" class="k-blue"/>
<text x="415.0" y="135.5" text-anchor="middle" class="kb-blue">web3</text>
<text x="480" y="146" text-anchor="start" class="s">one at a time: two always serve</text>
<text x="20" y="246" text-anchor="start" class="ma">serial: [1, "100%"]</text>
<rect x="200" y="220" width="70" height="22" rx="4" class="k-blue"/>
<text x="235.0" y="235.5" text-anchor="middle" class="kb-blue">web1</text>
<path d="M272 232 L286 232" class="ag"/>
<rect x="290" y="216" width="70" height="22" rx="4" class="k-blue"/>
<text x="325.0" y="231.5" text-anchor="middle" class="kb-blue">web2</text>
<rect x="290" y="242" width="70" height="22" rx="4" class="k-blue"/>
<text x="325.0" y="257.5" text-anchor="middle" class="kb-blue">web3</text>
<text x="480" y="246" text-anchor="start" class="s">a canary, then the rest</text>
</g>
</svg>
<figcaption>serial turns one play into batches.</figcaption>
</figure>

`serial: 1` updates one server at a time. `serial: "30%"` takes a share of the hosts per batch, and a list such as `serial: [1, "50%"]` starts with a single **canary** server, then moves faster once it has proven itself.

## The rollout playbook

```yaml
---
- name: Roll out a new version, one web server at a time
  hosts: web
  become: true
  serial: 1
  max_fail_percentage: 0

  pre_tasks:
    - name: Take the server out of the load balancer
      ansible.builtin.copy:
        dest: /etc/nginx/backends/{{ inventory_hostname }}.conf
        content: "server {{ inventory_hostname }}:80 down;\n"
        mode: "0644"
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"

    - name: Apply that on the load balancer
      ansible.builtin.service:
        name: nginx
        state: reloaded
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"

  tasks:
    - name: Deploy the new version
      ansible.builtin.template:
        src: roles/webserver/templates/index.html.j2
        dest: /var/www/html/index.html
        mode: "0644"

    - name: Check the server before users see it
      ansible.builtin.uri:
        url: "http://{{ inventory_hostname }}/health"
      register: health
      until: health.status == 200
      retries: 5
      delay: 2
      delegate_to: localhost
      become: false

  post_tasks:
    - name: Put it back in the load balancer
      ansible.builtin.copy:
        dest: /etc/nginx/backends/{{ inventory_hostname }}.conf
        content: "server {{ inventory_hostname }}:80;\n"
        mode: "0644"
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"

    - name: Apply that on the load balancer
      ansible.builtin.service:
        name: nginx
        state: reloaded
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"
```

<figure class="diagram">
<svg viewBox="0 0 720 380" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A sequence diagram of one batch. Control tells lb to mark web1 down and reload. Control deploys the new version on web1. Control checks http://web1/health and gets 200. Control tells lb to put web1 back and reload. Then the next batch, web2, begins." xmlns="http://www.w3.org/2000/svg">
<style>
.a12d{ font-family:var(--sans),sans-serif; }
.a12d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a12d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a12d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a12d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a12d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a12d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a12d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a12d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a12d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a12d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a12d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a12d .s{ fill:var(--text-dim); font-size:13px; }
.a12d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a12d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a12d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a12d .c{ fill:var(--text-faint); font-size:12.5px; }
.a12d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a12d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a12d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a12d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12d-a); }
.a12d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12d-a); marker-start:url(#a12d-s); }
.a12d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a12d-g); }
.a12d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a12d-a); }
.a12d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a12d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a12d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a12d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a12d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a12d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12d-tri); }
.a12d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12d-tri); }
.a12d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12d-open); }
.a12d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12d-open); }
.a12d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12d-dia); }
.a12d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12d-odia); }
.a12d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12d-dia); marker-end:url(#a12d-open); }
.a12d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12d-solid); }
.a12d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a12d-open); }
.a12d .fill{ fill:var(--text); stroke:none; }
.a12d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a12d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a12d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a12d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a12d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a12d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a12d .ks-blue{ fill:#185FA5; font-size:13px; }
.a12d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a12d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a12d .kb-blue, :root[data-theme="dark"] .a12d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a12d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a12d .kh-blue{ fill:#85B7EB; }
.a12d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a12d-blue); }
.a12d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12d-blue); }
.a12d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a12d .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a12d .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a12d .ks-teal{ fill:#0F6E56; font-size:13px; }
.a12d .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a12d .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12d .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a12d .kb-teal, :root[data-theme="dark"] .a12d .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a12d .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a12d .kh-teal{ fill:#5DCAA5; }
.a12d .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a12d-teal); }
.a12d .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12d-teal); }
.a12d .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a12d .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a12d .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a12d .ks-purple{ fill:#534AB7; font-size:13px; }
.a12d .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a12d .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12d .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a12d .kb-purple, :root[data-theme="dark"] .a12d .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a12d .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a12d .kh-purple{ fill:#AFA9EC; }
.a12d .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a12d-purple); }
.a12d .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12d-purple); }
.a12d .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a12d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a12d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a12d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a12d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a12d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a12d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a12d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a12d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a12d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a12d-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a12d-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a12d">
<rect x="45.0" y="12" width="130" height="36" rx="4" class="k-purple"/>
<text x="110.0" y="34.5" text-anchor="middle" class="kb-purple">control</text>
<line x1="110" y1="48" x2="110" y2="310" class="gd"/>
<rect x="295.0" y="12" width="130" height="36" rx="4" class="k-teal"/>
<text x="360.0" y="34.5" text-anchor="middle" class="kb-teal">lb</text>
<line x1="360" y1="48" x2="360" y2="310" class="gd"/>
<rect x="535.0" y="12" width="130" height="36" rx="4" class="k-blue"/>
<text x="600.0" y="34.5" text-anchor="middle" class="kb-blue">web1</text>
<line x1="600" y1="48" x2="600" y2="310" class="gd"/>
<line x1="115" y1="80" x2="355" y2="80" class="msg"/>
<text x="235.0" y="74" text-anchor="middle" class="s">pre_tasks: web1.conf = down; reload</text>
<line x1="115" y1="132" x2="595" y2="132" class="msg"/>
<text x="480" y="126" text-anchor="middle" class="s">tasks: deploy version 2.0</text>
<line x1="115" y1="184" x2="595" y2="184" class="msg"/>
<text x="480" y="178" text-anchor="middle" class="s">uri http://web1/health  (until 200)</text>
<line x1="595" y1="236" x2="115" y2="236" class="ret"/>
<text x="480" y="230" text-anchor="middle" class="c">200 OK web1</text>
<line x1="115" y1="288" x2="355" y2="288" class="msg"/>
<text x="235.0" y="282" text-anchor="middle" class="s">post_tasks: web1.conf = up; reload</text>
<rect x="355" y="70" width="10" height="28" class="act"/>
<rect x="595" y="122" width="10" height="28" class="act"/>
<rect x="355" y="278" width="10" height="24" class="act"/>
<rect x="160" y="330" width="400" height="36" rx="5" class="pnl"/>
<text x="360.0" y="352.5" text-anchor="middle" class="s">then the next batch: web2, then web3</text>
</g>
</svg>
<figcaption>Drain, update, check, return: for each server in turn.</figcaption>
</figure>

For each web server, in turn:

1. **pre_tasks** mark it `down` in the load balancer and reload it: new requests go to the other two. `delegate_to` runs these tasks **on lb**, while `inventory_hostname` still names the web server being updated. Looping over `groups['loadbalancers']` works for any number of balancers.
2. **tasks** deploy the new version and check `/health` from control, retrying for up to ten seconds.
3. **post_tasks** put it back and reload the balancer.

In a real deployment, step 2 would install a package or a container image and restart the application; the shape of the play stays the same.

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="delegate_to in the rollout. The play&#x27;s current host is web2, so inventory_hostname is web2. The task &#x27;Take the server out&#x27; is delegated to lb: it runs on lb, connecting there, but its variables are web2&#x27;s, so it writes /etc/nginx/backends/web2.conf on lb." xmlns="http://www.w3.org/2000/svg">
<style>
.a12e{ font-family:var(--sans),sans-serif; }
.a12e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a12e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a12e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a12e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a12e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a12e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a12e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a12e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a12e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a12e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a12e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a12e .s{ fill:var(--text-dim); font-size:13px; }
.a12e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a12e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a12e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a12e .c{ fill:var(--text-faint); font-size:12.5px; }
.a12e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a12e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a12e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a12e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12e-a); }
.a12e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12e-a); marker-start:url(#a12e-s); }
.a12e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a12e-g); }
.a12e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a12e-a); }
.a12e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a12e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a12e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a12e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a12e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a12e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12e-tri); }
.a12e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12e-tri); }
.a12e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12e-open); }
.a12e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12e-open); }
.a12e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12e-dia); }
.a12e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12e-odia); }
.a12e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12e-dia); marker-end:url(#a12e-open); }
.a12e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12e-solid); }
.a12e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a12e-open); }
.a12e .fill{ fill:var(--text); stroke:none; }
.a12e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a12e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a12e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a12e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a12e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a12e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a12e .ks-blue{ fill:#185FA5; font-size:13px; }
.a12e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a12e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a12e .kb-blue, :root[data-theme="dark"] .a12e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a12e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a12e .kh-blue{ fill:#85B7EB; }
.a12e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a12e-blue); }
.a12e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12e-blue); }
.a12e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a12e .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a12e .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a12e .ks-teal{ fill:#0F6E56; font-size:13px; }
.a12e .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a12e .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12e .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a12e .kb-teal, :root[data-theme="dark"] .a12e .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a12e .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a12e .kh-teal{ fill:#5DCAA5; }
.a12e .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a12e-teal); }
.a12e .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12e-teal); }
.a12e .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a12e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a12e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a12e .ks-purple{ fill:#534AB7; font-size:13px; }
.a12e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a12e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a12e .kb-purple, :root[data-theme="dark"] .a12e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a12e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a12e .kh-purple{ fill:#AFA9EC; }
.a12e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a12e-purple); }
.a12e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12e-purple); }
.a12e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a12e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a12e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a12e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a12e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a12e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a12e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a12e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a12e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a12e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a12e-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a12e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a12e">
<rect x="20" y="70" width="180" height="70" rx="7" class="k-blue"/>
<text x="110.0" y="91.5" text-anchor="middle" class="kb-blue">current host: web2</text>
<text x="110.0" y="109.5" text-anchor="middle" class="km-blue">inventory_hostname</text>
<text x="110.0" y="127.5" text-anchor="middle" class="km-blue">= web2</text>
<rect x="250" y="70" width="200" height="70" rx="7" class="k-purple"/>
<text x="350.0" y="100.5" text-anchor="middle" class="kb-purple">task: take it out</text>
<text x="350.0" y="118.5" text-anchor="middle" class="km-purple">delegate_to: lb</text>
<path d="M202 105 L246 105" class="ar"/>
<rect x="510" y="40" width="190" height="54" rx="7" class="k-teal"/>
<line x1="520" y1="49" x2="536" y2="49" class="ln-teal"/>
<line x1="520" y1="54" x2="536" y2="54" class="ln-teal"/>
<text x="613.0" y="62" text-anchor="middle" class="kb-teal">lb</text>
<text x="605.0" y="81" text-anchor="middle" class="ks-teal">the task runs here</text>
<rect x="510" y="120" width="190" height="50" rx="7" class="pnl"/>
<text x="605.0" y="140.5" text-anchor="middle" class="m">backends/web2.conf</text>
<text x="605.0" y="158.5" text-anchor="middle" class="c">written on lb</text>
<path d="M452 95 L506 70" class="ar-teal"/>
<text x="360" y="205" text-anchor="middle" class="kh-purple">where the task runs: lb.   whose variables: web2's.</text>
</g>
</svg>
<figcaption>Delegation separates where a task runs from whom it is about.</figcaption>
</figure>

## Watch it from the users' side

A loop in the background plays the customer, asking for the home page ten times a second through the load balancer, while the rollout runs:

```
(for i in $(seq 1 120); do curl -s -m 2 -o /dev/null -w '%{http_code} ' http://lb/; sleep 0.1; done > /tmp/watch.log) &
ansible-playbook rollout.yml -e webserver_app_version=2.0 | grep -E '^PLAY \[|^(web|lb)'
wait
tr ' ' '\n' < /tmp/watch.log | sort | uniq -c
```

```
PLAY [Roll out a new version, one web server at a time] ************************
PLAY [Roll out a new version, one web server at a time] ************************
PLAY [Roll out a new version, one web server at a time] ************************
web1   : ok=7    changed=5    unreachable=0    failed=0 ...
web2   : ok=7    changed=5    unreachable=0    failed=0 ...
web3   : ok=7    changed=5    unreachable=0    failed=0 ...
    120 200
```

The *PLAY* line appears once per batch: three batches of one. All 120 requests got a **200**: customers never noticed.

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The customers&#x27; view during the rollout: 120 requests over twelve seconds, all answered with status 200. Underneath, web1, web2 and web3 are each drained for about three seconds in turn, but at least two servers serve at every moment." xmlns="http://www.w3.org/2000/svg">
<style>
.a12f{ font-family:var(--sans),sans-serif; }
.a12f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a12f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a12f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a12f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a12f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a12f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a12f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a12f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a12f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a12f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a12f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a12f .s{ fill:var(--text-dim); font-size:13px; }
.a12f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a12f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a12f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a12f .c{ fill:var(--text-faint); font-size:12.5px; }
.a12f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a12f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a12f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a12f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12f-a); }
.a12f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12f-a); marker-start:url(#a12f-s); }
.a12f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a12f-g); }
.a12f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a12f-a); }
.a12f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a12f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a12f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a12f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a12f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a12f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12f-tri); }
.a12f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12f-tri); }
.a12f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12f-open); }
.a12f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12f-open); }
.a12f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12f-dia); }
.a12f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12f-odia); }
.a12f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12f-dia); marker-end:url(#a12f-open); }
.a12f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12f-solid); }
.a12f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a12f-open); }
.a12f .fill{ fill:var(--text); stroke:none; }
.a12f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a12f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a12f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a12f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a12f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a12f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a12f .ks-blue{ fill:#185FA5; font-size:13px; }
.a12f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a12f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a12f .kb-blue, :root[data-theme="dark"] .a12f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a12f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a12f .kh-blue{ fill:#85B7EB; }
.a12f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a12f-blue); }
.a12f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12f-blue); }
.a12f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a12f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a12f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a12f .ks-green{ fill:#3B6D11; font-size:13px; }
.a12f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a12f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a12f .kb-green, :root[data-theme="dark"] .a12f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a12f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a12f .kh-green{ fill:#97C459; }
.a12f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a12f-green); }
.a12f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12f-green); }
.a12f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a12f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a12f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a12f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a12f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a12f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a12f .kb-gray, :root[data-theme="dark"] .a12f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a12f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a12f .kh-gray{ fill:#B4B2A9; }
.a12f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a12f-gray); }
.a12f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12f-gray); }
.a12f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a12f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a12f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a12f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a12f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a12f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a12f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a12f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a12f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a12f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a12f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a12f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a12f">
<text x="20" y="30" text-anchor="start" class="c">requests</text>
<rect x="90" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="100" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="110" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="120" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="130" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="140" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="150" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="160" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="170" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="180" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="190" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="200" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="210" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="220" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="230" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="240" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="250" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="260" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="270" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="280" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="290" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="300" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="310" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="320" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="330" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="340" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="350" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="360" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="370" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="380" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="390" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="400" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="410" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="420" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="430" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="440" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="450" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="460" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="470" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="480" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="490" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="500" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="510" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="520" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="530" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="540" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="550" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="560" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="570" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="580" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="590" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="600" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="610" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="620" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="630" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="640" y="18" width="7" height="20" rx="1" class="k-green"/>
<rect x="650" y="18" width="7" height="20" rx="1" class="k-green"/>
<text x="700" y="32" text-anchor="end" class="kh-green">200</text>
<text x="80" y="86" text-anchor="end" class="kb-blue">web1</text>
<rect x="90" y="70" width="600" height="22" rx="4" class="k-blue"/>
<rect x="130" y="70" width="160" height="22" rx="4" class="k-gray"/>
<text x="210" y="86" text-anchor="middle" class="ks-gray">drained, updating</text>
<text x="80" y="120" text-anchor="end" class="kb-blue">web2</text>
<rect x="90" y="104" width="600" height="22" rx="4" class="k-blue"/>
<rect x="320" y="104" width="160" height="22" rx="4" class="k-gray"/>
<text x="400" y="120" text-anchor="middle" class="ks-gray">drained, updating</text>
<text x="80" y="154" text-anchor="end" class="kb-blue">web3</text>
<rect x="90" y="138" width="600" height="22" rx="4" class="k-blue"/>
<rect x="510" y="138" width="160" height="22" rx="4" class="k-gray"/>
<text x="590" y="154" text-anchor="middle" class="ks-gray">drained, updating</text>
<line x1="90" y1="182" x2="690" y2="182" class="ln"/>
<text x="90" y="200" text-anchor="middle" class="c">0 s</text>
<text x="690" y="200" text-anchor="middle" class="c">12 s</text>
</g>
</svg>
<figcaption>120 requests during the rollout, 120 answers with status 200.</figcaption>
</figure>

## A rollout that goes wrong

Break web2 (its web server stops), then roll out version 3.0:

```
ansible web2 -b -a 'service nginx stop'
ansible-playbook rollout.yml -e webserver_app_version=3.0 | grep -E '^PLAY \[|^(web|lb)|NO MORE HOSTS|fatal'
```

```
PLAY [Roll out a new version, one web server at a time] ************************
PLAY [Roll out a new version, one web server at a time] ************************
fatal: [web2 -> localhost]: FAILED! => {"attempts": 5, "changed": false, "msg": "Status code was -1 and not [200]: ..."}
NO MORE HOSTS LEFT *************************************************************
web1   : ok=7    changed=5    unreachable=0    failed=0 ...
web2   : ok=4    changed=3    unreachable=0    failed=1 ...
```

```
for h in web1 web2 web3; do echo "$h: $(curl -s -m 2 http://$h/ | grep -o 'version [0-9.]*' || echo DOWN)"; done
web1: version 3.0
web2: DOWN
web3: version 2.0
ssh lb cat /etc/nginx/backends/web2.conf
server web2:80 down;
```

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The failed rollout. web1 is drained, updated to 3.0, checked and returned. web2 is drained and updated, but its health check fails five times, so the rollout stops, with max_fail_percentage 0. web2 stays drained; web3 is never touched and keeps version 2.0. Customers are served by web1 and web3." xmlns="http://www.w3.org/2000/svg">
<style>
.a12g{ font-family:var(--sans),sans-serif; }
.a12g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a12g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a12g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a12g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a12g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a12g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a12g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a12g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a12g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a12g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a12g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a12g .s{ fill:var(--text-dim); font-size:13px; }
.a12g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a12g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a12g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a12g .c{ fill:var(--text-faint); font-size:12.5px; }
.a12g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a12g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a12g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a12g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12g-a); }
.a12g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a12g-a); marker-start:url(#a12g-s); }
.a12g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a12g-g); }
.a12g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-a); }
.a12g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a12g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a12g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a12g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a12g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a12g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12g-tri); }
.a12g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-tri); }
.a12g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12g-open); }
.a12g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-open); }
.a12g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12g-dia); }
.a12g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12g-odia); }
.a12g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a12g-dia); marker-end:url(#a12g-open); }
.a12g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a12g-solid); }
.a12g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a12g-open); }
.a12g .fill{ fill:var(--text); stroke:none; }
.a12g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a12g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a12g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a12g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a12g .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a12g .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a12g .ks-blue{ fill:#185FA5; font-size:13px; }
.a12g .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a12g .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12g .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a12g .kb-blue, :root[data-theme="dark"] .a12g .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a12g .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a12g .kh-blue{ fill:#85B7EB; }
.a12g .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a12g-blue); }
.a12g .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-blue); }
.a12g .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a12g .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a12g .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a12g .ks-teal{ fill:#0F6E56; font-size:13px; }
.a12g .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a12g .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12g .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a12g .kb-teal, :root[data-theme="dark"] .a12g .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a12g .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a12g .kh-teal{ fill:#5DCAA5; }
.a12g .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a12g-teal); }
.a12g .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-teal); }
.a12g .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a12g .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a12g .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a12g .ks-green{ fill:#3B6D11; font-size:13px; }
.a12g .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a12g .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12g .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a12g .kb-green, :root[data-theme="dark"] .a12g .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a12g .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a12g .kh-green{ fill:#97C459; }
.a12g .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a12g-green); }
.a12g .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-green); }
.a12g .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a12g .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a12g .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a12g .ks-red{ fill:#A32D2D; font-size:13px; }
.a12g .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a12g .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12g .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a12g .kb-red, :root[data-theme="dark"] .a12g .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a12g .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a12g .kh-red{ fill:#F09595; }
.a12g .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a12g-red); }
.a12g .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-red); }
.a12g .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a12g .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a12g .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a12g .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a12g .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a12g .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a12g .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a12g .kb-gray, :root[data-theme="dark"] .a12g .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a12g .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a12g .kh-gray{ fill:#B4B2A9; }
.a12g .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a12g-gray); }
.a12g .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a12g-gray); }
.a12g .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a12g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a12g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a12g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a12g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a12g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a12g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a12g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a12g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a12g-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a12g-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a12g-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a12g-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a12g-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a12g">
<text x="70" y="56" text-anchor="end" class="kb-blue">web1</text>
<rect x="90" y="30" width="140" height="40" rx="5" class="k-teal"/>
<text x="160.0" y="54.5" text-anchor="middle" class="kb-teal">drain</text>
<rect x="242" y="30" width="140" height="40" rx="5" class="k-blue"/>
<text x="312.0" y="54.5" text-anchor="middle" class="kb-blue">3.0</text>
<rect x="394" y="30" width="140" height="40" rx="5" class="k-green"/>
<text x="464.0" y="54.5" text-anchor="middle" class="kb-green">check ok</text>
<rect x="546" y="30" width="140" height="40" rx="5" class="k-teal"/>
<text x="616.0" y="54.5" text-anchor="middle" class="kb-teal">return</text>
<text x="70" y="112" text-anchor="end" class="kb-blue">web2</text>
<rect x="90" y="86" width="140" height="40" rx="5" class="k-teal"/>
<text x="160.0" y="110.5" text-anchor="middle" class="kb-teal">drain</text>
<rect x="242" y="86" width="140" height="40" rx="5" class="k-blue"/>
<text x="312.0" y="110.5" text-anchor="middle" class="kb-blue">3.0</text>
<rect x="394" y="86" width="140" height="40" rx="5" class="k-red"/>
<text x="464.0" y="110.5" text-anchor="middle" class="kb-red">check FAILED</text>
<rect x="546" y="86" width="140" height="40" rx="5" class="k-gray"/>
<text x="616.0" y="110.5" text-anchor="middle" class="kb-gray">(stays out)</text>
<text x="70" y="168" text-anchor="end" class="kb-blue">web3</text>
<rect x="90" y="142" width="600" height="40" rx="5" class="k-gray"/>
<text x="390.0" y="166.5" text-anchor="middle" class="kb-gray">not touched: still 2.0</text>
<text x="400" y="212" text-anchor="middle" class="kh-red">max_fail_percentage: 0 stopped the play: NO MORE HOSTS LEFT</text>
</g>
</svg>
<figcaption>A failed check stops the rollout; the damage stays on one server, out of the balancer.</figcaption>
</figure>

web1 was updated. web2 failed its health check, and `max_fail_percentage: 0` stopped the **whole** rollout: web3 was never touched and still runs 2.0. web2 stays marked `down` in the load balancer, because its post_tasks never ran, so customers kept getting pages from web1 and web3. Without the health check, the broken server would have been put back in the balancer; without `serial`, every server would have been out at once.

Fix web2 and run the same rollout again. web1 is already at 3.0, so only its drain and return report changes; web2 and web3 are updated; all three end up back in the balancer:

```
ansible web2 -b -a 'service nginx start'
ansible-playbook rollout.yml -e webserver_app_version=3.0
```

## Recording what is deployed

`-e webserver_app_version=3.0` is right for trying a release, but it leaves no record. Once a version is out, write it into `group_vars/web/vars.yml` and commit: the repository then says what production runs, and the next `site.yml` run won't quietly roll the page back to an older version. Many teams have their CI pipeline do exactly this: change the version in Git, review it, and let the pipeline run the rollout.

## Other tools for orchestration

| Setting or keyword | Effect |
|---|---|
| `serial: 1`, `"25%"`, `[1, 5, "100%"]` | batch size, or a growing list of batch sizes |
| `max_fail_percentage: N` | stop when more than N % of a batch fails |
| `any_errors_fatal: true` | stop everything at the first failure |
| `throttle: 1` | one host at a time, for a single task |
| `run_once: true` | one host runs the task for all (a database migration) |
| `order: sorted` / `reverse_sorted` / `shuffle` | the order hosts are taken in |
| `delegate_to: HOST` | run a task on another machine with this host's variables |
| `wait_for`, `uri` with `until` | wait until a service is really ready |

## Hands on

1. Use `serial: [1, "100%"]`: a canary, then all the rest. What does the output look like?
2. Add web4 to the topology and the inventory, run `site.yml`: is it in the balancer? Run the rollout.
3. Write `rollback.yml` that imports `rollout.yml` (`import_playbook`) with the previous version.
4. Wrap the deploy and check in a `block` whose `rescue` deploys the previous version on the failing host, so that it can go back into the balancer.
5. Capture the lb's cable in DockerLab during a rollout and watch the requests move from backend to backend.

## Summary

- A **load balancer** built from `groups['web']` follows the inventory; one file per backend makes draining a single server easy.
- `serial` runs a play in **batches**; `max_fail_percentage` stops a rollout before a failure spreads; a list of batch sizes gives a **canary** first.
- `pre_tasks` drain a server, `tasks` update and **health-check** it, `post_tasks` return it; `delegate_to` runs the balancer's part on the balancer.
- Watching from the users' side proves zero downtime: every request answered.
- A failed check stops the rollout, leaves the bad server out of the balancer, and the untouched servers keep serving.
- Record the deployed version in Git, not only on the command line.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-12-rolling-updates).

**1. Why does the loadbalancer role write one backend file per web server?**

- A. nginx requires it
- B. So the rollout can take out exactly one server by changing only its own file
- C. To use less memory
- D. To balance by IP

**2. Why is the load balancer's group called loadbalancers rather than lb?**

- A. lb is a reserved word
- B. Group names need a plural
- C. A group and a host with the same name confuse Ansible
- D. nginx needs it

**3. What does serial: 1 do?**

- A. Runs the whole play on one host at a time
- B. Runs one task at a time
- C. Allows one failure
- D. Uses one fork

**4. What does max_fail_percentage: 0 do in the rollout?**

- A. Stops the whole rollout as soon as any host in a batch fails
- B. Retries failures forever
- C. Ignores failures
- D. Rolls back automatically

**5. In pre_tasks, a task has delegate_to: lb while the play's current host is web2. Where does it run, and with whose variables?**

- A. On web2, with lb's variables
- B. On control, with lb's variables
- C. On lb, with web2's variables
- D. On lb, with lb's variables

**6. What proved the rollout caused no downtime?**

- A. The playbook recap
- B. A background loop requesting the page through lb, which got 200 for every request
- C. nginx logs
- D. The health check alone

**7. web2 fails its health check during a rollout. What state is it left in?**

- A. Back in the load balancer
- B. Rolled back to the old version
- C. Deleted
- D. Marked down in the load balancer, because its post_tasks never ran

**8. What does serial: [1, "100%"] give you?**

- A. One server at a time
- B. A canary: one server first, then all the rest at once
- C. Half the servers
- D. No batching
