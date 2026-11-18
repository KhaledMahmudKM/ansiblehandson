# Chapter 6: Variables and Facts

Northwind's playbook works, but it hard-codes everything: the site name, the port, the folders. When the staging server should say *(staging)* and listen on a different port, you don't want a second playbook. **Variables** let one playbook build different sites for different hosts; **facts** let it adapt to what each server actually is; **registered results** let one task's output feed the next. This chapter adds all three to `~/northwind`.

> **Try it:** DockerLab, *Ansible lab 5, Variables and facts*. Same topology. The finished project is in `examples/ch06`.

## Variables in a playbook

A variable is a name for a value. You use it in any task parameter, or in a template, with **Jinja2** expressions in double braces:

<figure class="diagram">
<svg viewBox="0 0 720 200" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A Jinja2 expression taken apart: double braces open and close it; inside, a variable name, owners, then a pipe passes its value to the filter join, whose argument is the string &#x27; and &#x27;. The result, Alice and Bob, replaces the whole expression." xmlns="http://www.w3.org/2000/svg">
<style>
.a6a{ font-family:var(--sans),sans-serif; }
.a6a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a6a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a6a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a6a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a6a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a6a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a6a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a6a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a6a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a6a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a6a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a6a .s{ fill:var(--text-dim); font-size:13px; }
.a6a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a6a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a6a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a6a .c{ fill:var(--text-faint); font-size:12.5px; }
.a6a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a6a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a6a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a6a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6a-a); }
.a6a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6a-a); marker-start:url(#a6a-s); }
.a6a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a6a-g); }
.a6a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-a); }
.a6a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a6a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a6a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a6a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a6a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a6a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6a-tri); }
.a6a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-tri); }
.a6a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6a-open); }
.a6a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-open); }
.a6a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6a-dia); }
.a6a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6a-odia); }
.a6a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6a-dia); marker-end:url(#a6a-open); }
.a6a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6a-solid); }
.a6a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a6a-open); }
.a6a .fill{ fill:var(--text); stroke:none; }
.a6a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a6a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a6a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a6a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a6a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a6a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a6a .ks-blue{ fill:#185FA5; font-size:13px; }
.a6a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a6a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a6a .kb-blue, :root[data-theme="dark"] .a6a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a6a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a6a .kh-blue{ fill:#85B7EB; }
.a6a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a6a-blue); }
.a6a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-blue); }
.a6a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a6a .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a6a .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a6a .ks-amber{ fill:#854F0B; font-size:13px; }
.a6a .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a6a .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6a .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a6a .kb-amber, :root[data-theme="dark"] .a6a .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a6a .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a6a .kh-amber{ fill:#EF9F27; }
.a6a .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a6a-amber); }
.a6a .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-amber); }
.a6a .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a6a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a6a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a6a .ks-purple{ fill:#534AB7; font-size:13px; }
.a6a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a6a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a6a .kb-purple, :root[data-theme="dark"] .a6a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a6a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a6a .kh-purple{ fill:#AFA9EC; }
.a6a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a6a-purple); }
.a6a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-purple); }
.a6a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a6a .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a6a .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a6a .ks-green{ fill:#3B6D11; font-size:13px; }
.a6a .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a6a .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6a .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a6a .kb-green, :root[data-theme="dark"] .a6a .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a6a .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a6a .kh-green{ fill:#97C459; }
.a6a .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a6a-green); }
.a6a .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-green); }
.a6a .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a6a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a6a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a6a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a6a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a6a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a6a .kb-gray, :root[data-theme="dark"] .a6a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a6a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a6a .kh-gray{ fill:#B4B2A9; }
.a6a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a6a-gray); }
.a6a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6a-gray); }
.a6a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a6a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a6a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a6a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a6a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a6a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a6a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a6a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a6a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a6a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a6a-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a6a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a6a-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a6a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a6a">
<rect x="70" y="30" width="44" height="44" rx="5" class="k-gray"/>
<text x="92.0" y="56.5" text-anchor="middle" class="km-gray">{{</text>
<line x1="92.0" y1="78" x2="92.0" y2="100" class="ln-gray"/>
<text x="92.0" y="118" text-anchor="middle" class="kh-gray">open</text>
<rect x="122" y="30" width="100" height="44" rx="5" class="k-blue"/>
<text x="172.0" y="56.5" text-anchor="middle" class="km-blue">owners</text>
<line x1="172.0" y1="78" x2="172.0" y2="100" class="ln-blue"/>
<text x="172.0" y="118" text-anchor="middle" class="kh-blue">a variable</text>
<rect x="230" y="30" width="34" height="44" rx="5" class="k-gray"/>
<text x="247.0" y="56.5" text-anchor="middle" class="km-gray">|</text>
<line x1="247.0" y1="78" x2="247.0" y2="100" class="ln-gray"/>
<text x="247.0" y="118" text-anchor="middle" class="kh-gray">pipe</text>
<rect x="272" y="30" width="70" height="44" rx="5" class="k-purple"/>
<text x="307.0" y="56.5" text-anchor="middle" class="km-purple">join</text>
<line x1="307.0" y1="78" x2="307.0" y2="100" class="ln-purple"/>
<text x="307.0" y="118" text-anchor="middle" class="kh-purple">a filter</text>
<rect x="350" y="30" width="120" height="44" rx="5" class="k-amber"/>
<text x="410.0" y="56.5" text-anchor="middle" class="km-amber">(' and ')</text>
<line x1="410.0" y1="78" x2="410.0" y2="100" class="ln-amber"/>
<text x="410.0" y="118" text-anchor="middle" class="kh-amber">its argument</text>
<rect x="478" y="30" width="44" height="44" rx="5" class="k-gray"/>
<text x="500.0" y="56.5" text-anchor="middle" class="km-gray">}}</text>
<line x1="500.0" y1="78" x2="500.0" y2="100" class="ln-gray"/>
<text x="500.0" y="118" text-anchor="middle" class="kh-gray">close</text>
<path d="M526 52 L560 52" class="ar"/>
<rect x="564" y="30" width="120" height="44" rx="7" class="k-green"/>
<text x="624.0" y="56.5" text-anchor="middle" class="ks-green">Alice and Bob</text>
<text x="360" y="168" text-anchor="middle" class="c">owners: [Alice, Bob]  -&gt;  the expression's value replaces it in the task or template</text>
</g>
</svg>
<figcaption>Variables, filters and arguments, evaluated on control for each host.</figcaption>
</figure>

