# Chapter 5: Playbooks

Ad-hoc commands do one thing at a time, and leave no record. Northwind's web servers need nginx installed, a site configuration, a home page and a running service; the database server needs SQLite and a data folder. That is a **job**, with steps in a fixed order, and it should be written down: in a file you can read, review, keep in Git and run again whenever a server is rebuilt. That file is a **playbook**.

> **Try it:** DockerLab, *Ansible lab 4, A first playbook*. Same topology. The finished project is in `examples/ch05`.

## YAML in five minutes

Ansible writes almost everything in **YAML**, a format for data that people can read. Three structures cover nearly all of it:

```
cat > yaml-demo.yml <<'EOF'
# a comment
name: Northwind Coffee          # a key and a value (a string)
port: 8080                      # a number
secure: true                    # a boolean
servers:                        # a list: one item per "- "
  - web1
  - web2
owner:                          # a dictionary (map) inside
  name: Alice
  email: alice@northwind.lab
message: "Quote a value that contains ': ' or starts with {{"
EOF
python3 -c 'import yaml, json; print(json.dumps(yaml.safe_load(open("yaml-demo.yml")), indent=2))'
```

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="YAML and the data it means. Scalars: name Northwind Coffee is a string, port 8080 a number, secure true a boolean. A list is written with a dash per item: servers web1 and web2. A dictionary is written as indented key: value pairs: owner with name and email. The same data in JSON uses quotes, brackets and braces." xmlns="http://www.w3.org/2000/svg">
<style>
.a5a{ font-family:var(--sans),sans-serif; }
.a5a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a5a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a5a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a5a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a5a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a5a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a5a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a5a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a5a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a5a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a5a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a5a .s{ fill:var(--text-dim); font-size:13px; }
.a5a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a5a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a5a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a5a .c{ fill:var(--text-faint); font-size:12.5px; }
.a5a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a5a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a5a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a5a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5a-a); }
.a5a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5a-a); marker-start:url(#a5a-s); }
.a5a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a5a-g); }
.a5a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a5a-a); }
.a5a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a5a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a5a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a5a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a5a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a5a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5a-tri); }
.a5a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5a-tri); }
.a5a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5a-open); }
.a5a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5a-open); }
.a5a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5a-dia); }
.a5a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5a-odia); }
.a5a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5a-dia); marker-end:url(#a5a-open); }
.a5a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5a-solid); }
.a5a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a5a-open); }
.a5a .fill{ fill:var(--text); stroke:none; }
.a5a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a5a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a5a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a5a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a5a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a5a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a5a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a5a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a5a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a5a .kb-teal, :root[data-theme="dark"] .a5a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a5a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a5a .kh-teal{ fill:#5DCAA5; }
.a5a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a5a-teal); }
.a5a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5a-teal); }
.a5a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a5a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a5a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a5a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a5a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a5a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a5a .kb-gray, :root[data-theme="dark"] .a5a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a5a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a5a .kh-gray{ fill:#B4B2A9; }
.a5a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a5a-gray); }
.a5a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5a-gray); }
.a5a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a5a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a5a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a5a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a5a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a5a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a5a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a5a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a5a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a5a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a5a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a5a">
<rect x="20" y="16" width="320" height="227" rx="6" class="k-teal"/>
<text x="30" y="36" text-anchor="start" class="kh-teal">YAML</text>
<text x="32.0" y="58" text-anchor="start" class="m">name: Northwind Coffee</text>
<text x="32.0" y="79" text-anchor="start" class="m">port: 8080</text>
<text x="32.0" y="100" text-anchor="start" class="m">secure: true</text>
<text x="32.0" y="121" text-anchor="start" class="m">servers:</text>
<text x="47.2" y="142" text-anchor="start" class="m">- web1</text>
<text x="47.2" y="163" text-anchor="start" class="m">- web2</text>
<text x="32.0" y="184" text-anchor="start" class="m">owner:</text>
<text x="47.2" y="205" text-anchor="start" class="m">name: Alice</text>
<text x="47.2" y="226" text-anchor="start" class="m">email: alice@northwind.lab</text>
<rect x="400" y="16" width="300" height="227" rx="6" class="k-gray"/>
<text x="410" y="36" text-anchor="start" class="kh-gray">the same, as JSON</text>
<text x="412.0" y="58" text-anchor="start" class="m">{</text>
<text x="427.2" y="79" text-anchor="start" class="m">"name": "Northwind Coffee",</text>
<text x="427.2" y="100" text-anchor="start" class="m">"port": 8080,</text>
<text x="427.2" y="121" text-anchor="start" class="m">"secure": true,</text>
<text x="427.2" y="142" text-anchor="start" class="m">"servers": ["web1", "web2"],</text>
<text x="427.2" y="163" text-anchor="start" class="m">"owner": {</text>
<text x="442.4" y="184" text-anchor="start" class="m">"name": "Alice",</text>
<text x="442.4" y="205" text-anchor="start" class="m">"email": "alice@..." }</text>
<text x="412.0" y="226" text-anchor="start" class="m">}</text>
<path d="M344 140 L396 140" class="ar-teal"/>
<text x="370" y="130" text-anchor="middle" class="c">means</text>
</g>
</svg>
<figcaption>Scalars, lists (- item) and dictionaries (key: value), nested by indentation.</figcaption>
</figure>

