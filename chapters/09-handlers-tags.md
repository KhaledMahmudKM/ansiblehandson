# Chapter 9: Handlers and Tags

Northwind's staging server, web2, was moved to port 8080 two chapters ago. The configuration file says 8080, and `ansible-playbook` reported *changed*. Yet `curl http://web2:8080` gets nothing, because nginx is still running with the configuration it read when it started. Something must tell nginx to reload. Doing it on every run would interrupt users for no reason; doing it by hand defeats the point of automation. **Handlers** do it only when the configuration actually changed. And as `site.yml` grows, **tags** let you run just the part you need.

> **Try it:** DockerLab, *Ansible lab 8, Handlers and tags*. Same topology. The finished project is in `examples/ch09`.

## notify and handlers

A **handler** is a task that runs only when another task **notifies** it, and only if that task reported **changed**:

```yaml
    - name: Site configuration
      ansible.builtin.template:
        src: templates/site.conf.j2
        dest: /etc/nginx/sites-available/default
        mode: "0644"
      notify: Reload nginx

  handlers:
    - name: Reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Tasks and handlers in a play. Task Site configuration reports changed and notifies Reload nginx; task Hide the version reports changed and notifies it too; task Home page reports changed but notifies nothing. At the end of the play the handler Reload nginx runs once." xmlns="http://www.w3.org/2000/svg">
<style>
.a9a{ font-family:var(--sans),sans-serif; }
.a9a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a9a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a9a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a9a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a9a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a9a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a9a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a9a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a9a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a9a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a9a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a9a .s{ fill:var(--text-dim); font-size:13px; }
.a9a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a9a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a9a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a9a .c{ fill:var(--text-faint); font-size:12.5px; }
.a9a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a9a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a9a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a9a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9a-a); }
.a9a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9a-a); marker-start:url(#a9a-s); }
.a9a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a9a-g); }
.a9a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a9a-a); }
.a9a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a9a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a9a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a9a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a9a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a9a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9a-tri); }
.a9a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9a-tri); }
.a9a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9a-open); }
.a9a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9a-open); }
.a9a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9a-dia); }
.a9a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9a-odia); }
.a9a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9a-dia); marker-end:url(#a9a-open); }
.a9a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9a-solid); }
.a9a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a9a-open); }
.a9a .fill{ fill:var(--text); stroke:none; }
.a9a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a9a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a9a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a9a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a9a .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a9a .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a9a .ks-blue{ fill:#185FA5; font-size:13px; }
.a9a .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a9a .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9a .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a9a .kb-blue, :root[data-theme="dark"] .a9a .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a9a .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a9a .kh-blue{ fill:#85B7EB; }
.a9a .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a9a-blue); }
.a9a .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9a-blue); }
.a9a .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a9a .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a9a .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a9a .ks-coral{ fill:#993C1D; font-size:13px; }
.a9a .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a9a .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9a .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a9a .kb-coral, :root[data-theme="dark"] .a9a .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a9a .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a9a .kh-coral{ fill:#F0997B; }
.a9a .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a9a-coral); }
.a9a .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9a-coral); }
.a9a .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
</style>
<defs><marker id="a9a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a9a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a9a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a9a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a9a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a9a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a9a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a9a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a9a-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a9a-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker></defs>
<g class="a9a">
<rect x="20" y="16" width="220" height="46" rx="7" class="k-blue"/>
<text x="130.0" y="34.5" text-anchor="middle" class="kb-blue">Site configuration</text>
<text x="130.0" y="52.5" text-anchor="middle" class="ks-blue">changed</text>
<path d="M242 37 C 330 37, 360 120, 438 120" class="ad-coral"/>
<rect x="20" y="68" width="220" height="46" rx="7" class="k-blue"/>
<text x="130.0" y="86.5" text-anchor="middle" class="kb-blue">Hide the version</text>
<text x="130.0" y="104.5" text-anchor="middle" class="ks-blue">changed</text>
<path d="M242 89 C 330 89, 360 120, 438 120" class="ad-coral"/>
<rect x="20" y="120" width="220" height="46" rx="7" class="k-blue"/>
<text x="130.0" y="138.5" text-anchor="middle" class="kb-blue">Home page</text>
<text x="130.0" y="156.5" text-anchor="middle" class="ks-blue">changed</text>
<rect x="20" y="172" width="220" height="46" rx="7" class="pnl"/>
<text x="130.0" y="190.5" text-anchor="middle" class="b">Start nginx</text>
<text x="130.0" y="208.5" text-anchor="middle" class="c">ok</text>
<text x="330" y="92" text-anchor="middle" class="kh-coral">notify</text>
<rect x="440" y="70" width="120" height="100" rx="10" class="k-coral"/>
<text x="500" y="96" text-anchor="middle" class="kb-coral">queue</text>
<text x="500" y="122" text-anchor="middle" class="ks-coral">Reload nginx</text>
<text x="500" y="142" text-anchor="middle" class="c">(once)</text>
<rect x="590" y="90" width="110" height="60" rx="7" class="k-coral"/>
<text x="645.0" y="115.5" text-anchor="middle" class="kb-coral">end of play</text>
<text x="645.0" y="133.5" text-anchor="middle" class="ks-coral">handler runs</text>
<path d="M562 120 L586 120" class="ar-coral"/>
<text x="130" y="232" text-anchor="middle" class="c">tasks run in order</text>
</g>
</svg>
<figcaption>Notifications queue up; each handler runs once, at the end of the play.</figcaption>
</figure>

Handlers live in their own `handlers:` section of the play and are found by **name**. Notified handlers run **once**, at the **end of the play**, after all its tasks, no matter how many tasks notified them. If no task changed anything, no handler runs.

## No change, no restart

nginx writes its process number (PID) to `/run/nginx.pid`. Watching it shows exactly when nginx restarts:

```
ansible web -a 'cat /run/nginx.pid'
ansible-playbook site.yml | grep -E 'RUNNING HANDLER|^(web|db)'
ansible web -a 'cat /run/nginx.pid'
```

With nothing changed, nothing is notified: no handler, the same PIDs, and users are not interrupted. A new home page needs no reload either, since nginx reads pages from disk on every request, so the home page task doesn't notify. Now change web2's port in `host_vars/web2.yml` to 8080 and run again:

```
TASK [Site configuration] ******************************************************
changed: [web2]