Variables can be set in many places. Northwind keeps its settings where Chapter 4 put them, in `group_vars` and `host_vars`:

```
cat > group_vars/web.yml <<'EOF'
site_name: Northwind Coffee
http_port: 80
web_root: /var/www/html
EOF
printf 'site_name: Northwind Coffee (staging)\nhttp_port: 8080\n' > host_vars/web2.yml
printf 'data_dir: /srv/data\n' > group_vars/db.yml
```

and `site.yml` uses them instead of fixed values:

```yaml
    - name: Configure the site (IPv4 only)
      ansible.builtin.copy:
        dest: /etc/nginx/sites-available/default
        content: |
          server {
              listen {{ http_port }} default_server;
              root {{ web_root }};
              index index.html;
              server_name _;
          }
        mode: "0644"

    - name: Put the home page in place
      ansible.builtin.copy:
        dest: "{{ web_root }}/index.html"
        content: "<h1>{{ site_name }}</h1><p>Served by {{ inventory_hostname }}</p>\n"
        mode: "0644"
```

```
ansible-playbook site.yml
curl -s http://web1
```

The same tasks now write a different site on each server. A value that starts with `{{` must be quoted (`dest: "{{ web_root }}/index.html"`), or YAML mistakes it for a dictionary.

> Run the playbook and try `curl http://web2:8080`: nothing answers. The configuration file says 8080, but nginx is still listening on 80, because nothing restarted it. Chapter 9 fixes this with a **handler**.