Nesting is shown by **indentation with spaces**, never tabs. Most YAML errors are a wrong indent or a missing space after `:` or `-`. Two more rules save trouble: quote values that start with `{{` (Ansible variables), and quote file modes like `"0644"`, which YAML would otherwise read as a number.

## A first playbook

The playbook below turns two bare servers into web servers and prepares the database server. It goes in `~/northwind/site.yml`, the conventional name for the playbook that describes the whole site:

```yaml
---
- name: Web servers
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true
        cache_valid_time: 3600

    - name: Configure the site (IPv4 only)
      ansible.builtin.copy:
        dest: /etc/nginx/sites-available/default
        content: |
          server {
              listen 80 default_server;
              root /var/www/html;
              index index.html;
              server_name _;
          }
        mode: "0644"

    - name: Put the home page in place
      ansible.builtin.copy:
        dest: /var/www/html/index.html
        content: "<h1>Northwind Coffee</h1><p>Served by {{ inventory_hostname }}</p>\n"
        mode: "0644"

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started

- name: Database server
  hosts: db
  become: true

  tasks:
    - name: Install SQLite
      ansible.builtin.apt:
        name: sqlite3
        state: present
        update_cache: true
        cache_valid_time: 3600

    - name: Make the data folder
      ansible.builtin.file:
        path: /srv/data
        state: directory
        owner: student
        mode: "0750"
```