RUNNING HANDLER [Check the nginx configuration] ********************************
ok: [web2]

RUNNING HANDLER [Reload nginx for real] ****************************************
changed: [web2]
```

```
curl -s -o /dev/null -w '%{http_code}\n' http://web2:8080/
200
```

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="nginx&#x27;s PID on web2 across three runs. Run 1: nothing changed, no handler, PID 847. Run 2: http_port changed to 8080, the site task changed and notified, the handler reloaded nginx, and port 8080 now answers. Run 3: nothing changed, no handler." xmlns="http://www.w3.org/2000/svg">
<style>
.a9b{ font-family:var(--sans),sans-serif; }
.a9b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a9b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a9b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a9b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a9b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a9b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a9b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a9b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a9b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a9b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a9b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a9b .s{ fill:var(--text-dim); font-size:13px; }
.a9b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a9b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a9b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a9b .c{ fill:var(--text-faint); font-size:12.5px; }
.a9b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a9b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a9b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a9b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9b-a); }
.a9b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9b-a); marker-start:url(#a9b-s); }
.a9b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a9b-g); }
.a9b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a9b-a); }
.a9b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a9b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a9b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a9b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a9b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a9b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9b-tri); }
.a9b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9b-tri); }
.a9b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9b-open); }
.a9b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9b-open); }
.a9b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9b-dia); }
.a9b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9b-odia); }
.a9b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9b-dia); marker-end:url(#a9b-open); }
.a9b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9b-solid); }
.a9b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a9b-open); }
.a9b .fill{ fill:var(--text); stroke:none; }
.a9b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a9b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a9b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a9b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a9b .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a9b .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a9b .ks-teal{ fill:#0F6E56; font-size:13px; }
.a9b .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a9b .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9b .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a9b .kb-teal, :root[data-theme="dark"] .a9b .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a9b .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a9b .kh-teal{ fill:#5DCAA5; }
.a9b .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a9b-teal); }
.a9b .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9b-teal); }
.a9b .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a9b .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a9b .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a9b .ks-coral{ fill:#993C1D; font-size:13px; }
.a9b .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a9b .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9b .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a9b .kb-coral, :root[data-theme="dark"] .a9b .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a9b .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a9b .kh-coral{ fill:#F0997B; }
.a9b .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a9b-coral); }
.a9b .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9b-coral); }
.a9b .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a9b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a9b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a9b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a9b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a9b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a9b .kb-gray, :root[data-theme="dark"] .a9b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a9b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a9b .kh-gray{ fill:#B4B2A9; }
.a9b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a9b-gray); }
.a9b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9b-gray); }
.a9b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a9b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a9b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a9b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a9b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a9b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a9b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a9b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a9b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a9b-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a9b-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a9b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a9b">
<rect x="20" y="30" width="216" height="90" rx="7" class="k-gray"/>
<text x="128.0" y="61.5" text-anchor="middle" class="kb-gray">run 1</text>
<text x="128.0" y="79.5" text-anchor="middle" class="ks-gray">nothing changed</text>
<text x="128.0" y="97.5" text-anchor="middle" class="ks-gray">no handler</text>
<rect x="252" y="30" width="216" height="90" rx="7" class="k-coral"/>
<text x="360.0" y="61.5" text-anchor="middle" class="kb-coral">run 2: port 8080</text>
<text x="360.0" y="79.5" text-anchor="middle" class="ks-coral">Site configuration: changed</text>
<text x="360.0" y="97.5" text-anchor="middle" class="ks-coral">Reload nginx runs</text>
<rect x="484" y="30" width="216" height="90" rx="7" class="k-gray"/>
<text x="592.0" y="61.5" text-anchor="middle" class="kb-gray">run 3</text>
<text x="592.0" y="79.5" text-anchor="middle" class="ks-gray">nothing changed</text>
<text x="592.0" y="97.5" text-anchor="middle" class="ks-gray">no handler</text>
<line x1="20" y1="160" x2="700" y2="160" class="ln"/>
<text x="128" y="186" text-anchor="middle" class="s">listens on 80</text>
<text x="360" y="186" text-anchor="middle" class="s">listens on 8080: curl gives 200</text>
<text x="592" y="186" text-anchor="middle" class="s">still 8080, untouched</text>
<text x="360" y="210" text-anchor="middle" class="kh-teal">users are interrupted only when something actually changed</text>
</g>
</svg>
<figcaption>Handlers turn 'changed' into exactly one reload.</figcaption>
</figure>

Only web2's configuration changed, so only web2's nginx reloaded, and now it answers on 8080. web1 was not touched. Without the handler, the file would say 8080 while nginx still listened on 80: the classic *I changed it but nothing happened*.

## Reload or restart

| `state:` | What nginx does | Users notice? |
|---|---|---|
| `reloaded` | the master process rereads its configuration and replaces its workers gracefully; same PID | no: open connections finish |
| `restarted` | stop, then start: a new PID | yes: connections drop, briefly |

Prefer **reload** when the service supports it and the change allows it. Some changes need a restart (a new module, some listen-socket changes), and some services can only restart. The PID test above shows which happened: after a reload, the master process keeps its PID.

## Several tasks, one handler, two steps

Northwind's web play now has two tasks that touch nginx's configuration: the site file, and a line in `nginx.conf` hiding the version number in error pages. Both notify the same **topic**, and two handlers **listen** to it:

```yaml
    - name: Hide the nginx version in error pages
      ansible.builtin.lineinfile:
        path: /etc/nginx/nginx.conf
        regexp: '^\s*#?\s*server_tokens'
        line: '        server_tokens off;'
      notify: Reload nginx

  handlers:
    - name: Check the nginx configuration
      ansible.builtin.command: nginx -t
      changed_when: false
      listen: Reload nginx

    - name: Reload nginx for real
      ansible.builtin.service:
        name: nginx
        state: reloaded
      listen: Reload nginx