## Where variables come from

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Sources of variables flowing into one host&#x27;s set of variables: inventory files and group_vars and host_vars, play vars and vars_files, role defaults and vars, facts gathered from the host, registered results and set_fact from earlier tasks, and extra vars from the command line. Together they form the host&#x27;s variables, used by tasks and templates." xmlns="http://www.w3.org/2000/svg">
<style>
.a6b{ font-family:var(--sans),sans-serif; }
.a6b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a6b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a6b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a6b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a6b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a6b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a6b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a6b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a6b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a6b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a6b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a6b .s{ fill:var(--text-dim); font-size:13px; }
.a6b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a6b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .c{ fill:var(--text-faint); font-size:12.5px; }
.a6b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a6b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a6b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a6b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6b-a); }
.a6b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6b-a); marker-start:url(#a6b-s); }
.a6b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a6b-g); }
.a6b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-a); }
.a6b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a6b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a6b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a6b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a6b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a6b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6b-tri); }
.a6b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-tri); }
.a6b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6b-open); }
.a6b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-open); }
.a6b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6b-dia); }
.a6b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6b-odia); }
.a6b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6b-dia); marker-end:url(#a6b-open); }
.a6b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6b-solid); }
.a6b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a6b-open); }
.a6b .fill{ fill:var(--text); stroke:none; }
.a6b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a6b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a6b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a6b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a6b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a6b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a6b .ks-blue{ fill:#185FA5; font-size:13px; }
.a6b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a6b .kb-blue, :root[data-theme="dark"] .a6b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a6b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a6b .kh-blue{ fill:#85B7EB; }
.a6b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a6b-blue); }
.a6b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-blue); }
.a6b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a6b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a6b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a6b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a6b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a6b .kb-teal, :root[data-theme="dark"] .a6b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a6b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a6b .kh-teal{ fill:#5DCAA5; }
.a6b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a6b-teal); }
.a6b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-teal); }
.a6b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a6b .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a6b .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a6b .ks-amber{ fill:#854F0B; font-size:13px; }
.a6b .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6b .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a6b .kb-amber, :root[data-theme="dark"] .a6b .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a6b .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a6b .kh-amber{ fill:#EF9F27; }
.a6b .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a6b-amber); }
.a6b .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-amber); }
.a6b .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a6b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a6b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a6b .ks-purple{ fill:#534AB7; font-size:13px; }
.a6b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a6b .kb-purple, :root[data-theme="dark"] .a6b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a6b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a6b .kh-purple{ fill:#AFA9EC; }
.a6b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a6b-purple); }
.a6b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-purple); }
.a6b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a6b .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a6b .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a6b .ks-green{ fill:#3B6D11; font-size:13px; }
.a6b .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6b .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a6b .kb-green, :root[data-theme="dark"] .a6b .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a6b .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a6b .kh-green{ fill:#97C459; }
.a6b .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a6b-green); }
.a6b .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-green); }
.a6b .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a6b .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a6b .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a6b .ks-red{ fill:#A32D2D; font-size:13px; }
.a6b .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6b .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a6b .kb-red, :root[data-theme="dark"] .a6b .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a6b .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a6b .kh-red{ fill:#F09595; }
.a6b .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a6b-red); }
.a6b .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-red); }
.a6b .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a6b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a6b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a6b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a6b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a6b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a6b .kb-gray, :root[data-theme="dark"] .a6b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a6b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a6b .kh-gray{ fill:#B4B2A9; }
.a6b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a6b-gray); }
.a6b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6b-gray); }
.a6b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a6b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a6b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a6b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a6b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a6b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a6b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a6b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a6b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a6b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a6b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a6b-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a6b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a6b-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a6b-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a6b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a6b">
<rect x="20" y="16" width="280" height="36" rx="6" class="k-teal"/>
<text x="160.0" y="38.5" text-anchor="middle" class="kb-teal">inventory, group_vars, host_vars</text>
<path d="M302 34 C 360 34, 380 140, 420 140" class="ar-teal"/>
<rect x="20" y="60" width="280" height="36" rx="6" class="k-blue"/>
<text x="160.0" y="82.5" text-anchor="middle" class="kb-blue">play vars:, vars_files:</text>
<path d="M302 78 C 360 78, 380 140, 420 140" class="ar-blue"/>
<rect x="20" y="104" width="280" height="36" rx="6" class="k-purple"/>
<text x="160.0" y="126.5" text-anchor="middle" class="kb-purple">role defaults/, vars/</text>
<path d="M302 122 C 360 122, 380 140, 420 140" class="ar-purple"/>
<rect x="20" y="148" width="280" height="36" rx="6" class="k-green"/>
<text x="160.0" y="170.5" text-anchor="middle" class="kb-green">facts (setup)</text>
<path d="M302 166 C 360 166, 380 140, 420 140" class="ar-green"/>
<rect x="20" y="192" width="280" height="36" rx="6" class="k-amber"/>
<text x="160.0" y="214.5" text-anchor="middle" class="kb-amber">register:, set_fact:</text>
<path d="M302 210 C 360 210, 380 140, 420 140" class="ar-amber"/>
<rect x="20" y="236" width="280" height="36" rx="6" class="k-red"/>
<text x="160.0" y="258.5" text-anchor="middle" class="kb-red">-e extra vars</text>
<path d="M302 254 C 360 254, 380 140, 420 140" class="ar-red"/>
<rect x="424" y="100" width="140" height="80" rx="7" class="k-gray"/>
<text x="494.0" y="124.5" text-anchor="middle" class="kb-gray">web2's</text>
<text x="494.0" y="144.5" text-anchor="middle" class="kb-gray">variables</text>
<text x="494.0" y="164.5" text-anchor="middle" class="km-gray">hostvars['web2']</text>
<path d="M566 140 L600 140" class="ar"/>
<rect x="604" y="100" width="100" height="80" rx="7" class="pnl"/>
<text x="654.0" y="135.5" text-anchor="middle" class="b">tasks</text>
<text x="654.0" y="153.5" text-anchor="middle" class="b">templates</text>
</g>
</svg>
<figcaption>Many sources, one merged set of variables per host.</figcaption>
</figure>

| Where | Written as | Typical use |
|---|---|---|
| inventory, `group_vars/`, `host_vars/` | YAML files beside the inventory | settings per group and host: the usual place |
| play `vars:` | a dictionary in the play | values local to one play |
| `vars_files:` | a list of YAML files in the play | shared settings, secrets (Chapter 11) |
| role `defaults/`, `vars/` | files in a role | Chapter 10 |
| **facts** | gathered from each host | adapting to the host |
| `register:` | a task's result | using one task's output in the next |
| `set_fact:` | a task | computing a new value |
| `-e` (extra vars) | command line | one-off runs, overrides |

A play can also define its own:

```yaml
- name: Variables in a play
  hosts: web
  gather_facts: false
  vars:
    owners:
      - Alice
      - Bob
  tasks:
    - name: Show them
      ansible.builtin.debug:
        msg: "{{ site_name }} on {{ http_port }}; owners: {{ owners | join(' and ') }}; first: {{ owners[0] }}"
```

`owners[0]` is the first item of a list; a dictionary's values are reached as `owner.name` or `owner['name']`. `| join(' and ')` passes the list through a **filter**. `gather_facts: false` skips fact gathering, which is faster when no facts are needed.

## Which value wins

When the same variable is set in several places, Ansible uses a fixed **precedence** order. The full list has more than twenty levels; this simplified one covers almost every real case, weakest first:

<figure class="diagram">
<svg viewBox="0 0 720 330" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Simplified variable precedence, weakest at the bottom: role defaults; inventory group_vars all, other groups, then host_vars; facts; play vars and vars_files; role vars, task vars, registered results and set_fact; and extra vars from the command line, which always win." xmlns="http://www.w3.org/2000/svg">
<style>
.a6c{ font-family:var(--sans),sans-serif; }
.a6c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a6c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a6c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a6c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a6c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a6c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a6c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a6c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a6c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a6c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a6c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a6c .s{ fill:var(--text-dim); font-size:13px; }
.a6c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a6c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .c{ fill:var(--text-faint); font-size:12.5px; }
.a6c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a6c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a6c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a6c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6c-a); }
.a6c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6c-a); marker-start:url(#a6c-s); }
.a6c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a6c-g); }
.a6c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-a); }
.a6c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a6c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a6c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a6c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a6c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a6c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6c-tri); }
.a6c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-tri); }
.a6c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6c-open); }
.a6c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-open); }
.a6c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6c-dia); }
.a6c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6c-odia); }
.a6c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6c-dia); marker-end:url(#a6c-open); }
.a6c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6c-solid); }
.a6c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a6c-open); }
.a6c .fill{ fill:var(--text); stroke:none; }
.a6c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a6c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a6c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a6c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a6c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a6c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a6c .ks-blue{ fill:#185FA5; font-size:13px; }
.a6c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a6c .kb-blue, :root[data-theme="dark"] .a6c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a6c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a6c .kh-blue{ fill:#85B7EB; }
.a6c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a6c-blue); }
.a6c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-blue); }
.a6c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a6c .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a6c .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a6c .ks-teal{ fill:#0F6E56; font-size:13px; }
.a6c .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6c .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a6c .kb-teal, :root[data-theme="dark"] .a6c .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a6c .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a6c .kh-teal{ fill:#5DCAA5; }
.a6c .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a6c-teal); }
.a6c .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-teal); }
.a6c .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a6c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a6c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a6c .ks-amber{ fill:#854F0B; font-size:13px; }
.a6c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a6c .kb-amber, :root[data-theme="dark"] .a6c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a6c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a6c .kh-amber{ fill:#EF9F27; }
.a6c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a6c-amber); }
.a6c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-amber); }
.a6c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a6c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a6c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a6c .ks-purple{ fill:#534AB7; font-size:13px; }
.a6c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a6c .kb-purple, :root[data-theme="dark"] .a6c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a6c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a6c .kh-purple{ fill:#AFA9EC; }
.a6c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a6c-purple); }
.a6c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-purple); }
.a6c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a6c .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a6c .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a6c .ks-green{ fill:#3B6D11; font-size:13px; }
.a6c .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6c .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a6c .kb-green, :root[data-theme="dark"] .a6c .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a6c .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a6c .kh-green{ fill:#97C459; }
.a6c .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a6c-green); }
.a6c .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-green); }
.a6c .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a6c .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a6c .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a6c .ks-red{ fill:#A32D2D; font-size:13px; }
.a6c .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a6c .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6c .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a6c .kb-red, :root[data-theme="dark"] .a6c .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a6c .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a6c .kh-red{ fill:#F09595; }
.a6c .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a6c-red); }
.a6c .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6c-red); }
.a6c .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a6c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a6c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a6c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a6c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a6c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a6c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a6c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a6c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a6c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a6c-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a6c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a6c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a6c-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a6c-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a6c">
<rect x="150" y="16" width="420" height="36" rx="5" class="k-red"/>
<text x="360.0" y="38.5" text-anchor="middle" class="kb-red">-e extra vars   (always win)</text>
<rect x="150" y="59" width="420" height="36" rx="5" class="k-amber"/>
<text x="360.0" y="81.5" text-anchor="middle" class="kb-amber">role vars, task vars, register, set_fact   (the code)</text>
<rect x="150" y="102" width="420" height="36" rx="5" class="k-blue"/>
<text x="360.0" y="124.5" text-anchor="middle" class="kb-blue">play vars, vars_files   (this play)</text>
<rect x="150" y="145" width="420" height="36" rx="5" class="k-green"/>
<text x="360.0" y="167.5" text-anchor="middle" class="kb-green">facts   (discovered)</text>
<rect x="150" y="188" width="420" height="36" rx="5" class="k-teal"/>
<text x="360.0" y="210.5" text-anchor="middle" class="kb-teal">host_vars   (one host's data)</text>
<rect x="150" y="231" width="420" height="36" rx="5" class="k-teal"/>
<text x="360.0" y="253.5" text-anchor="middle" class="kb-teal">group_vars: all, then groups   (inventory data)</text>
<rect x="150" y="274" width="420" height="36" rx="5" class="k-purple"/>
<text x="360.0" y="296.5" text-anchor="middle" class="kb-purple">role defaults   (meant to be overridden)</text>
<path d="M110 310 L110 24" class="ar"/>
<text x="100" y="170" text-anchor="end" class="a">stronger</text>
<text x="600" y="40" text-anchor="start" class="kh-red">wins</text>
<text x="600" y="300" text-anchor="start" class="kh-purple">loses</text>
</g>
</svg>
<figcaption>A simplified ladder: the higher a value is set, the more it wins.</figcaption>
</figure>

```
ansible-playbook site.yml --limit web2 -e http_port=9090 --check --diff
```

```
-    listen 8080 default_server;
+    listen 9090 default_server;
```

**Extra variables** given with `-e` always win: useful for a one-off run, such as the version to deploy, but dangerous as a habit, because nothing records them. The practical rule is simpler than the table: keep each variable in **one** obvious place, defaults low and specific values high, and precedence rarely matters.

## Facts: what each host is

Facts are gathered at the start of each play and stored in `ansible_facts`:

```yaml
- name: What are these servers?
  hosts: all
  tasks:
    - name: Describe each host from its facts
      ansible.builtin.debug:
        msg: >-
          {{ inventory_hostname }} runs {{ ansible_facts['distribution'] }}
          {{ ansible_facts['distribution_version'] }} with
          {{ ansible_facts['processor_vcpus'] }} CPUs and
          {{ ansible_facts['memtotal_mb'] }} MB, at
          {{ ansible_facts['eth0']['ipv4']['address'] }}
```

```
ansible-playbook facts.yml
```

```
ok: [web1] => {
    "msg": "web1 runs Debian 13 with 2 CPUs and 7944 MB, at 10.0.0.11"
}
```

`>-` starts a **folded** string: the lines are joined with spaces into one. Facts are nested dictionaries; follow the path one key at a time:

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Following a path into the facts: ansible_facts is a dictionary; its key eth0 holds a dictionary for that interface; its key ipv4 holds a dictionary; its key address holds the string 10.0.0.11." xmlns="http://www.w3.org/2000/svg">
<style>
.a6d{ font-family:var(--sans),sans-serif; }
.a6d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a6d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a6d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a6d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a6d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a6d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a6d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a6d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a6d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a6d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a6d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a6d .s{ fill:var(--text-dim); font-size:13px; }
.a6d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a6d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a6d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a6d .c{ fill:var(--text-faint); font-size:12.5px; }
.a6d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a6d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a6d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a6d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6d-a); }
.a6d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6d-a); marker-start:url(#a6d-s); }
.a6d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a6d-g); }
.a6d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a6d-a); }
.a6d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a6d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a6d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a6d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a6d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a6d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6d-tri); }
.a6d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6d-tri); }
.a6d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6d-open); }
.a6d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6d-open); }
.a6d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6d-dia); }
.a6d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6d-odia); }
.a6d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6d-dia); marker-end:url(#a6d-open); }
.a6d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6d-solid); }
.a6d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a6d-open); }
.a6d .fill{ fill:var(--text); stroke:none; }
.a6d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a6d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a6d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a6d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a6d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a6d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a6d .ks-blue{ fill:#185FA5; font-size:13px; }
.a6d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a6d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a6d .kb-blue, :root[data-theme="dark"] .a6d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a6d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a6d .kh-blue{ fill:#85B7EB; }
.a6d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a6d-blue); }
.a6d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6d-blue); }
.a6d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a6d .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a6d .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a6d .ks-teal{ fill:#0F6E56; font-size:13px; }
.a6d .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a6d .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6d .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a6d .kb-teal, :root[data-theme="dark"] .a6d .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a6d .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a6d .kh-teal{ fill:#5DCAA5; }
.a6d .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a6d-teal); }
.a6d .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6d-teal); }
.a6d .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a6d .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a6d .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a6d .ks-green{ fill:#3B6D11; font-size:13px; }
.a6d .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a6d .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6d .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a6d .kb-green, :root[data-theme="dark"] .a6d .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a6d .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a6d .kh-green{ fill:#97C459; }
.a6d .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a6d-green); }
.a6d .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6d-green); }
.a6d .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a6d .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a6d .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a6d .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a6d .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a6d .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6d .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a6d .kb-gray, :root[data-theme="dark"] .a6d .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a6d .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a6d .kh-gray{ fill:#B4B2A9; }
.a6d .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a6d-gray); }
.a6d .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6d-gray); }
.a6d .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a6d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a6d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a6d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a6d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a6d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a6d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a6d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a6d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a6d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a6d-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a6d-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a6d-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a6d">
<rect x="20" y="40" width="130" height="44" rx="5" class="k-gray"/>
<text x="85.0" y="66.5" text-anchor="middle" class="km-gray">ansible_facts</text>
<text x="85.0" y="104" text-anchor="middle" class="kh-gray">all facts</text>
<path d="M152 62 L172 62" class="ar"/>
<rect x="176" y="40" width="110" height="44" rx="5" class="k-blue"/>
<text x="231.0" y="66.5" text-anchor="middle" class="km-blue">['eth0']</text>
<text x="231.0" y="104" text-anchor="middle" class="kh-blue">an interface</text>
<path d="M288 62 L308 62" class="ar"/>
<rect x="312" y="40" width="110" height="44" rx="5" class="k-teal"/>
<text x="367.0" y="66.5" text-anchor="middle" class="km-teal">['ipv4']</text>
<text x="367.0" y="104" text-anchor="middle" class="kh-teal">its IPv4 settings</text>
<path d="M424 62 L444 62" class="ar"/>
<rect x="448" y="40" width="110" height="44" rx="5" class="k-green"/>
<text x="503.0" y="66.5" text-anchor="middle" class="km-green">['address']</text>
<text x="503.0" y="104" text-anchor="middle" class="kh-green">one value</text>
<path d="M560 62 L580 62" class="ar"/>
<rect x="584" y="40" width="100" height="44" rx="7" class="pnl"/>
<text x="634.0" y="66.5" text-anchor="middle" class="m">10.0.0.11</text>
<rect x="20" y="128" width="680" height="52" rx="6" class="pnl"/>
<text x="32.0" y="148" text-anchor="start" class="m">{{ ansible_facts['eth0']['ipv4']['address'] }}</text>
<text x="32.0" y="166" text-anchor="start" class="m">{{ ansible_facts.eth0.ipv4.address }}      (the same, with dots)</text>
</g>
</svg>
<figcaption>Facts are nested dictionaries; follow the keys.</figcaption>
</figure>