<figure class="diagram">
<svg viewBox="0 0 720 360" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The anatomy of a playbook. The file starts with three dashes and holds a list of plays. A play has a name, hosts (a pattern from the inventory), become, and a list of tasks. A task has a name and one module, written with its fully qualified name, followed by the module&#x27;s parameters." xmlns="http://www.w3.org/2000/svg">
<style>
.a5b{ font-family:var(--sans),sans-serif; }
.a5b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a5b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a5b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a5b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a5b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a5b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a5b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a5b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a5b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a5b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a5b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a5b .s{ fill:var(--text-dim); font-size:13px; }
.a5b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a5b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .c{ fill:var(--text-faint); font-size:12.5px; }
.a5b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a5b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a5b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a5b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5b-a); }
.a5b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5b-a); marker-start:url(#a5b-s); }
.a5b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a5b-g); }
.a5b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-a); }
.a5b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a5b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a5b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a5b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a5b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a5b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5b-tri); }
.a5b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-tri); }
.a5b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5b-open); }
.a5b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-open); }
.a5b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5b-dia); }
.a5b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5b-odia); }
.a5b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5b-dia); marker-end:url(#a5b-open); }
.a5b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5b-solid); }
.a5b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a5b-open); }
.a5b .fill{ fill:var(--text); stroke:none; }
.a5b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a5b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a5b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a5b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a5b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a5b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a5b .ks-blue{ fill:#185FA5; font-size:13px; }
.a5b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a5b .kb-blue, :root[data-theme="dark"] .a5b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a5b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a5b .kh-blue{ fill:#85B7EB; }
.a5b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a5b-blue); }
.a5b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-blue); }
.a5b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a5b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a5b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a5b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a5b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a5b .kb-teal, :root[data-theme="dark"] .a5b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a5b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a5b .kh-teal{ fill:#5DCAA5; }
.a5b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a5b-teal); }
.a5b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-teal); }
.a5b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a5b .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a5b .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a5b .ks-amber{ fill:#854F0B; font-size:13px; }
.a5b .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5b .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a5b .kb-amber, :root[data-theme="dark"] .a5b .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a5b .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a5b .kh-amber{ fill:#EF9F27; }
.a5b .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a5b-amber); }
.a5b .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-amber); }
.a5b .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a5b .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a5b .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a5b .ks-coral{ fill:#993C1D; font-size:13px; }
.a5b .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5b .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a5b .kb-coral, :root[data-theme="dark"] .a5b .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a5b .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a5b .kh-coral{ fill:#F0997B; }
.a5b .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a5b-coral); }
.a5b .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-coral); }
.a5b .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a5b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a5b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a5b .ks-purple{ fill:#534AB7; font-size:13px; }
.a5b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a5b .kb-purple, :root[data-theme="dark"] .a5b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a5b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a5b .kh-purple{ fill:#AFA9EC; }
.a5b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a5b-purple); }
.a5b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-purple); }
.a5b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a5b .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a5b .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a5b .ks-green{ fill:#3B6D11; font-size:13px; }
.a5b .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5b .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a5b .kb-green, :root[data-theme="dark"] .a5b .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a5b .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a5b .kh-green{ fill:#97C459; }
.a5b .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a5b-green); }
.a5b .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-green); }
.a5b .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a5b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a5b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a5b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a5b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a5b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a5b .kb-gray, :root[data-theme="dark"] .a5b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a5b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a5b .kh-gray{ fill:#B4B2A9; }
.a5b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a5b-gray); }
.a5b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5b-gray); }
.a5b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a5b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a5b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a5b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a5b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a5b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a5b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a5b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a5b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a5b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a5b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a5b-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a5b-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a5b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a5b-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a5b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a5b">
<rect x="20" y="16" width="330" height="289" rx="6" class="pnl"/>
<text x="32.0" y="36" text-anchor="start" class="m">---</text>
<text x="32.0" y="57" text-anchor="start" class="m">- name: Web servers</text>
<text x="47.2" y="78" text-anchor="start" class="m">hosts: web</text>
<text x="47.2" y="99" text-anchor="start" class="m">become: true</text>
<text x="32.0" y="120" text-anchor="start" class="m"></text>
<text x="47.2" y="141" text-anchor="start" class="m">tasks:</text>
<text x="62.4" y="162" text-anchor="start" class="m">- name: Install nginx</text>
<text x="77.6" y="183" text-anchor="start" class="m">ansible.builtin.apt:</text>
<text x="92.8" y="204" text-anchor="start" class="m">name: nginx</text>
<text x="92.8" y="225" text-anchor="start" class="m">state: present</text>
<text x="32.0" y="246" text-anchor="start" class="m"></text>
<text x="62.4" y="267" text-anchor="start" class="m">- name: Start nginx</text>
<text x="77.6" y="288" text-anchor="start" class="m">ansible.builtin.service: ...</text>
<line x1="352" y1="33" x2="400" y2="33" class="ln-gray"/>
<text x="408" y="37" text-anchor="start" class="kh-gray">start of the YAML document</text>
<line x1="352" y1="54" x2="400" y2="54" class="ln-blue"/>
<text x="408" y="58" text-anchor="start" class="kh-blue">a play: a list item</text>
<line x1="352" y1="75" x2="400" y2="75" class="ln-teal"/>
<text x="408" y="79" text-anchor="start" class="kh-teal">which hosts: a pattern</text>
<line x1="352" y1="96" x2="400" y2="96" class="ln-coral"/>
<text x="408" y="100" text-anchor="start" class="kh-coral">run tasks as root</text>
<line x1="352" y1="138" x2="400" y2="138" class="ln-blue"/>
<text x="408" y="142" text-anchor="start" class="kh-blue">the play's list of tasks</text>
<line x1="352" y1="159" x2="400" y2="159" class="ln-purple"/>
<text x="408" y="163" text-anchor="start" class="kh-purple">a task: its name, printed in the output</text>
<line x1="352" y1="180" x2="400" y2="180" class="ln-green"/>
<text x="408" y="184" text-anchor="start" class="kh-green">one module, by full name</text>
<line x1="352" y1="211" x2="400" y2="211" class="ln-amber"/>
<text x="408" y="215" text-anchor="start" class="kh-amber">the module's parameters: desired state</text>
<text x="560" y="330" text-anchor="middle" class="s">a play = hosts + tasks; a task = name + one module</text>
</g>
</svg>
<figcaption>Every playbook has this shape: plays, which hold tasks, which call modules.</figcaption>
</figure>