```

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Two tasks notify the topic Reload nginx. Two handlers listen to that topic: Check the nginx configuration, which runs nginx -t, and Reload nginx for real. They run once each, in the order written; if the check fails, the reload never happens." xmlns="http://www.w3.org/2000/svg">
<style>
.a9c{ font-family:var(--sans),sans-serif; }
.a9c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a9c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a9c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a9c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a9c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a9c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a9c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a9c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a9c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a9c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a9c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a9c .s{ fill:var(--text-dim); font-size:13px; }
.a9c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a9c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a9c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a9c .c{ fill:var(--text-faint); font-size:12.5px; }
.a9c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a9c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a9c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a9c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9c-a); }
.a9c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9c-a); marker-start:url(#a9c-s); }
.a9c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a9c-g); }
.a9c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a9c-a); }
.a9c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a9c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a9c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a9c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a9c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a9c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9c-tri); }
.a9c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9c-tri); }
.a9c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9c-open); }
.a9c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9c-open); }
.a9c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9c-dia); }
.a9c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9c-odia); }
.a9c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9c-dia); marker-end:url(#a9c-open); }
.a9c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9c-solid); }
.a9c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a9c-open); }
.a9c .fill{ fill:var(--text); stroke:none; }
.a9c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a9c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a9c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a9c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a9c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a9c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a9c .ks-blue{ fill:#185FA5; font-size:13px; }
.a9c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a9c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a9c .kb-blue, :root[data-theme="dark"] .a9c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a9c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a9c .kh-blue{ fill:#85B7EB; }
.a9c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a9c-blue); }
.a9c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9c-blue); }
.a9c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a9c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a9c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a9c .ks-amber{ fill:#854F0B; font-size:13px; }
.a9c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a9c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a9c .kb-amber, :root[data-theme="dark"] .a9c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a9c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a9c .kh-amber{ fill:#EF9F27; }
.a9c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a9c-amber); }
.a9c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9c-amber); }
.a9c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a9c .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a9c .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a9c .ks-coral{ fill:#993C1D; font-size:13px; }
.a9c .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a9c .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9c .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a9c .kb-coral, :root[data-theme="dark"] .a9c .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a9c .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a9c .kh-coral{ fill:#F0997B; }
.a9c .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a9c-coral); }
.a9c .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9c-coral); }
.a9c .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a9c .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a9c .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a9c .ks-green{ fill:#3B6D11; font-size:13px; }
.a9c .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a9c .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9c .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a9c .kb-green, :root[data-theme="dark"] .a9c .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a9c .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a9c .kh-green{ fill:#97C459; }
.a9c .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a9c-green); }
.a9c .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9c-green); }
.a9c .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
</style>
<defs><marker id="a9c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a9c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a9c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a9c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a9c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a9c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a9c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a9c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a9c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a9c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a9c-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a9c-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker></defs>
<g class="a9c">
<rect x="20" y="30" width="210" height="50" rx="7" class="k-blue"/>
<text x="125.0" y="59.5" text-anchor="middle" class="kb-blue">Site configuration</text>
<rect x="20" y="120" width="210" height="50" rx="7" class="k-blue"/>
<text x="125.0" y="149.5" text-anchor="middle" class="kb-blue">Hide the version</text>
<rect x="280" y="70" width="160" height="60" rx="7" class="k-coral"/>
<text x="360.0" y="95.5" text-anchor="middle" class="ks-coral">topic</text>
<text x="360.0" y="113.5" text-anchor="middle" class="kb-coral">Reload nginx</text>
<path d="M232 55 L276 92" class="ad-coral"/>
<path d="M232 145 L276 110" class="ad-coral"/>
<rect x="490" y="20" width="210" height="60" rx="7" class="k-amber"/>
<text x="595.0" y="45.5" text-anchor="middle" class="kb-amber">1. Check the configuration</text>
<text x="595.0" y="63.5" text-anchor="middle" class="km-amber">nginx -t</text>
<rect x="490" y="120" width="210" height="60" rx="7" class="k-green"/>
<text x="595.0" y="145.5" text-anchor="middle" class="kb-green">2. Reload nginx for real</text>
<text x="595.0" y="163.5" text-anchor="middle" class="km-green">service: reloaded</text>
<path d="M442 92 L486 54" class="ar-coral"/>
<path d="M442 110 L486 146" class="ar-coral"/>
<text x="595" y="104" text-anchor="middle" class="c">fails? reload never happens</text>
</g>
</svg>
<figcaption>One notification, several handlers, in order.</figcaption>
</figure>

`listen` lets several handlers answer one notification, in the order they are written. Here the first checks the whole configuration with `nginx -t`; if a template produced a broken file, the run fails **there**, before nginx is told to reload. Two tasks notified, but each handler runs once.

```
curl -sI http://web1/nothing | grep -i ^server
Server: nginx
```

## Handlers now: flush_handlers

The web play ends by checking that the site answers (Chapter 8). But if the port just changed, the check must run **after** the reload, and handlers normally wait for the end of the play. `meta: flush_handlers` runs any pending handlers right there:

```yaml
    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started

    - name: Apply any configuration change now, before testing
      ansible.builtin.meta: flush_handlers

    - name: Wait until the site answers
      ansible.builtin.uri:
        url: "http://{{ inventory_hostname }}:{{ http_port }}/"
      ...