Useful facts include `os_family` and `distribution` (choose `apt` or `dnf`), `distribution_major_version`, `processor_vcpus` and `memtotal_mb` (size workers and caches), `hostname`, `fqdn`, interfaces such as `eth0` with their addresses, and `default_ipv4`. In DockerLab the default route goes out through `nat0`, the internet connection, so `default_ipv4` is that interface; use `eth0` for a host's lab address.

## Results as variables: register

Every task returns a result. `register` keeps it in a variable for later tasks:

```yaml
- name: Use what a task returns
  hosts: all
  gather_facts: false
  tasks:
    - name: How full is the root file system?
      ansible.builtin.command: df --output=pcent /
      register: disk
      changed_when: false

    - name: Turn the output into a number
      ansible.builtin.set_fact:
        disk_used: "{{ disk.stdout_lines[1] | trim | replace('%', '') | int }}"

    - name: Report
      ansible.builtin.debug:
        msg: "{{ inventory_hostname }}: root is {{ disk_used }}% full (rc={{ disk.rc }})"
```

<figure class="diagram">
<svg viewBox="0 0 720 280" style="width:100%;height:auto;max-width:720px" role="img" aria-label="register in action. The command task runs df on the host and its result is saved as disk: cmd, rc 0, stdout, stdout_lines with two lines (Use% and 40%), stderr, changed false because of changed_when, failed false. The next task, set_fact, reads disk.stdout_lines[1] and filters it into disk_used = 40, which the debug task prints." xmlns="http://www.w3.org/2000/svg">
<style>
.a6e{ font-family:var(--sans),sans-serif; }
.a6e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a6e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a6e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a6e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a6e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a6e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a6e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a6e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a6e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a6e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a6e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a6e .s{ fill:var(--text-dim); font-size:13px; }
.a6e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a6e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a6e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a6e .c{ fill:var(--text-faint); font-size:12.5px; }
.a6e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a6e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a6e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a6e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6e-a); }
.a6e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6e-a); marker-start:url(#a6e-s); }
.a6e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a6e-g); }
.a6e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a6e-a); }
.a6e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a6e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a6e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a6e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a6e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a6e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6e-tri); }
.a6e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6e-tri); }
.a6e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6e-open); }
.a6e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6e-open); }
.a6e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6e-dia); }
.a6e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6e-odia); }
.a6e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6e-dia); marker-end:url(#a6e-open); }
.a6e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6e-solid); }
.a6e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a6e-open); }
.a6e .fill{ fill:var(--text); stroke:none; }
.a6e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a6e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a6e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a6e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a6e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a6e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a6e .ks-blue{ fill:#185FA5; font-size:13px; }
.a6e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a6e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a6e .kb-blue, :root[data-theme="dark"] .a6e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a6e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a6e .kh-blue{ fill:#85B7EB; }
.a6e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a6e-blue); }
.a6e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6e-blue); }
.a6e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a6e .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a6e .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a6e .ks-amber{ fill:#854F0B; font-size:13px; }
.a6e .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a6e .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6e .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a6e .kb-amber, :root[data-theme="dark"] .a6e .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a6e .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a6e .kh-amber{ fill:#EF9F27; }
.a6e .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a6e-amber); }
.a6e .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6e-amber); }
.a6e .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a6e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a6e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a6e .ks-purple{ fill:#534AB7; font-size:13px; }
.a6e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a6e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a6e .kb-purple, :root[data-theme="dark"] .a6e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a6e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a6e .kh-purple{ fill:#AFA9EC; }
.a6e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a6e-purple); }
.a6e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6e-purple); }
.a6e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a6e .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a6e .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a6e .ks-green{ fill:#3B6D11; font-size:13px; }
.a6e .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a6e .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6e .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a6e .kb-green, :root[data-theme="dark"] .a6e .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a6e .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a6e .kh-green{ fill:#97C459; }
.a6e .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a6e-green); }
.a6e .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6e-green); }
.a6e .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
</style>
<defs><marker id="a6e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a6e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a6e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a6e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a6e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a6e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a6e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a6e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a6e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a6e-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a6e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a6e-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker></defs>
<g class="a6e">
<rect x="10" y="20" width="245" height="56" rx="7" class="k-blue"/>
<text x="132.5" y="43.5" text-anchor="middle" class="km-blue">command: df --output=pcent /</text>
<text x="132.5" y="61.5" text-anchor="middle" class="km-blue">register: disk</text>
<rect x="280" y="16" width="290" height="149" rx="6" class="k-amber"/>
<text x="292.0" y="36" text-anchor="start" class="m">disk = {</text>
<text x="307.2" y="55" text-anchor="start" class="m">"rc": 0,</text>
<text x="307.2" y="74" text-anchor="start" class="m">"stdout": "Use%\n 40%",</text>
<text x="307.2" y="93" text-anchor="start" class="m">"stdout_lines": ["Use%", " 40%"],</text>
<text x="307.2" y="112" text-anchor="start" class="m">"stderr": "",</text>
<text x="307.2" y="131" text-anchor="start" class="m">"changed": false,</text>
<text x="307.2" y="150" text-anchor="start" class="m">"failed": false, ... }</text>
<path d="M257 48 L276 48" class="ar-amber"/>
<rect x="10" y="170" width="245" height="56" rx="7" class="k-purple"/>
<text x="132.5" y="193.5" text-anchor="middle" class="km-purple">set_fact: disk_used</text>
<text x="132.5" y="211.5" text-anchor="middle" class="km-purple">stdout_lines[1] | trim ...</text>
<path d="M420 172 C 420 200, 300 198, 258 198" class="ar-purple"/>
<rect x="590" y="200" width="120" height="56" rx="7" class="k-green"/>
<text x="650.0" y="223.5" text-anchor="middle" class="km-green">debug</text>
<text x="650.0" y="241.5" text-anchor="middle" class="ks-green">40% full</text>
<path d="M257 214 C 400 260, 500 230, 586 228" class="ar-green"/>
</g>
</svg>
<figcaption>A registered result is a dictionary; later tasks read its fields.</figcaption>
</figure>