A playbook is a **list of plays**. Each play says:

- `name`: what it is for, printed in the output.
- `hosts`: a host pattern, here a group from the inventory.
- `become: true`: run its tasks as root.
- `tasks`: a list of tasks, each with a `name` and **one module** with its parameters, the same modules as Chapter 3, now written as YAML.

The site configuration replaces Debian's default one, which also listens on IPv6 that these containers may not have. `content: |` starts a **literal block**: the indented lines below it are kept exactly, newlines included. `{{ inventory_hostname }}` is the host's name in the inventory; Chapter 6 is about variables.

## Run it

```
cd ~/northwind
ansible-playbook site.yml
```

```
PLAY [Web servers] *************************************************************

TASK [Gathering Facts] *********************************************************
ok: [web1]
ok: [web2]

TASK [Install nginx] ***********************************************************
changed: [web2]
changed: [web1]

TASK [Configure the site (IPv4 only)] ******************************************
changed: [web1]
changed: [web2]
...
PLAY RECAP *********************************************************************
db1    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web1   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web2   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Each play starts with **Gathering Facts**, the `setup` module run for you. Then every task runs on every host of the play before the next task starts, so the output is grouped by task. The **PLAY RECAP** sums up each host: how many tasks were ok, changed, failed, and so on.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The default linear strategy. Within a play, the first task runs on all hosts (in parallel, up to the forks limit) and Ansible waits for all of them before starting the second task on all hosts, and so on. Plays run one after another: the web play, then the database play." xmlns="http://www.w3.org/2000/svg">
<style>
.a5c{ font-family:var(--sans),sans-serif; }
.a5c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a5c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a5c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a5c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a5c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a5c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a5c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a5c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a5c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a5c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a5c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a5c .s{ fill:var(--text-dim); font-size:13px; }
.a5c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a5c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a5c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a5c .c{ fill:var(--text-faint); font-size:12.5px; }
.a5c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a5c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a5c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a5c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5c-a); }
.a5c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5c-a); marker-start:url(#a5c-s); }
.a5c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a5c-g); }
.a5c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a5c-a); }
.a5c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a5c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a5c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a5c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a5c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a5c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5c-tri); }
.a5c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5c-tri); }
.a5c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5c-open); }
.a5c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5c-open); }
.a5c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5c-dia); }
.a5c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5c-odia); }
.a5c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5c-dia); marker-end:url(#a5c-open); }
.a5c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5c-solid); }
.a5c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a5c-open); }
.a5c .fill{ fill:var(--text); stroke:none; }
.a5c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a5c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a5c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a5c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a5c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a5c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a5c .ks-blue{ fill:#185FA5; font-size:13px; }
.a5c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a5c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a5c .kb-blue, :root[data-theme="dark"] .a5c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a5c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a5c .kh-blue{ fill:#85B7EB; }
.a5c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a5c-blue); }
.a5c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5c-blue); }
.a5c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a5c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a5c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a5c .ks-amber{ fill:#854F0B; font-size:13px; }
.a5c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a5c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a5c .kb-amber, :root[data-theme="dark"] .a5c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a5c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a5c .kh-amber{ fill:#EF9F27; }
.a5c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a5c-amber); }
.a5c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5c-amber); }
.a5c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
</style>
<defs><marker id="a5c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a5c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a5c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a5c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a5c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a5c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a5c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a5c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a5c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a5c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker></defs>
<g class="a5c">
<text x="80" y="40" text-anchor="middle" class="b">play 1: web</text>
<text x="60" y="82" text-anchor="end" class="kb-blue">web1</text>
<text x="60" y="122" text-anchor="end" class="kb-blue">web2</text>
<text x="126" y="62" text-anchor="middle" class="c">facts</text>
<rect x="80" y="66" width="92" height="28" rx="5" class="k-blue"/>
<text x="126" y="85" text-anchor="middle" class="ks-blue">ok</text>
<rect x="80" y="106" width="92" height="28" rx="5" class="k-blue"/>
<text x="126" y="125" text-anchor="middle" class="ks-blue">ok</text>
<line x1="178" y1="60" x2="178" y2="150" class="gd"/>
<text x="230" y="62" text-anchor="middle" class="c">install nginx</text>
<rect x="184" y="66" width="92" height="28" rx="5" class="k-blue"/>
<text x="230" y="85" text-anchor="middle" class="ks-blue">changed</text>
<rect x="184" y="106" width="92" height="28" rx="5" class="k-blue"/>
<text x="230" y="125" text-anchor="middle" class="ks-blue">changed</text>
<line x1="282" y1="60" x2="282" y2="150" class="gd"/>
<text x="334" y="62" text-anchor="middle" class="c">configure site</text>
<rect x="288" y="66" width="92" height="28" rx="5" class="k-blue"/>
<text x="334" y="85" text-anchor="middle" class="ks-blue">changed</text>
<rect x="288" y="106" width="92" height="28" rx="5" class="k-blue"/>
<text x="334" y="125" text-anchor="middle" class="ks-blue">changed</text>
<line x1="386" y1="60" x2="386" y2="150" class="gd"/>
<text x="438" y="62" text-anchor="middle" class="c">home page</text>
<rect x="392" y="66" width="92" height="28" rx="5" class="k-blue"/>
<text x="438" y="85" text-anchor="middle" class="ks-blue">changed</text>
<rect x="392" y="106" width="92" height="28" rx="5" class="k-blue"/>
<text x="438" y="125" text-anchor="middle" class="ks-blue">changed</text>
<line x1="490" y1="60" x2="490" y2="150" class="gd"/>
<text x="542" y="62" text-anchor="middle" class="c">start nginx</text>
<rect x="496" y="66" width="92" height="28" rx="5" class="k-blue"/>
<text x="542" y="85" text-anchor="middle" class="ks-blue">changed</text>
<rect x="496" y="106" width="92" height="28" rx="5" class="k-blue"/>
<text x="542" y="125" text-anchor="middle" class="ks-blue">changed</text>
<text x="80" y="180" text-anchor="middle" class="b">play 2: db</text>
<rect x="80" y="196" width="92" height="28" rx="5" class="k-amber"/>
<text x="126" y="215" text-anchor="middle" class="ks-amber">facts</text>
<rect x="184" y="196" width="92" height="28" rx="5" class="k-amber"/>
<text x="230" y="215" text-anchor="middle" class="ks-amber">install sqlite3</text>
<rect x="288" y="196" width="92" height="28" rx="5" class="k-amber"/>
<text x="334" y="215" text-anchor="middle" class="ks-amber">data folder</text>
<text x="60" y="215" text-anchor="end" class="kb-amber">db1</text>
<path d="M600 120 L600 160" class="ar"/>
<text x="612" y="146" text-anchor="start" class="c">then</text>
<text x="500" y="240" text-anchor="middle" class="c">dashed line: every host finishes a task before the next task</text>
</g>
</svg>
<figcaption>Task by task across hosts, play after play.</figcaption>
</figure>

```
curl -s http://web1
curl -s http://web2
```

```
<h1>Northwind Coffee</h1><p>Served by web1</p>
<h1>Northwind Coffee</h1><p>Served by web2</p>
```

## Run it again

```
ansible-playbook site.yml
```

```
PLAY RECAP *********************************************************************
db1    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web1   : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web2   : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