```

<figure class="diagram">
<svg viewBox="0 0 720 180" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Without flush_handlers, the handler would run after the site check, so the check tests the old port and fails. With meta flush_handlers placed after the configuration tasks, the reload happens first, and the check sees the new configuration." xmlns="http://www.w3.org/2000/svg">
<style>
.a9d{ font-family:var(--sans),sans-serif; }
.a9d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a9d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a9d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a9d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a9d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a9d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a9d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a9d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a9d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a9d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a9d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a9d .s{ fill:var(--text-dim); font-size:13px; }
.a9d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a9d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a9d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a9d .c{ fill:var(--text-faint); font-size:12.5px; }
.a9d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a9d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a9d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a9d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9d-a); }
.a9d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9d-a); marker-start:url(#a9d-s); }
.a9d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a9d-g); }
.a9d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a9d-a); }
.a9d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a9d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a9d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a9d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a9d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a9d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9d-tri); }
.a9d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9d-tri); }
.a9d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9d-open); }
.a9d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9d-open); }
.a9d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9d-dia); }
.a9d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9d-odia); }
.a9d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9d-dia); marker-end:url(#a9d-open); }
.a9d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9d-solid); }
.a9d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a9d-open); }
.a9d .fill{ fill:var(--text); stroke:none; }
.a9d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a9d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a9d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a9d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a9d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a9d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a9d .ks-blue{ fill:#185FA5; font-size:13px; }
.a9d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a9d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a9d .kb-blue, :root[data-theme="dark"] .a9d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a9d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a9d .kh-blue{ fill:#85B7EB; }
.a9d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a9d-blue); }
.a9d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9d-blue); }
.a9d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a9d .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a9d .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a9d .ks-coral{ fill:#993C1D; font-size:13px; }
.a9d .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a9d .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9d .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a9d .kb-coral, :root[data-theme="dark"] .a9d .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a9d .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a9d .kh-coral{ fill:#F0997B; }
.a9d .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a9d-coral); }
.a9d .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9d-coral); }
.a9d .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a9d .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a9d .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a9d .ks-green{ fill:#3B6D11; font-size:13px; }
.a9d .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a9d .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9d .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a9d .kb-green, :root[data-theme="dark"] .a9d .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a9d .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a9d .kh-green{ fill:#97C459; }
.a9d .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a9d-green); }
.a9d .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9d-green); }
.a9d .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
</style>
<defs><marker id="a9d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a9d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a9d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a9d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a9d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a9d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a9d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a9d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a9d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a9d-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a9d-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker></defs>
<g class="a9d">
<rect x="20" y="50" width="128" height="50" rx="7" class="k-blue"/>
<text x="84.0" y="79.5" text-anchor="middle" class="kb-blue">config: changed</text>
<path d="M150 75 L158 75" class="ar"/>
<rect x="160" y="50" width="128" height="50" rx="7" class="pnl"/>
<text x="224.0" y="79.5" text-anchor="middle" class="b">start nginx</text>
<path d="M290 75 L298 75" class="ar"/>
<rect x="300" y="50" width="128" height="50" rx="7" class="k-coral"/>
<text x="364.0" y="79.5" text-anchor="middle" class="kb-coral">flush_handlers</text>
<path d="M430 75 L438 75" class="ar"/>
<rect x="440" y="50" width="128" height="50" rx="7" class="k-coral"/>
<text x="504.0" y="79.5" text-anchor="middle" class="kb-coral">reload (handler)</text>
<path d="M570 75 L578 75" class="ar"/>
<rect x="580" y="50" width="128" height="50" rx="7" class="k-green"/>
<text x="644.0" y="79.5" text-anchor="middle" class="kb-green">check the site</text>
<text x="360" y="30" text-anchor="middle" class="b">with meta: flush_handlers</text>
<text x="360" y="140" text-anchor="middle" class="c">without it, the reload would wait for the end of the play,</text>
<text x="360" y="158" text-anchor="middle" class="c">and the check would test the old configuration</text>
</g>
</svg>
<figcaption>Run pending handlers at the point where later tasks depend on them.</figcaption>
</figure>

## When a later task fails

Handlers run at the end of the play. If a task **after** the change fails on a host, the play stops for that host before its handlers run: the file is new, the service still runs the old configuration, and the next run won't notice, because the file is already right.

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The failure trap. A configuration task changes the file and notifies the handler, then a later task fails, so the play stops for that host before handlers run. The file says port 8081 but nginx still listens on the old port. The next run sees the file as already correct and does nothing. force_handlers runs the handler anyway." xmlns="http://www.w3.org/2000/svg">
<style>
.a9e{ font-family:var(--sans),sans-serif; }
.a9e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a9e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a9e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a9e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a9e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a9e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a9e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a9e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a9e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a9e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a9e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a9e .s{ fill:var(--text-dim); font-size:13px; }
.a9e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a9e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a9e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a9e .c{ fill:var(--text-faint); font-size:12.5px; }
.a9e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a9e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a9e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a9e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9e-a); }
.a9e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9e-a); marker-start:url(#a9e-s); }
.a9e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a9e-g); }
.a9e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-a); }
.a9e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a9e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a9e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a9e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a9e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a9e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9e-tri); }
.a9e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-tri); }
.a9e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9e-open); }
.a9e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-open); }
.a9e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9e-dia); }
.a9e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9e-odia); }
.a9e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9e-dia); marker-end:url(#a9e-open); }
.a9e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9e-solid); }
.a9e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a9e-open); }
.a9e .fill{ fill:var(--text); stroke:none; }
.a9e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a9e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a9e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a9e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a9e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a9e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a9e .ks-blue{ fill:#185FA5; font-size:13px; }
.a9e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a9e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a9e .kb-blue, :root[data-theme="dark"] .a9e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a9e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a9e .kh-blue{ fill:#85B7EB; }
.a9e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a9e-blue); }
.a9e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-blue); }
.a9e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a9e .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a9e .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a9e .ks-amber{ fill:#854F0B; font-size:13px; }
.a9e .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a9e .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9e .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a9e .kb-amber, :root[data-theme="dark"] .a9e .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a9e .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a9e .kh-amber{ fill:#EF9F27; }
.a9e .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a9e-amber); }
.a9e .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-amber); }
.a9e .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a9e .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a9e .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a9e .ks-green{ fill:#3B6D11; font-size:13px; }
.a9e .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a9e .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9e .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a9e .kb-green, :root[data-theme="dark"] .a9e .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a9e .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a9e .kh-green{ fill:#97C459; }
.a9e .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a9e-green); }
.a9e .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-green); }
.a9e .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a9e .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a9e .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a9e .ks-red{ fill:#A32D2D; font-size:13px; }
.a9e .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a9e .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9e .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a9e .kb-red, :root[data-theme="dark"] .a9e .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a9e .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a9e .kh-red{ fill:#F09595; }
.a9e .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a9e-red); }
.a9e .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-red); }
.a9e .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a9e .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a9e .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a9e .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a9e .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a9e .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9e .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a9e .kb-gray, :root[data-theme="dark"] .a9e .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a9e .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a9e .kh-gray{ fill:#B4B2A9; }
.a9e .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a9e-gray); }
.a9e .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9e-gray); }
.a9e .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a9e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a9e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a9e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a9e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a9e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a9e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a9e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a9e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a9e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a9e-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a9e-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a9e-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a9e-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a9e">
<rect x="20" y="30" width="180" height="60" rx="7" class="k-blue"/>
<text x="110.0" y="55.5" text-anchor="middle" class="kb-blue">config: changed</text>
<text x="110.0" y="73.5" text-anchor="middle" class="ks-blue">notify: Restart</text>
<rect x="240" y="30" width="180" height="60" rx="7" class="k-red"/>
<text x="330.0" y="55.5" text-anchor="middle" class="kb-red">later task</text>
<text x="330.0" y="73.5" text-anchor="middle" class="ks-red">FAILED</text>
<path d="M202 60 L236 60" class="ar"/>
<rect x="460" y="30" width="240" height="60" rx="7" class="k-gray"/>
<text x="580.0" y="55.5" text-anchor="middle" class="kb-gray">handler never runs</text>
<text x="580.0" y="73.5" text-anchor="middle" class="ks-gray">the play stopped for this host</text>
<path d="M422 60 L456 60" class="ag"/>
<rect x="20" y="130" width="330" height="70" rx="7" class="k-amber"/>
<text x="185.0" y="150.5" text-anchor="middle" class="km-amber">file: listen 8081</text>
<text x="185.0" y="169.5" text-anchor="middle" class="km-amber">nginx: still on 80</text>
<text x="185.0" y="188.5" text-anchor="middle" class="ks-amber">next run: file already right, no notify</text>
<rect x="380" y="130" width="320" height="70" rx="7" class="k-green"/>
<text x="540.0" y="150.5" text-anchor="middle" class="km-green">--force-handlers</text>
<text x="540.0" y="169.5" text-anchor="middle" class="km-green">or force_handlers = True</text>
<text x="540.0" y="188.5" text-anchor="middle" class="ks-green">handlers run even after a failure</text>
</g>
</svg>
<figcaption>Make sure a changed file and its running service can never disagree.</figcaption>
</figure>

Three defences:

- `--force-handlers` on the command line, or `force_handlers = True` in `ansible.cfg`: notified handlers run even after a failure.
- `meta: flush_handlers` right after the changes, as above, so the reload happens before anything that might fail.
- Fix it by hand once (`ansible web1 -b -m service -a 'name=nginx state=restarted'`) when it has already happened.

## Tags: run part of a playbook

`site.yml` now installs packages, writes configuration, publishes content and checks the result. Often you want just one of those. **Tags** label tasks, and `--tags` or `--skip-tags` choose among them:

```yaml
    - name: Install nginx
      ansible.builtin.apt: ...
      tags: packages

    - name: Site configuration
      ansible.builtin.template: ...
      tags: config

    - name: Home page
      ansible.builtin.template: ...
      tags: content

    - name: Start nginx
      ansible.builtin.service: ...
      tags: always