- `disk` holds the whole result: `stdout`, `stdout_lines`, `stderr`, `rc` (the exit code), `changed`, `failed` and more. `ansible.builtin.debug: var=disk` shows all of it.
- `changed_when: false` tells Ansible that this command only reads, so the recap stays honest: without it, the task would always count as changed.
- `set_fact` creates a new variable for the host, here computed by a chain of filters: second line, spaces trimmed, `%` removed, converted to an integer.

## Magic variables: the whole inventory

Ansible defines some variables itself. These **magic variables** let a host see the rest of the inventory:

| Variable | Holds |
|---|---|
| `inventory_hostname` | this host's name in the inventory |
| `group_names` | the groups this host is in |
| `groups` | every group, with its hosts: `groups['web']` |
| `hostvars` | every host's variables and facts: `hostvars['db1']` |
| `play_hosts`, `ansible_play_hosts` | the hosts in the current play |
| `playbook_dir`, `inventory_dir` | where the playbook and inventory are |

```yaml
- name: Gather facts from every host
  hosts: all
  tasks: []

- name: What does each web server know about the others?
  hosts: web
  gather_facts: false
  tasks:
    - name: Magic variables
      ansible.builtin.debug:
        msg:
          - "I am {{ inventory_hostname }}, in {{ group_names | join(', ') }}"
          - "The web servers are {{ groups['web'] | join(' and ') }}"
          - "The database is at {{ hostvars['db1']['ansible_facts']['eth0']['ipv4']['address'] }}"
```