`changed=0` everywhere: the servers already look like the playbook says, so nothing is done. This is what makes a playbook safe to run whenever you like: after rebuilding a server, when a new one joins the group, or every night, to undo anything changed by hand. A playbook that reports changes on its second run has a problem, usually a `command` or `shell` task that should be a module.

## Before running: check it

Changing servers is serious, so Ansible offers several ways to look before you leap:

```
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --list-hosts
ansible-playbook site.yml --list-tasks
```

`--syntax-check` parses the playbook without connecting anywhere. `--list-hosts` and `--list-tasks` show what would run, and where. Then a dry run shows what would change. Change the heading on the home page, then:

```
sed -i 's|<h1>Northwind Coffee</h1>|<h1>Northwind Coffee: now open on Sundays</h1>|' site.yml
ansible-playbook site.yml --check --diff --limit web1
```

```
TASK [Put the home page in place] **********************************************
--- before: /var/www/html/index.html
+++ after: /var/www/html/index.html
@@ -1 +1 @@
-<h1>Northwind Coffee</h1><p>Served by web1</p>
+<h1>Northwind Coffee: now open on Sundays</h1><p>Served by web1</p>
changed: [web1]
```

Nothing was changed: `--check` only reports. Now apply it to the staging server first, look, then to all:

```
ansible-playbook site.yml --limit web2
curl -s http://web2; curl -s http://web1
ansible-playbook site.yml
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A careful change workflow: syntax-check, then list tasks and hosts, then a dry run with check and diff, then apply to one host with limit, inspect it, and finally apply to all hosts." xmlns="http://www.w3.org/2000/svg">
<style>
.a5d{ font-family:var(--sans),sans-serif; }
.a5d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a5d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a5d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a5d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a5d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a5d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a5d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a5d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a5d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a5d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a5d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a5d .s{ fill:var(--text-dim); font-size:13px; }
.a5d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a5d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a5d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a5d .c{ fill:var(--text-faint); font-size:12.5px; }
.a5d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a5d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a5d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a5d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5d-a); }
.a5d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5d-a); marker-start:url(#a5d-s); }
.a5d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a5d-g); }
.a5d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-a); }
.a5d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a5d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a5d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a5d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a5d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a5d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5d-tri); }
.a5d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-tri); }
.a5d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5d-open); }
.a5d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-open); }
.a5d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5d-dia); }
.a5d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5d-odia); }
.a5d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5d-dia); marker-end:url(#a5d-open); }
.a5d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5d-solid); }
.a5d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a5d-open); }
.a5d .fill{ fill:var(--text); stroke:none; }
.a5d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a5d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a5d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a5d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a5d .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a5d .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a5d .ks-teal{ fill:#0F6E56; font-size:13px; }
.a5d .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a5d .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5d .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a5d .kb-teal, :root[data-theme="dark"] .a5d .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a5d .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a5d .kh-teal{ fill:#5DCAA5; }
.a5d .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a5d-teal); }
.a5d .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-teal); }
.a5d .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a5d .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a5d .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a5d .ks-amber{ fill:#854F0B; font-size:13px; }
.a5d .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a5d .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5d .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a5d .kb-amber, :root[data-theme="dark"] .a5d .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a5d .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a5d .kh-amber{ fill:#EF9F27; }
.a5d .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a5d-amber); }
.a5d .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-amber); }
.a5d .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a5d .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a5d .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a5d .ks-coral{ fill:#993C1D; font-size:13px; }
.a5d .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a5d .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5d .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a5d .kb-coral, :root[data-theme="dark"] .a5d .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a5d .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a5d .kh-coral{ fill:#F0997B; }
.a5d .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a5d-coral); }
.a5d .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-coral); }
.a5d .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a5d .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a5d .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a5d .ks-green{ fill:#3B6D11; font-size:13px; }
.a5d .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a5d .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5d .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a5d .kb-green, :root[data-theme="dark"] .a5d .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a5d .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a5d .kh-green{ fill:#97C459; }
.a5d .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a5d-green); }
.a5d .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-green); }
.a5d .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a5d .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a5d .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a5d .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a5d .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a5d .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5d .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a5d .kb-gray, :root[data-theme="dark"] .a5d .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a5d .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a5d .kh-gray{ fill:#B4B2A9; }
.a5d .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a5d-gray); }
.a5d .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5d-gray); }
.a5d .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a5d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a5d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a5d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a5d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a5d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a5d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a5d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a5d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a5d-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a5d-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a5d-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a5d-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a5d-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a5d">
<rect x="30" y="46" width="200" height="60" rx="7" class="k-gray"/>
<text x="130.0" y="71.5" text-anchor="middle" class="km-gray">--syntax-check</text>
<text x="130.0" y="89.5" text-anchor="middle" class="ks-gray">parses?</text>
<path d="M232 76 L256 76" class="ar"/>
<rect x="260" y="46" width="200" height="60" rx="7" class="k-gray"/>
<text x="360.0" y="71.5" text-anchor="middle" class="km-gray">--list-tasks</text>
<text x="360.0" y="89.5" text-anchor="middle" class="ks-gray">what, where</text>
<path d="M462 76 L486 76" class="ar"/>
<rect x="490" y="46" width="200" height="60" rx="7" class="k-amber"/>
<text x="590.0" y="71.5" text-anchor="middle" class="km-amber">--check --diff</text>
<text x="590.0" y="89.5" text-anchor="middle" class="ks-amber">what would change</text>
<rect x="30" y="136" width="200" height="60" rx="7" class="k-coral"/>
<text x="130.0" y="161.5" text-anchor="middle" class="km-coral">--limit web2</text>
<text x="130.0" y="179.5" text-anchor="middle" class="ks-coral">one host first</text>
<path d="M232 166 L256 166" class="ar"/>
<rect x="260" y="136" width="200" height="60" rx="7" class="k-teal"/>
<text x="360.0" y="161.5" text-anchor="middle" class="km-teal">curl, look</text>
<text x="360.0" y="179.5" text-anchor="middle" class="ks-teal">it works?</text>
<path d="M462 166 L486 166" class="ar"/>
<rect x="490" y="136" width="200" height="60" rx="7" class="k-green"/>
<text x="590.0" y="161.5" text-anchor="middle" class="km-green">all hosts</text>
<text x="590.0" y="179.5" text-anchor="middle" class="ks-green">apply</text>
<path d="M590 108 C 590 124, 130 120, 130 132" class="ar"/>
<text x="360" y="30" text-anchor="middle" class="b">look before you leap</text>
<text x="360" y="236" text-anchor="middle" class="c">each step is cheap; skipping them is how outages start</text>
</g>
</svg>
<figcaption>From an edited playbook to every server, safely.</figcaption>
</figure>

This sequence, check syntax, preview, apply to one host, apply to all, is how careful operators change production. Later chapters add automated checks (`ansible-lint`, Chapter 10) and verification playbooks (Chapter 14).

## When a task fails

Misspell the package name and run the database play:

```
sed -i 's/name: sqlite3/name: sqlite-three/' site.yml
ansible-playbook site.yml --limit db1
```

```
TASK [Install SQLite] **********************************************************
fatal: [db1]: FAILED! => {"changed": false, "msg": "No package matching 'sqlite-three' is available"}