```

```
ansible-playbook site.yml --list-tags
ansible-playbook site.yml --tags content
ansible-playbook site.yml --skip-tags packages
```

```
playbook: site.yml
  play #1 (all): Preflight checks	TAGS: []
  play #2 (all): Every server	TAGS: []
  play #3 (web): Web servers	TAGS: []
      TASK TAGS: [always, config, content, packages]
  play #4 (db): Database server	TAGS: [db]
      TASK TAGS: [db]
```

<figure class="diagram">
<svg viewBox="0 0 720 280" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Which tasks run for each tag selection. Tasks: install nginx tagged packages, site configuration tagged config, home page tagged content, start nginx tagged always, check the site tagged always. No option: all run. --tags content: home page plus the always tasks. --skip-tags packages: everything except install nginx." xmlns="http://www.w3.org/2000/svg">
<style>
.a9f{ font-family:var(--sans),sans-serif; }
.a9f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a9f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a9f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a9f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a9f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a9f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a9f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a9f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a9f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a9f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a9f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a9f .s{ fill:var(--text-dim); font-size:13px; }
.a9f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a9f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a9f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a9f .c{ fill:var(--text-faint); font-size:12.5px; }
.a9f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a9f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a9f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a9f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9f-a); }
.a9f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a9f-a); marker-start:url(#a9f-s); }
.a9f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a9f-g); }
.a9f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a9f-a); }
.a9f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a9f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a9f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a9f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a9f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a9f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9f-tri); }
.a9f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9f-tri); }
.a9f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9f-open); }
.a9f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a9f-open); }
.a9f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9f-dia); }
.a9f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9f-odia); }
.a9f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a9f-dia); marker-end:url(#a9f-open); }
.a9f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a9f-solid); }
.a9f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a9f-open); }
.a9f .fill{ fill:var(--text); stroke:none; }
.a9f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a9f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a9f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a9f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a9f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a9f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a9f .ks-green{ fill:#3B6D11; font-size:13px; }
.a9f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a9f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a9f .kb-green, :root[data-theme="dark"] .a9f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a9f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a9f .kh-green{ fill:#97C459; }
.a9f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a9f-green); }
.a9f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9f-green); }
.a9f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a9f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a9f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a9f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a9f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a9f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a9f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a9f .kb-gray, :root[data-theme="dark"] .a9f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a9f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a9f .kh-gray{ fill:#B4B2A9; }
.a9f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a9f-gray); }
.a9f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a9f-gray); }
.a9f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a9f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a9f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a9f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a9f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a9f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a9f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a9f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a9f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a9f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a9f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a9f">
<text x="400" y="16" text-anchor="middle" class="c">no option</text>
<text x="400" y="35" text-anchor="middle" class="m"></text>
<text x="510" y="16" text-anchor="middle" class="m">--tags</text>
<text x="510" y="35" text-anchor="middle" class="m">content</text>
<text x="620" y="16" text-anchor="middle" class="m">--skip-tags</text>
<text x="620" y="35" text-anchor="middle" class="m">packages</text>
<rect x="20" y="46" width="230" height="34" rx="5" class="pnl"/>
<text x="135.0" y="67.5" text-anchor="middle" class="s">Install nginx</text>
<text x="262" y="68" text-anchor="start" class="ma">packages</text>
<rect x="355" y="48" width="90" height="30" rx="5" class="k-green"/>
<text x="400.0" y="67.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="465" y="48" width="90" height="30" rx="5" class="k-gray"/>
<text x="510.0" y="67.5" text-anchor="middle" class="kb-gray">skipped</text>
<rect x="575" y="48" width="90" height="30" rx="5" class="k-gray"/>
<text x="620.0" y="67.5" text-anchor="middle" class="kb-gray">skipped</text>
<rect x="20" y="88" width="230" height="34" rx="5" class="pnl"/>
<text x="135.0" y="109.5" text-anchor="middle" class="s">Site configuration</text>
<text x="262" y="110" text-anchor="start" class="ma">config</text>
<rect x="355" y="90" width="90" height="30" rx="5" class="k-green"/>
<text x="400.0" y="109.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="465" y="90" width="90" height="30" rx="5" class="k-gray"/>
<text x="510.0" y="109.5" text-anchor="middle" class="kb-gray">skipped</text>
<rect x="575" y="90" width="90" height="30" rx="5" class="k-green"/>
<text x="620.0" y="109.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="20" y="130" width="230" height="34" rx="5" class="pnl"/>
<text x="135.0" y="151.5" text-anchor="middle" class="s">Home page</text>
<text x="262" y="152" text-anchor="start" class="ma">content</text>
<rect x="355" y="132" width="90" height="30" rx="5" class="k-green"/>
<text x="400.0" y="151.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="465" y="132" width="90" height="30" rx="5" class="k-green"/>
<text x="510.0" y="151.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="575" y="132" width="90" height="30" rx="5" class="k-green"/>
<text x="620.0" y="151.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="20" y="172" width="230" height="34" rx="5" class="pnl"/>
<text x="135.0" y="193.5" text-anchor="middle" class="s">Start nginx</text>
<text x="262" y="194" text-anchor="start" class="ma">always</text>
<rect x="355" y="174" width="90" height="30" rx="5" class="k-green"/>
<text x="400.0" y="193.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="465" y="174" width="90" height="30" rx="5" class="k-green"/>
<text x="510.0" y="193.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="575" y="174" width="90" height="30" rx="5" class="k-green"/>
<text x="620.0" y="193.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="20" y="214" width="230" height="34" rx="5" class="pnl"/>
<text x="135.0" y="235.5" text-anchor="middle" class="s">Wait until the site answers</text>
<text x="262" y="236" text-anchor="start" class="ma">always</text>
<rect x="355" y="216" width="90" height="30" rx="5" class="k-green"/>
<text x="400.0" y="235.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="465" y="216" width="90" height="30" rx="5" class="k-green"/>
<text x="510.0" y="235.5" text-anchor="middle" class="kb-green">runs</text>
<rect x="575" y="216" width="90" height="30" rx="5" class="k-green"/>
<text x="620.0" y="235.5" text-anchor="middle" class="kb-green">runs</text>
<text x="360" y="270" text-anchor="middle" class="c">always runs unless skipped by name; never runs only when asked for by name</text>
</g>
</svg>
<figcaption>Tags choose tasks; always and never are special.</figcaption>
</figure>

- `--tags content` runs only tasks tagged *content*, plus those tagged **always** (here: start nginx, and the check that the site answers) and fact gathering.
- `--skip-tags packages` runs everything except the package installs: quicker when you know they're there.
- Tags on a **play**, a **block** or a **role** apply to every task inside. The database play has `tags: db`.
- A task tagged **never** runs only when one of its other tags is asked for by name: right for dangerous jobs such as `tags: [never, purge]`, run with `--tags purge`.

Tags are a convenience, not a design. If you need a tag to make a playbook safe, the playbook is doing too much; split it.

## The web play now

```yaml
- name: Web servers
  hosts: web
  become: true

  tasks:
    - name: Install nginx                       # tags: packages
    - name: Site configuration                  # notify: Reload nginx, tags: config
    - name: Home page                           # tags: content
    - name: A sudo rule for the deploy group    # validate: visudo
    - name: Hide the nginx version in error pages   # notify: Reload nginx
    - name: Start nginx                         # tags: always
    - name: Apply any configuration change now  # meta: flush_handlers
    - name: Wait until the site answers         # uri, until, tags: always

  handlers:
    - name: Check the nginx configuration       # listen: Reload nginx
    - name: Reload nginx for real               # listen: Reload nginx
```

## Hands on

1. Notify a handler from a handler: after reloading nginx, a handler that checks the site with `uri`.
2. Tag the whole database play and run just it with `--tags db`.
3. Add a task tagged `never, purge` that empties `/var/cache/nginx`, and run it with `--tags purge`.
4. What do `--start-at-task` and `--step` offer that tags don't?
5. Break the site template on purpose (`listen {{ http_port }}` without the semicolon) and run the playbook. Which handler stops the damage?

## Summary

- A task with `notify` triggers a **handler** only when it **changed** something; handlers run **once**, at the **end of the play**.
- Prefer `reloaded` to `restarted` where possible; watching the PID shows which happened.
- `listen` lets several handlers answer one topic: check the configuration, then reload.
- `meta: flush_handlers` runs pending handlers immediately; `--force-handlers` runs them even after a failure, so files and services never disagree.
- **Tags** select tasks: `--tags`, `--skip-tags`, `--list-tags`; `always` and `never` are special.