```
ok: [web1] => {
    "msg": [
        "I am web1, in linux, production, web",
        "The web servers are web1 and web2",
        "The database is at 10.0.0.21"
    ]
}
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="hostvars seen from web1: a table with one entry per host in the inventory. web1 reads its own variables directly, and db1&#x27;s facts through hostvars[&#x27;db1&#x27;], for example its eth0 address 10.0.0.21. groups[&#x27;web&#x27;] lists web1 and web2. db1&#x27;s facts are present only if they were gathered earlier in the run." xmlns="http://www.w3.org/2000/svg">
<style>
.a6f{ font-family:var(--sans),sans-serif; }
.a6f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a6f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a6f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a6f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a6f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a6f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a6f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a6f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a6f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a6f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a6f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a6f .s{ fill:var(--text-dim); font-size:13px; }
.a6f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a6f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a6f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a6f .c{ fill:var(--text-faint); font-size:12.5px; }
.a6f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a6f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a6f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a6f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6f-a); }
.a6f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a6f-a); marker-start:url(#a6f-s); }
.a6f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a6f-g); }
.a6f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a6f-a); }
.a6f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a6f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a6f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a6f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a6f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a6f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6f-tri); }
.a6f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6f-tri); }
.a6f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6f-open); }
.a6f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a6f-open); }
.a6f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6f-dia); }
.a6f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6f-odia); }
.a6f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a6f-dia); marker-end:url(#a6f-open); }
.a6f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a6f-solid); }
.a6f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a6f-open); }
.a6f .fill{ fill:var(--text); stroke:none; }
.a6f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a6f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a6f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a6f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a6f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a6f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a6f .ks-blue{ fill:#185FA5; font-size:13px; }
.a6f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a6f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a6f .kb-blue, :root[data-theme="dark"] .a6f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a6f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a6f .kh-blue{ fill:#85B7EB; }
.a6f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a6f-blue); }
.a6f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6f-blue); }
.a6f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a6f .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a6f .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a6f .ks-teal{ fill:#0F6E56; font-size:13px; }
.a6f .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a6f .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a6f .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a6f .kb-teal, :root[data-theme="dark"] .a6f .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a6f .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a6f .kh-teal{ fill:#5DCAA5; }
.a6f .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a6f-teal); }
.a6f .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a6f-teal); }
.a6f .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
</style>
<defs><marker id="a6f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a6f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a6f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a6f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a6f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a6f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a6f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a6f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a6f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a6f-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker></defs>
<g class="a6f">
<rect x="20" y="90" width="150" height="54" rx="7" class="k-blue"/>
<line x1="30" y1="99" x2="46" y2="99" class="ln-blue"/>
<line x1="30" y1="104" x2="46" y2="104" class="ln-blue"/>
<text x="103.0" y="112" text-anchor="middle" class="kb-blue">web1</text>
<text x="95.0" y="131" text-anchor="middle" class="ks-blue">running a task</text>
<text x="470" y="22" text-anchor="middle" class="b">hostvars</text>
<rect x="230" y="34" width="150" height="120" rx="8" class="pnl"/>
<text x="305" y="56" text-anchor="middle" class="m">['web1']</text>
<text x="305" y="82" text-anchor="middle" class="c">group_names: web</text>
<text x="305" y="104" text-anchor="middle" class="c">eth0: 10.0.0.11</text>
<text x="305" y="126" text-anchor="middle" class="c">... facts, vars</text>
<rect x="390" y="34" width="150" height="120" rx="8" class="pnl"/>
<text x="465" y="56" text-anchor="middle" class="m">['web2']</text>
<text x="465" y="82" text-anchor="middle" class="c">group_names: web</text>
<text x="465" y="104" text-anchor="middle" class="c">eth0: 10.0.0.12</text>
<text x="465" y="126" text-anchor="middle" class="c">... facts, vars</text>
<rect x="550" y="34" width="150" height="120" rx="8" class="k-teal"/>
<text x="625" y="56" text-anchor="middle" class="m">['db1']</text>
<text x="625" y="82" text-anchor="middle" class="c">group_names: db</text>
<text x="625" y="104" text-anchor="middle" class="c">eth0: 10.0.0.21</text>
<text x="625" y="126" text-anchor="middle" class="c">... facts, vars</text>
<path d="M172 110 C 300 190, 600 200, 620 158" class="ar-teal"/>
<text x="470" y="200" text-anchor="middle" class="km-teal">hostvars['db1']['ansible_facts']['eth0']['ipv4']['address']</text>
<text x="470" y="228" text-anchor="middle" class="c">needs db1's facts: gather them in an earlier play</text>
</g>
</svg>
<figcaption>Every host can read every other host's variables and facts.</figcaption>
</figure>

The first play exists only to gather facts from every host, including db1, which is not in the second play. Without it, `hostvars['db1']['ansible_facts']` would be empty and the task would fail. This is how a web server's configuration can contain the database's address without anyone typing it: Chapters 7, 12 and 14 build load balancers and firewalls this way.

## Variables on the command line, and prompts

```
printf 'site_name: Northwind Coffee (holiday hours)\n' > holiday.yml
ansible-playbook site.yml -e @holiday.yml --check --diff     # extra variables from a file
ansible-playbook facts.yml -e '{"owners": ["Carol"]}'        # JSON, for lists and dictionaries
```

A play can also ask for a value when it runs, with `vars_prompt`; useful for a release number or a confirmation, but it stops automation, so give such values with `-e` in pipelines.

## Naming variables

- Use lowercase and underscores: `http_port`, not `httpPort` or `http-port` (a hyphen is subtraction in Jinja2).
- Don't reuse names Ansible reserves: anything starting with `ansible_`, and words like `environment`, `name`, `hosts`, `tags`.
- Prefix variables that belong to a role with the role's name (`webserver_port`, Chapter 10).
- Give every variable a **default** in one place, so a playbook never fails with *undefined variable*. `{{ x | default('fallback') }}` provides one inline.

## Hands on

1. Use `ansible_facts['os_family']` in a message that says which package manager each host would use.
2. web2's `http_port` is set in `host_vars/web2.yml`. Add `http_port: 81` to `group_vars/all.yml`. Which value does web2 get? web1? Why?
3. `ansible web1 -m debug -a 'var=hostvars[inventory_hostname]' | head -40`: what does Ansible know about web1?
4. Add a `vars_prompt` to a play asking for a release number, and print it.
5. What is the difference between `inventory_hostname` and `ansible_facts['hostname']`? When could they differ?

## Summary

- **Variables** make one playbook serve many hosts; use them with `{{ name }}`, and quote values that start with `{{`.
- They come from the inventory and `group_vars`/`host_vars` (the usual place), play `vars` and `vars_files`, roles, facts, `register`, `set_fact`, and `-e`, which always wins.
- Keep each variable in one place, and precedence rarely matters.
- **Facts** describe each host (`ansible_facts['...']`); in DockerLab, use `eth0` for the lab address.
- `register` stores a task's result; `changed_when: false` keeps read-only commands honest.
- **Magic variables** (`inventory_hostname`, `group_names`, `groups`, `hostvars`) give a host a view of the whole inventory, provided the other hosts' facts were gathered.