PLAY RECAP *********************************************************************
db1    : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

A failed task stops the play **for that host**: its remaining tasks are skipped. Other hosts carry on. Read the `msg`: it usually says exactly what is wrong. Fix the playbook and run it again; tasks that already succeeded simply report ok.

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A task fails on one host. On db1, Install SQLite fails, so the remaining task Make the data folder is skipped for db1 and the recap counts failed=1. Other hosts in the same play would continue with their remaining tasks." xmlns="http://www.w3.org/2000/svg">
<style>
.a5e{ font-family:var(--sans),sans-serif; }
.a5e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a5e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a5e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a5e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a5e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a5e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a5e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a5e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a5e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a5e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a5e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a5e .s{ fill:var(--text-dim); font-size:13px; }
.a5e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a5e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a5e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a5e .c{ fill:var(--text-faint); font-size:12.5px; }
.a5e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a5e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a5e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a5e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5e-a); }
.a5e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a5e-a); marker-start:url(#a5e-s); }
.a5e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a5e-g); }
.a5e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a5e-a); }
.a5e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a5e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a5e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a5e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a5e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a5e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5e-tri); }
.a5e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5e-tri); }
.a5e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5e-open); }
.a5e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a5e-open); }
.a5e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5e-dia); }
.a5e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5e-odia); }
.a5e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a5e-dia); marker-end:url(#a5e-open); }
.a5e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a5e-solid); }
.a5e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a5e-open); }
.a5e .fill{ fill:var(--text); stroke:none; }
.a5e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a5e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a5e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a5e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a5e .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a5e .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a5e .ks-green{ fill:#3B6D11; font-size:13px; }
.a5e .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a5e .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5e .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a5e .kb-green, :root[data-theme="dark"] .a5e .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a5e .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a5e .kh-green{ fill:#97C459; }
.a5e .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a5e-green); }
.a5e .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5e-green); }
.a5e .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a5e .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a5e .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a5e .ks-red{ fill:#A32D2D; font-size:13px; }
.a5e .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a5e .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5e .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a5e .kb-red, :root[data-theme="dark"] .a5e .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a5e .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a5e .kh-red{ fill:#F09595; }
.a5e .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a5e-red); }
.a5e .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5e-red); }
.a5e .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a5e .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a5e .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a5e .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a5e .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a5e .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a5e .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a5e .kb-gray, :root[data-theme="dark"] .a5e .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a5e .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a5e .kh-gray{ fill:#B4B2A9; }
.a5e .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a5e-gray); }
.a5e .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a5e-gray); }
.a5e .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a5e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a5e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a5e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a5e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a5e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a5e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a5e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a5e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a5e-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a5e-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a5e-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a5e">
<text x="200" y="30" text-anchor="middle" class="c">facts</text>
<text x="350" y="30" text-anchor="middle" class="c">task 1</text>
<text x="500" y="30" text-anchor="middle" class="c">task 2</text>
<text x="100" y="64" text-anchor="end" class="b">web1</text>
<rect x="140" y="44" width="120" height="32" rx="5" class="k-green"/>
<text x="200.0" y="64.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="290" y="44" width="120" height="32" rx="5" class="k-green"/>
<text x="350.0" y="64.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="440" y="44" width="120" height="32" rx="5" class="k-green"/>
<text x="500.0" y="64.5" text-anchor="middle" class="kb-green">ok</text>
<text x="100" y="108" text-anchor="end" class="b">web2</text>
<rect x="140" y="88" width="120" height="32" rx="5" class="k-green"/>
<text x="200.0" y="108.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="290" y="88" width="120" height="32" rx="5" class="k-green"/>
<text x="350.0" y="108.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="440" y="88" width="120" height="32" rx="5" class="k-green"/>
<text x="500.0" y="108.5" text-anchor="middle" class="kb-green">ok</text>
<text x="100" y="152" text-anchor="end" class="b">db1</text>
<rect x="140" y="132" width="120" height="32" rx="5" class="k-green"/>
<text x="200.0" y="152.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="290" y="132" width="120" height="32" rx="5" class="k-red"/>
<text x="350.0" y="152.5" text-anchor="middle" class="kb-red">FAILED</text>
<rect x="440" y="132" width="120" height="32" rx="5" class="k-gray"/>
<text x="500.0" y="152.5" text-anchor="middle" class="kb-gray">not run</text>
<path d="M262 148 L288 148" class="ar-red"/>
<rect x="140" y="180" width="420" height="32" rx="5" class="pnl"/>
<text x="350.0" y="200.5" text-anchor="middle" class="m">recap: db1  ok=2  failed=1   web1, web2: failed=0</text>
</g>
</svg>
<figcaption>A failure stops that host, not the play.</figcaption>
</figure>

```
sed -i 's/name: sqlite-three/name: sqlite3/' site.yml
ansible-playbook site.yml --limit db1 --start-at-task 'Make the data folder'
ansible-playbook site.yml --limit web1 -v
```

`--start-at-task` skips the tasks before the named one, handy in long playbooks. `-v` shows each module's result; `-vv`, `-vvv` and `-vvvv` show more, down to the SSH commands. Chapter 8 shows how to handle failures inside a playbook, with `block` and `rescue`.

## Writing good tasks

- **Name every task**, in plain words, saying what the task achieves: the names are the playbook's documentation and its log.
- Use **full module names** (`ansible.builtin.copy`): unambiguous, and required by `ansible-lint`'s stricter profiles.
- One module per task, with parameters as YAML (`name: nginx`) rather than the `name=nginx` form of ad-hoc commands.
- Prefer modules over `command` and `shell`, so the playbook stays idempotent.
- Quote modes (`"0644"`) and values starting with `{{`.

## Playbook command options

| Option | Effect |
|---|---|
| `--syntax-check` | parse only |
| `--list-hosts`, `--list-tasks`, `--list-tags` | show what would run |
| `-C`, `--check` | dry run: report changes, make none |
| `-D`, `--diff` | show file changes |
| `-l`, `--limit PATTERN` | run on fewer hosts |
| `--start-at-task NAME` | start in the middle |
| `--step` | ask before each task |
| `-e VAR=VALUE` | set a variable (Chapter 6) |
| `-t`, `--tags`, `--skip-tags` | run part of a playbook (Chapter 9) |
| `-v` to `-vvvv` | more output |
| `-f N` | forks |

## Hands on

1. Add a first play, for `hosts: all`, that installs `curl` and `less` on every server. Where must it go to run first?
2. Stop nginx on web1 by hand (`ansible web1 -b -a 'service nginx stop'`) and run the playbook. Which task reports changed?
3. Change the site to listen on port 8080 and run the playbook. Why does `curl http://web1:8080` fail until nginx restarts? (Chapter 9 fixes this properly.)
4. Run `ansible-lint site.yml`. What does it suggest?

## Summary

- A **playbook** is a YAML file holding a list of **plays**; each play targets hosts and lists **tasks**; each task calls one **module**.
- YAML nests by **indentation with spaces**; quote modes and values starting with `{{`.
- `ansible-playbook` gathers facts, runs each task on all hosts of the play, then the next task, and ends with a **recap**.
- A second run with `changed=0` proves the playbook is **idempotent**.
- Look before you leap: `--syntax-check`, `--list-tasks`, `--check --diff`, `--limit` one host, then all.
- A failed task stops that host's play; the other hosts continue.
