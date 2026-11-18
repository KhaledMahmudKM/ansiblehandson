# Chapter 8: Conditions, Loops and Errors

Northwind hires. Alice joins the operations team and needs an admin account on every server; Bob, a web developer, needs one on the web servers only; Carol, the database administrator, only on db1. Dave has left and must disappear from everywhere. A menu page should come from a menu server, when that server is up. And the playbook should refuse to run on a host it was never written for. Real playbooks make decisions, repeat work and recover from failures. This chapter adds **conditions**, **loops**, **retries** and **error handling** to `~/northwind`.

> **Try it:** DockerLab, *Ansible lab 7, Conditionals, loops and errors*. Same topology. The finished project is in `examples/ch08`.

## when: run a task only where it applies

`when` runs a task only on hosts where its condition is true; elsewhere it is **skipped**. The condition is a Jinja2 expression **without** braces:

```yaml
    - name: Web tools, on web servers only
      ansible.builtin.apt:
        name: curl
      when: "'web' in group_names"

    - name: Is there an old configuration file?
      ansible.builtin.stat:
        path: /etc/legacy-app.conf
      register: legacy

    - name: Warn about it
      ansible.builtin.debug:
        msg: "{{ inventory_hostname }} still has /etc/legacy-app.conf"
      when: legacy.stat.exists

    - name: Debian family, version 12 or newer
      ansible.builtin.debug:
        msg: "{{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_major_version'] }}"
      when:
        - ansible_facts['os_family'] == 'Debian'
        - ansible_facts['distribution_major_version'] | int >= 12
```

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="when evaluated per host. The task Web tools has the condition &#x27;web&#x27; in group_names. On web1 and web2 the condition is true and the task runs; on db1 it is false and the task is skipped." xmlns="http://www.w3.org/2000/svg">
<style>
.a8a{ font-family:var(--sans),sans-serif; }
.a8a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a8a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a8a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a8a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a8a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a8a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a8a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a8a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a8a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a8a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a8a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a8a .s{ fill:var(--text-dim); font-size:13px; }
.a8a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a8a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a8a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a8a .c{ fill:var(--text-faint); font-size:12.5px; }
.a8a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a8a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a8a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a8a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8a-a); }
.a8a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8a-a); marker-start:url(#a8a-s); }
.a8a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a8a-g); }
.a8a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a8a-a); }
.a8a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a8a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a8a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a8a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a8a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a8a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8a-tri); }
.a8a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8a-tri); }
.a8a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8a-open); }
.a8a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8a-open); }
.a8a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8a-dia); }
.a8a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8a-odia); }
.a8a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8a-dia); marker-end:url(#a8a-open); }
.a8a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8a-solid); }
.a8a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a8a-open); }
.a8a .fill{ fill:var(--text); stroke:none; }
.a8a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a8a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a8a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a8a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a8a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a8a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a8a .ks-purple{ fill:#534AB7; font-size:13px; }
.a8a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a8a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a8a .kb-purple, :root[data-theme="dark"] .a8a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a8a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a8a .kh-purple{ fill:#AFA9EC; }
.a8a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a8a-purple); }
.a8a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8a-purple); }
.a8a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a8a .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a8a .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a8a .ks-green{ fill:#3B6D11; font-size:13px; }
.a8a .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a8a .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8a .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a8a .kb-green, :root[data-theme="dark"] .a8a .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a8a .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a8a .kh-green{ fill:#97C459; }
.a8a .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a8a-green); }
.a8a .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8a-green); }
.a8a .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a8a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a8a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a8a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a8a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a8a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a8a .kb-gray, :root[data-theme="dark"] .a8a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a8a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a8a .kh-gray{ fill:#B4B2A9; }
.a8a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a8a-gray); }
.a8a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8a-gray); }
.a8a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a8a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a8a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a8a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a8a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a8a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a8a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a8a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a8a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a8a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a8a-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a8a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a8a">
<rect x="20" y="80" width="230" height="70" rx="7" class="k-purple"/>
<text x="135.0" y="110.5" text-anchor="middle" class="kb-purple">task: Web tools</text>
<text x="135.0" y="128.5" text-anchor="middle" class="km-purple">when: 'web' in group_names</text>
<path d="M252 115 L330 45" class="ag"/>
<rect x="334" y="20" width="120" height="50" rx="7" class="pnl"/>
<text x="394.0" y="40.5" text-anchor="middle" class="b">web1</text>
<text x="394.0" y="58.5" text-anchor="middle" class="m">true</text>
<path d="M456 45 L496 45" class="ar-green"/>
<rect x="500" y="20" width="200" height="50" rx="7" class="k-green"/>
<text x="600.0" y="49.5" text-anchor="middle" class="kb-green">runs: changed or ok</text>
<path d="M252 115 L330 115" class="ag"/>
<rect x="334" y="90" width="120" height="50" rx="7" class="pnl"/>
<text x="394.0" y="110.5" text-anchor="middle" class="b">web2</text>
<text x="394.0" y="128.5" text-anchor="middle" class="m">true</text>
<path d="M456 115 L496 115" class="ar-green"/>
<rect x="500" y="90" width="200" height="50" rx="7" class="k-green"/>
<text x="600.0" y="119.5" text-anchor="middle" class="kb-green">runs: changed or ok</text>
<path d="M252 115 L330 185" class="ag"/>
<rect x="334" y="160" width="120" height="50" rx="7" class="pnl"/>
<text x="394.0" y="180.5" text-anchor="middle" class="b">db1</text>
<text x="394.0" y="198.5" text-anchor="middle" class="m">false</text>
<path d="M456 185 L496 185" class="ag"/>
<rect x="500" y="160" width="200" height="50" rx="7" class="k-gray"/>
<text x="600.0" y="189.5" text-anchor="middle" class="kb-gray">skipping</text>
</g>
</svg>
<figcaption>The same task, decided host by host.</figcaption>
</figure>

A condition can test group membership, a fact, a variable, or a **registered result**, as with `legacy.stat.exists` (the `stat` module only looks). A list of conditions must **all** be true. Use `or`, `and` and `not` inside one condition for anything else. Facts are often strings, so `| int` turns `"13"` into a number before comparing. Tests such as `is defined`, `is failed`, `is changed` and `is succeeded` read naturally: `when: result is changed`.

## loop: the team's accounts

The team is **data**, a list of dictionaries in `group_vars/all.yml`:

```yaml
team:
  - name: alice
    comment: Alice Admin
    groups: [sudo]
  - name: bob
    comment: Bob Webdev
    only_on: web
  - name: carol
    comment: Carol DBA
    only_on: db
  - name: dave
    comment: Dave (left the company)
    state: absent
```

and one task manages all of it:

```yaml
---
- name: The team's accounts, on every server
  hosts: all
  become: true
  tasks:
    - name: One account per team member
      ansible.builtin.user:
        name: "{{ item.name }}"
        comment: "{{ item.comment }}"
        groups: "{{ item.groups | default([]) }}"
        append: true
        shell: /bin/bash
        state: "{{ item.state | default('present') }}"
        remove: true
      loop: "{{ team }}"
      loop_control:
        label: "{{ item.name }}"
      when: item.only_on is not defined or item.only_on in group_names
```

```
ansible-playbook users.yml
```

```
TASK [One account per team member] *********************************************
changed: [web1] => (item=alice)
changed: [db1] => (item=alice)
skipping: [db1] => (item=bob)
changed: [web1] => (item=bob)
skipping: [web1] => (item=carol)
changed: [db1] => (item=carol)
ok: [web1] => (item=dave)
...
```

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The team loop as a grid of hosts and team members. alice: present on web1, web2 and db1, in sudo. bob: only on the web servers, skipped on db1. carol: only on db1, skipped on web1 and web2. dave: absent everywhere; an account found by hand is removed." xmlns="http://www.w3.org/2000/svg">
<style>
.a8b{ font-family:var(--sans),sans-serif; }
.a8b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a8b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a8b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a8b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a8b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a8b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a8b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a8b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a8b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a8b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a8b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a8b .s{ fill:var(--text-dim); font-size:13px; }
.a8b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a8b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a8b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a8b .c{ fill:var(--text-faint); font-size:12.5px; }
.a8b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a8b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a8b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a8b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8b-a); }
.a8b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8b-a); marker-start:url(#a8b-s); }
.a8b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a8b-g); }
.a8b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a8b-a); }
.a8b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a8b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a8b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a8b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a8b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a8b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8b-tri); }
.a8b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8b-tri); }
.a8b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8b-open); }
.a8b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8b-open); }
.a8b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8b-dia); }
.a8b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8b-odia); }
.a8b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8b-dia); marker-end:url(#a8b-open); }
.a8b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8b-solid); }
.a8b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a8b-open); }
.a8b .fill{ fill:var(--text); stroke:none; }
.a8b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a8b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a8b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a8b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a8b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a8b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a8b .ks-purple{ fill:#534AB7; font-size:13px; }
.a8b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a8b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a8b .kb-purple, :root[data-theme="dark"] .a8b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a8b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a8b .kh-purple{ fill:#AFA9EC; }
.a8b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a8b-purple); }
.a8b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8b-purple); }
.a8b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a8b .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a8b .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a8b .ks-green{ fill:#3B6D11; font-size:13px; }
.a8b .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a8b .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8b .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a8b .kb-green, :root[data-theme="dark"] .a8b .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a8b .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a8b .kh-green{ fill:#97C459; }
.a8b .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a8b-green); }
.a8b .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8b-green); }
.a8b .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a8b .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a8b .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a8b .ks-red{ fill:#A32D2D; font-size:13px; }
.a8b .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a8b .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8b .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a8b .kb-red, :root[data-theme="dark"] .a8b .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a8b .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a8b .kh-red{ fill:#F09595; }
.a8b .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a8b-red); }
.a8b .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8b-red); }
.a8b .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a8b .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a8b .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a8b .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a8b .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a8b .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8b .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a8b .kb-gray, :root[data-theme="dark"] .a8b .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a8b .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a8b .kh-gray{ fill:#B4B2A9; }
.a8b .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a8b-gray); }
.a8b .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8b-gray); }
.a8b .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a8b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a8b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a8b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a8b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a8b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a8b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a8b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a8b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a8b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a8b-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a8b-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a8b-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a8b">
<text x="330" y="30" text-anchor="middle" class="b">web1</text>
<text x="460" y="30" text-anchor="middle" class="b">web2</text>
<text x="590" y="30" text-anchor="middle" class="b">db1</text>
<rect x="20" y="46" width="220" height="42" rx="7" class="k-purple"/>
<text x="130.0" y="63.5" text-anchor="middle" class="kb-purple">alice</text>
<text x="130.0" y="79.5" text-anchor="middle" class="ks-purple">Alice Admin, sudo</text>
<rect x="275" y="50" width="110" height="34" rx="5" class="k-green"/>
<text x="330.0" y="71.5" text-anchor="middle" class="kb-green">present</text>
<rect x="405" y="50" width="110" height="34" rx="5" class="k-green"/>
<text x="460.0" y="71.5" text-anchor="middle" class="kb-green">present</text>
<rect x="535" y="50" width="110" height="34" rx="5" class="k-green"/>
<text x="590.0" y="71.5" text-anchor="middle" class="kb-green">present</text>
<rect x="20" y="98" width="220" height="42" rx="7" class="k-purple"/>
<text x="130.0" y="115.5" text-anchor="middle" class="kb-purple">bob</text>
<text x="130.0" y="131.5" text-anchor="middle" class="ks-purple">only_on: web</text>
<rect x="275" y="102" width="110" height="34" rx="5" class="k-green"/>
<text x="330.0" y="123.5" text-anchor="middle" class="kb-green">present</text>
<rect x="405" y="102" width="110" height="34" rx="5" class="k-green"/>
<text x="460.0" y="123.5" text-anchor="middle" class="kb-green">present</text>
<rect x="535" y="102" width="110" height="34" rx="5" class="k-gray"/>
<text x="590.0" y="123.5" text-anchor="middle" class="kb-gray">skipped</text>
<rect x="20" y="150" width="220" height="42" rx="7" class="k-purple"/>
<text x="130.0" y="167.5" text-anchor="middle" class="kb-purple">carol</text>
<text x="130.0" y="183.5" text-anchor="middle" class="ks-purple">only_on: db</text>
<rect x="275" y="154" width="110" height="34" rx="5" class="k-gray"/>
<text x="330.0" y="175.5" text-anchor="middle" class="kb-gray">skipped</text>
<rect x="405" y="154" width="110" height="34" rx="5" class="k-gray"/>
<text x="460.0" y="175.5" text-anchor="middle" class="kb-gray">skipped</text>
<rect x="535" y="154" width="110" height="34" rx="5" class="k-green"/>
<text x="590.0" y="175.5" text-anchor="middle" class="kb-green">present</text>
<rect x="20" y="202" width="220" height="42" rx="7" class="k-red"/>
<text x="130.0" y="219.5" text-anchor="middle" class="kb-red">dave</text>
<text x="130.0" y="235.5" text-anchor="middle" class="ks-red">state: absent</text>
<rect x="275" y="206" width="110" height="34" rx="5" class="k-red"/>
<text x="330.0" y="227.5" text-anchor="middle" class="kb-red">removed</text>
<rect x="405" y="206" width="110" height="34" rx="5" class="k-red"/>
<text x="460.0" y="227.5" text-anchor="middle" class="kb-red">removed</text>
<rect x="535" y="206" width="110" height="34" rx="5" class="k-red"/>
<text x="590.0" y="227.5" text-anchor="middle" class="kb-red">removed</text>
<text x="360" y="262" text-anchor="middle" class="c">one list in group_vars/all.yml; one task; when checked for every item</text>
</g>
</svg>
<figcaption>Data in, accounts out: the loop applies the team list to every host.</figcaption>
</figure>

`loop` runs the task once per item, with the current item in `item`. `when` is evaluated **for each item**, so bob is skipped on db1 and carol on the web servers. `default(...)` fills in what an entry leaves out, and `loop_control: label` shows just the name in the output instead of the whole dictionary.

Now the leaver. Suppose someone recreated dave on web1 by hand:

```
ssh web1 'sudo useradd -m dave'
ansible-playbook users.yml --limit web1 | grep dave
```

```
changed: [web1] => (item=dave)
```

The list says dave is `absent`, so the next run removes him, and with `remove: true` his home folder too. Onboarding and offboarding become one-line edits to `team`, applied to every server at once, and reviewable in Git.

## Loops over dictionaries, and over lists of things

`loop` takes a list. A dictionary becomes one with `dict2items`, which turns each entry into `{key: ..., value: ...}`:

```yaml
  vars:
    folders:
      /srv/www: "0755"
      /srv/backup: "0700"
      /srv/uploads: "0770"
  tasks:
    - name: Make each folder
      ansible.builtin.file:
        path: "{{ item.key }}"
        mode: "{{ item.value }}"
        state: directory
      loop: "{{ folders | dict2items }}"
      loop_control:
        label: "{{ item.key }} ({{ item.value }})"
```

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="dict2items turns a dictionary of folders and modes into a list of items, each with a key and a value: key /srv/www value 0755, key /srv/backup value 0700, key /srv/uploads value 0770. The loop then reads item.key and item.value." xmlns="http://www.w3.org/2000/svg">
<style>
.a8c{ font-family:var(--sans),sans-serif; }
.a8c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a8c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a8c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a8c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a8c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a8c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a8c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a8c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a8c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a8c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a8c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a8c .s{ fill:var(--text-dim); font-size:13px; }
.a8c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a8c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a8c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a8c .c{ fill:var(--text-faint); font-size:12.5px; }
.a8c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a8c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a8c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a8c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8c-a); }
.a8c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8c-a); marker-start:url(#a8c-s); }
.a8c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a8c-g); }
.a8c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a8c-a); }
.a8c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a8c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a8c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a8c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a8c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a8c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8c-tri); }
.a8c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8c-tri); }
.a8c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8c-open); }
.a8c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8c-open); }
.a8c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8c-dia); }
.a8c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8c-odia); }
.a8c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8c-dia); marker-end:url(#a8c-open); }
.a8c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8c-solid); }
.a8c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a8c-open); }
.a8c .fill{ fill:var(--text); stroke:none; }
.a8c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a8c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a8c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a8c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a8c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a8c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a8c .ks-blue{ fill:#185FA5; font-size:13px; }
.a8c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a8c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a8c .kb-blue, :root[data-theme="dark"] .a8c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a8c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a8c .kh-blue{ fill:#85B7EB; }
.a8c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a8c-blue); }
.a8c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8c-blue); }
.a8c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a8c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a8c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a8c .ks-amber{ fill:#854F0B; font-size:13px; }
.a8c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a8c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a8c .kb-amber, :root[data-theme="dark"] .a8c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a8c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a8c .kh-amber{ fill:#EF9F27; }
.a8c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a8c-amber); }
.a8c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8c-amber); }
.a8c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
</style>
<defs><marker id="a8c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a8c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a8c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a8c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a8c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a8c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a8c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a8c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a8c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a8c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker></defs>
<g class="a8c">
<rect x="20" y="30" width="230" height="110" rx="6" class="k-amber"/>
<text x="30" y="50" text-anchor="start" class="kh-amber">a dictionary</text>
<text x="32.0" y="72" text-anchor="start" class="m">folders:</text>
<text x="47.2" y="90" text-anchor="start" class="m">/srv/www: "0755"</text>
<text x="47.2" y="108" text-anchor="start" class="m">/srv/backup: "0700"</text>
<text x="47.2" y="126" text-anchor="start" class="m">/srv/uploads: "0770"</text>
<path d="M254 90 L300 90" class="ar-amber"/>
<text x="277" y="124" text-anchor="middle" class="kh-amber">dict2items</text>
<rect x="306" y="30" width="390" height="92" rx="6" class="k-blue"/>
<text x="316" y="50" text-anchor="start" class="kh-blue">a list, for loop</text>
<text x="318.0" y="72" text-anchor="start" class="m">- { key: /srv/www,     value: "0755" }</text>
<text x="318.0" y="90" text-anchor="start" class="m">- { key: /srv/backup,  value: "0700" }</text>
<text x="318.0" y="108" text-anchor="start" class="m">- { key: /srv/uploads, value: "0770" }</text>
<text x="360" y="190" text-anchor="middle" class="m">path: "{{ item.key }}"     mode: "{{ item.value }}"</text>
</g>
</svg>
<figcaption>Loops take lists; dict2items makes one from a dictionary.</figcaption>
</figure>

Other useful loop sources: `query('fileglob', 'files/*.conf')` (files on control), `range(1, 4)`, `groups['web']`, and `subelements`, which pairs each item with each element of one of its lists (Chapter 14 uses it for firewall rules). Inside `loop_control`, `index_var: i` numbers the items and `pause: 2` waits between them.

> Don't loop where a module takes a list. `apt: name: [tree, rsync, jq]` installs three packages in **one** transaction; a loop would call apt three times.

## Waiting and retrying

Some things take time: a service starting, a page answering, a cluster forming. `until` repeats a task until its condition is true, up to `retries` times, `delay` seconds apart. Northwind's web play now ends by waiting for the site to answer, checked from control:

```yaml
    - name: Wait until the site answers
      ansible.builtin.uri:
        url: "http://{{ inventory_hostname }}:{{ http_port }}/"
      register: page
      until: page.status == 200
      retries: 5
      delay: 2
      delegate_to: localhost
      become: false
```

<figure class="diagram">
<svg viewBox="0 0 720 210" style="width:100%;height:auto;max-width:720px" role="img" aria-label="until with retries and delay over time. Attempt one at time 0 gets no answer, attempt two after a 2 second delay gets no answer, attempt three after another 2 seconds gets status 200 and the task succeeds. If all five retries failed, the task would fail." xmlns="http://www.w3.org/2000/svg">
<style>
.a8d{ font-family:var(--sans),sans-serif; }
.a8d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a8d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a8d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a8d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a8d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a8d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a8d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a8d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a8d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a8d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a8d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a8d .s{ fill:var(--text-dim); font-size:13px; }
.a8d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a8d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a8d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a8d .c{ fill:var(--text-faint); font-size:12.5px; }
.a8d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a8d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a8d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a8d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8d-a); }
.a8d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8d-a); marker-start:url(#a8d-s); }
.a8d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a8d-g); }
.a8d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a8d-a); }
.a8d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a8d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a8d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a8d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a8d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a8d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8d-tri); }
.a8d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8d-tri); }
.a8d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8d-open); }
.a8d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8d-open); }
.a8d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8d-dia); }
.a8d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8d-odia); }
.a8d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8d-dia); marker-end:url(#a8d-open); }
.a8d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8d-solid); }
.a8d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a8d-open); }
.a8d .fill{ fill:var(--text); stroke:none; }
.a8d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a8d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a8d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a8d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a8d .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a8d .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a8d .ks-amber{ fill:#854F0B; font-size:13px; }
.a8d .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a8d .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8d .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a8d .kb-amber, :root[data-theme="dark"] .a8d .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a8d .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a8d .kh-amber{ fill:#EF9F27; }
.a8d .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a8d-amber); }
.a8d .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8d-amber); }
.a8d .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a8d .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a8d .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a8d .ks-green{ fill:#3B6D11; font-size:13px; }
.a8d .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a8d .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8d .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a8d .kb-green, :root[data-theme="dark"] .a8d .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a8d .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a8d .kh-green{ fill:#97C459; }
.a8d .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a8d-green); }
.a8d .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8d-green); }
.a8d .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
</style>
<defs><marker id="a8d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a8d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a8d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a8d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a8d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a8d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a8d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a8d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a8d-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a8d-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker></defs>
<g class="a8d">
<rect x="20" y="50" width="130" height="60" rx="7" class="k-amber"/>
<text x="85.0" y="75.5" text-anchor="middle" class="kb-amber">attempt 1</text>
<text x="85.0" y="93.5" text-anchor="middle" class="ks-amber">no answer</text>
<text x="85" y="140" text-anchor="middle" class="c">t = 0 s</text>
<path d="M152 80 L196 80" class="ag"/>
<text x="174" y="70" text-anchor="middle" class="c">2 s</text>
<rect x="200" y="50" width="130" height="60" rx="7" class="k-amber"/>
<text x="265.0" y="75.5" text-anchor="middle" class="kb-amber">attempt 2</text>
<text x="265.0" y="93.5" text-anchor="middle" class="ks-amber">no answer</text>
<text x="265" y="140" text-anchor="middle" class="c">t = 2 s</text>
<path d="M332 80 L376 80" class="ag"/>
<text x="354" y="70" text-anchor="middle" class="c">2 s</text>
<rect x="380" y="50" width="130" height="60" rx="7" class="k-green"/>
<text x="445.0" y="75.5" text-anchor="middle" class="kb-green">attempt 3</text>
<text x="445.0" y="93.5" text-anchor="middle" class="ks-green">status 200</text>
<text x="445" y="140" text-anchor="middle" class="c">t = 4 s</text>
<rect x="570" y="50" width="130" height="60" rx="7" class="k-green"/>
<text x="635.0" y="75.5" text-anchor="middle" class="kb-green">ok</text>
<text x="635.0" y="93.5" text-anchor="middle" class="ks-green">go on</text>
<path d="M512 80 L566 80" class="ar-green"/>
<text x="360" y="186" text-anchor="middle" class="c">FAILED - RETRYING ... (4 retries left); only when retries run out does the task fail</text>
</g>
</svg>
<figcaption>Retry until a condition holds, or give up after a limit.</figcaption>
</figure>

Each failed attempt prints *FAILED - RETRYING* with the attempts left; only when all are used up does the task fail. The `wait_for` module does the common cases on its own: wait for a port to open (or close), a file to appear, or a line in a log (`search_regex`).

## Deciding what counts as failed, or changed

Ansible judges a command by its exit code: 0 is success, anything else a failure. That is not always right. `grep -c` exits with **1** when it counts zero matches, which for a log check is good news:

```yaml
    - name: Count the errors
      ansible.builtin.command: grep -c ERROR /tmp/app.log
      register: errors
      changed_when: false
      failed_when: errors.rc > 1

    - name: Fail the host if the log has errors
      ansible.builtin.fail:
        msg: "{{ errors.stdout }} errors in the log"
      when: errors.stdout | int > 0
```

```
fatal: [web2]: FAILED! => {"changed": false, "msg": "2 errors in the log"}
web1   : ok=2    changed=1    unreachable=0    failed=0    skipped=1 ...
web2   : ok=2    changed=1    unreachable=0    failed=1    skipped=0 ...
```

<figure class="diagram">
<svg viewBox="0 0 720 210" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Ansible&#x27;s default judgment of a command compared with failed_when errors.rc greater than 1. grep -c exit code 0 means matches found: ok in both. Exit code 1 means no matches: Ansible&#x27;s default calls it failed, failed_when calls it ok. Exit code 2 means an error such as a missing file: failed in both." xmlns="http://www.w3.org/2000/svg">
<style>
.a8e{ font-family:var(--sans),sans-serif; }
.a8e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a8e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a8e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a8e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a8e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a8e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a8e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a8e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a8e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a8e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a8e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a8e .s{ fill:var(--text-dim); font-size:13px; }
.a8e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a8e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a8e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a8e .c{ fill:var(--text-faint); font-size:12.5px; }
.a8e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a8e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a8e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a8e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8e-a); }
.a8e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8e-a); marker-start:url(#a8e-s); }
.a8e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a8e-g); }
.a8e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a8e-a); }
.a8e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a8e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a8e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a8e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a8e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a8e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8e-tri); }
.a8e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8e-tri); }
.a8e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8e-open); }
.a8e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8e-open); }
.a8e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8e-dia); }
.a8e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8e-odia); }
.a8e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8e-dia); marker-end:url(#a8e-open); }
.a8e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8e-solid); }
.a8e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a8e-open); }
.a8e .fill{ fill:var(--text); stroke:none; }
.a8e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a8e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a8e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a8e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a8e .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a8e .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a8e .ks-green{ fill:#3B6D11; font-size:13px; }
.a8e .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a8e .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8e .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a8e .kb-green, :root[data-theme="dark"] .a8e .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a8e .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a8e .kh-green{ fill:#97C459; }
.a8e .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a8e-green); }
.a8e .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8e-green); }
.a8e .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a8e .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a8e .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a8e .ks-red{ fill:#A32D2D; font-size:13px; }
.a8e .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a8e .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8e .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a8e .kb-red, :root[data-theme="dark"] .a8e .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a8e .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a8e .kh-red{ fill:#F09595; }
.a8e .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a8e-red); }
.a8e .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8e-red); }
.a8e .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a8e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a8e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a8e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a8e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a8e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a8e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a8e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a8e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a8e-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a8e-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a8e">
<text x="120" y="30" text-anchor="middle" class="m">grep -c ERROR app.log</text>
<text x="380" y="30" text-anchor="middle" class="b">default rule (rc != 0)</text>
<text x="600" y="30" text-anchor="middle" class="b">failed_when: rc &gt; 1</text>
<rect x="20" y="50" width="200" height="38" rx="5" class="pnl"/>
<text x="120.0" y="73.5" text-anchor="middle" class="s">rc 0: errors found</text>
<rect x="300" y="50" width="160" height="38" rx="5" class="k-green"/>
<text x="380.0" y="73.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="520" y="50" width="160" height="38" rx="5" class="k-green"/>
<text x="600.0" y="73.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="20" y="98" width="200" height="38" rx="5" class="pnl"/>
<text x="120.0" y="121.5" text-anchor="middle" class="s">rc 1: no errors</text>
<rect x="300" y="98" width="160" height="38" rx="5" class="k-red"/>
<text x="380.0" y="121.5" text-anchor="middle" class="kb-red">failed</text>
<rect x="520" y="98" width="160" height="38" rx="5" class="k-green"/>
<text x="600.0" y="121.5" text-anchor="middle" class="kb-green">ok</text>
<rect x="20" y="146" width="200" height="38" rx="5" class="pnl"/>
<text x="120.0" y="169.5" text-anchor="middle" class="s">rc 2: file missing</text>
<rect x="300" y="146" width="160" height="38" rx="5" class="k-red"/>
<text x="380.0" y="169.5" text-anchor="middle" class="kb-red">failed</text>
<rect x="520" y="146" width="160" height="38" rx="5" class="k-red"/>
<text x="600.0" y="169.5" text-anchor="middle" class="kb-red">failed</text>
</g>
</svg>
<figcaption>You decide what a command's exit code means.</figcaption>
</figure>

- `failed_when` replaces Ansible's rule with yours: only a real error (exit code 2, file missing) fails.
- `changed_when: false` says this command never changes anything; `changed_when: "'created' in result.stdout"` would say when it does.
- `fail` stops a host on purpose, with a message.
- `ignore_errors: true` lets the play carry on past a failed task (shown as *...ignoring* and counted as `ignored`). Use it sparingly: a hidden failure is still a failure.

## block, rescue and always

A **block** groups tasks. If any of them fails, its **rescue** tasks run instead of stopping the host; its **always** tasks run whatever happened. It is *try, catch, finally* for playbooks:

```yaml
- name: Today's menu, from the menu server if it's up
  hosts: web
  become: true
  gather_facts: false
  tasks:
    - name: Get the menu
      block:
        - name: Download it from the menu server
          ansible.builtin.get_url:
            url: http://menu.northwind.lab/menu.html
            dest: "{{ web_root }}/menu.html"
            timeout: 5
            mode: "0644"
      rescue:
        - name: The menu server is down; publish a fallback page
          ansible.builtin.copy:
            dest: "{{ web_root }}/menu.html"
            content: "<h1>Menu</h1><p>Ask at the counter today.</p>\n"
            mode: "0644"
      always:
        - name: Either way, record what happened
          ansible.builtin.debug:
            msg: "{{ inventory_hostname }}: menu {{ 'downloaded' if ansible_failed_task is not defined else 'replaced by the fallback' }}"
```

```
"msg": "web1: menu replaced by the fallback"
web1   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
```

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="block, rescue and always as a flow. The tasks in the block run first. If all succeed, rescue is skipped. If one fails, the rest of the block is skipped and the rescue tasks run. Either way, the always tasks run last. A failure handled by rescue counts as rescued, not failed." xmlns="http://www.w3.org/2000/svg">
<style>
.a8f{ font-family:var(--sans),sans-serif; }
.a8f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a8f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a8f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a8f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a8f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a8f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a8f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a8f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a8f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a8f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a8f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a8f .s{ fill:var(--text-dim); font-size:13px; }
.a8f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a8f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .c{ fill:var(--text-faint); font-size:12.5px; }
.a8f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a8f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a8f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a8f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8f-a); }
.a8f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8f-a); marker-start:url(#a8f-s); }
.a8f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a8f-g); }
.a8f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-a); }
.a8f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a8f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a8f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a8f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a8f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a8f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8f-tri); }
.a8f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-tri); }
.a8f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8f-open); }
.a8f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-open); }
.a8f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8f-dia); }
.a8f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8f-odia); }
.a8f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8f-dia); marker-end:url(#a8f-open); }
.a8f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8f-solid); }
.a8f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a8f-open); }
.a8f .fill{ fill:var(--text); stroke:none; }
.a8f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a8f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a8f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a8f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a8f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a8f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a8f .ks-blue{ fill:#185FA5; font-size:13px; }
.a8f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a8f .kb-blue, :root[data-theme="dark"] .a8f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a8f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a8f .kh-blue{ fill:#85B7EB; }
.a8f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a8f-blue); }
.a8f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-blue); }
.a8f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a8f .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a8f .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a8f .ks-teal{ fill:#0F6E56; font-size:13px; }
.a8f .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8f .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a8f .kb-teal, :root[data-theme="dark"] .a8f .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a8f .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a8f .kh-teal{ fill:#5DCAA5; }
.a8f .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a8f-teal); }
.a8f .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-teal); }
.a8f .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a8f .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a8f .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a8f .ks-amber{ fill:#854F0B; font-size:13px; }
.a8f .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8f .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a8f .kb-amber, :root[data-theme="dark"] .a8f .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a8f .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a8f .kh-amber{ fill:#EF9F27; }
.a8f .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a8f-amber); }
.a8f .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-amber); }
.a8f .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a8f .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a8f .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a8f .ks-coral{ fill:#993C1D; font-size:13px; }
.a8f .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8f .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a8f .kb-coral, :root[data-theme="dark"] .a8f .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a8f .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a8f .kh-coral{ fill:#F0997B; }
.a8f .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a8f-coral); }
.a8f .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-coral); }
.a8f .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a8f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a8f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a8f .ks-green{ fill:#3B6D11; font-size:13px; }
.a8f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a8f .kb-green, :root[data-theme="dark"] .a8f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a8f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a8f .kh-green{ fill:#97C459; }
.a8f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a8f-green); }
.a8f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-green); }
.a8f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a8f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a8f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a8f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a8f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a8f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a8f .kb-gray, :root[data-theme="dark"] .a8f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a8f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a8f .kh-gray{ fill:#B4B2A9; }
.a8f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a8f-gray); }
.a8f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8f-gray); }
.a8f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a8f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a8f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a8f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a8f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a8f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a8f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a8f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a8f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a8f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a8f-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a8f-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a8f-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a8f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a8f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a8f">
<rect x="20" y="100" width="160" height="70" rx="7" class="k-blue"/>
<text x="100.0" y="130.5" text-anchor="middle" class="km-blue">block:</text>
<text x="100.0" y="148.5" text-anchor="middle" class="ks-blue">download menu</text>
<path d="M250 105 L280 135 L250 165 L220 135 z" class="k-amber"/>
<text x="250" y="139.5" text-anchor="middle" class="kb-amber">ok?</text>
<path d="M182 135 L218 135" class="ar"/>
<rect x="330" y="20" width="180" height="60" rx="7" class="k-gray"/>
<text x="420.0" y="54.5" text-anchor="middle" class="kb-gray">skip rescue</text>
<rect x="330" y="190" width="180" height="70" rx="7" class="k-coral"/>
<text x="420.0" y="212.5" text-anchor="middle" class="km-coral">rescue:</text>
<text x="420.0" y="229.5" text-anchor="middle" class="ks-coral">publish fallback</text>
<text x="420.0" y="246.5" text-anchor="middle" class="ks-coral">counts: rescued=1</text>
<rect x="560" y="100" width="140" height="70" rx="7" class="k-teal"/>
<text x="630.0" y="130.5" text-anchor="middle" class="km-teal">always:</text>
<text x="630.0" y="148.5" text-anchor="middle" class="ks-teal">record it</text>
<path d="M266 112 L326 54" class="ar-green"/>
<text x="286" y="72" text-anchor="middle" class="kh-green">yes</text>
<path d="M266 158 L326 214" class="ar-coral"/>
<text x="286" y="206" text-anchor="middle" class="kh-coral">no</text>
<path d="M512 50 L556 120" class="ar"/>
<path d="M512 224 L556 152" class="ar"/>
</g>
</svg>
<figcaption>Try, catch, finally: the host survives a handled failure.</figcaption>
</figure>

The menu server doesn't exist, so the download failed and the rescue published a fallback. The recap counts `rescued=1` and `failed=0`: the failure was handled. Inside `rescue`, `ansible_failed_task` and `ansible_failed_result` describe what went wrong. Typical uses: roll back a failed upgrade, restore a configuration backup, and take a server out of a load balancer, then **always** put it back (Chapter 12). A block also lets several tasks share one `when`, `become` or tag.

## assert: check assumptions first

A playbook written for Debian servers with enough memory should refuse anything else, **before** changing anything. Northwind's `site.yml` now starts with a preflight play:

```yaml
- name: Preflight checks
  hosts: all
  tasks:
    - name: Refuse to run on hosts this project doesn't support
      ansible.builtin.assert:
        that:
          - ansible_facts['os_family'] == 'Debian'
          - ansible_facts['memtotal_mb'] >= 256
          - http_port is not defined or (http_port | int > 0 and http_port | int < 65536)
        fail_msg: "{{ inventory_hostname }} is not a host this project supports"
        quiet: true
```

```
ansible-playbook site.yml -e http_port=70000
```

```
fatal: [web1]: FAILED! => {"assertion": "http_port is not defined or (http_port | int > 0 and http_port | int < 65536)",
  "changed": false, "evaluated_to": false, "msg": "web1 is not a host this project supports"}
```

<figure class="diagram">
<svg viewBox="0 0 720 200" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A preflight assert as a gate. Before any change, each host is checked: Debian family, at least 256 MB of memory, a valid http_port. Hosts that pass go on to the real plays; a host that fails is stopped with a clear message before anything is changed on it." xmlns="http://www.w3.org/2000/svg">
<style>
.a8g{ font-family:var(--sans),sans-serif; }
.a8g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a8g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a8g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a8g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a8g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a8g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a8g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a8g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a8g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a8g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a8g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a8g .s{ fill:var(--text-dim); font-size:13px; }
.a8g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a8g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a8g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a8g .c{ fill:var(--text-faint); font-size:12.5px; }
.a8g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a8g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a8g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a8g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8g-a); }
.a8g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a8g-a); marker-start:url(#a8g-s); }
.a8g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a8g-g); }
.a8g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a8g-a); }
.a8g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a8g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a8g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a8g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a8g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a8g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8g-tri); }
.a8g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8g-tri); }
.a8g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8g-open); }
.a8g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a8g-open); }
.a8g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8g-dia); }
.a8g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8g-odia); }
.a8g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a8g-dia); marker-end:url(#a8g-open); }
.a8g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a8g-solid); }
.a8g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a8g-open); }
.a8g .fill{ fill:var(--text); stroke:none; }
.a8g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a8g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a8g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a8g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a8g .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a8g .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a8g .ks-blue{ fill:#185FA5; font-size:13px; }
.a8g .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a8g .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8g .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a8g .kb-blue, :root[data-theme="dark"] .a8g .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a8g .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a8g .kh-blue{ fill:#85B7EB; }
.a8g .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a8g-blue); }
.a8g .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8g-blue); }
.a8g .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a8g .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a8g .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a8g .ks-amber{ fill:#854F0B; font-size:13px; }
.a8g .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a8g .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8g .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a8g .kb-amber, :root[data-theme="dark"] .a8g .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a8g .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a8g .kh-amber{ fill:#EF9F27; }
.a8g .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a8g-amber); }
.a8g .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8g-amber); }
.a8g .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a8g .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a8g .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a8g .ks-green{ fill:#3B6D11; font-size:13px; }
.a8g .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a8g .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8g .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a8g .kb-green, :root[data-theme="dark"] .a8g .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a8g .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a8g .kh-green{ fill:#97C459; }
.a8g .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a8g-green); }
.a8g .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8g-green); }
.a8g .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a8g .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a8g .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a8g .ks-red{ fill:#A32D2D; font-size:13px; }
.a8g .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a8g .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a8g .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a8g .kb-red, :root[data-theme="dark"] .a8g .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a8g .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a8g .kh-red{ fill:#F09595; }
.a8g .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a8g-red); }
.a8g .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a8g-red); }
.a8g .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a8g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a8g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a8g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a8g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a8g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a8g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a8g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a8g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a8g-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a8g-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a8g-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a8g-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a8g">
<rect x="20" y="16" width="110" height="40" rx="7" class="k-blue"/>
<line x1="30" y1="25" x2="46" y2="25" class="ln-blue"/>
<line x1="30" y1="30" x2="46" y2="30" class="ln-blue"/>
<text x="83.0" y="38" text-anchor="middle" class="kb-blue">web1</text>
<path d="M132 36 L226 100" class="ag"/>
<rect x="20" y="76" width="110" height="40" rx="7" class="k-blue"/>
<line x1="30" y1="85" x2="46" y2="85" class="ln-blue"/>
<line x1="30" y1="90" x2="46" y2="90" class="ln-blue"/>
<text x="83.0" y="98" text-anchor="middle" class="kb-blue">web2</text>
<path d="M132 96 L226 100" class="ag"/>
<rect x="20" y="136" width="110" height="40" rx="7" class="k-blue"/>
<line x1="30" y1="145" x2="46" y2="145" class="ln-blue"/>
<line x1="30" y1="150" x2="46" y2="150" class="ln-blue"/>
<text x="83.0" y="158" text-anchor="middle" class="kb-blue">db1</text>
<path d="M132 156 L226 100" class="ag"/>
<rect x="230" y="50" width="220" height="100" rx="7" class="k-amber"/>
<text x="340.0" y="76.0" text-anchor="middle" class="kb-amber">preflight: assert</text>
<text x="340.0" y="95.0" text-anchor="middle" class="km-amber">os_family == Debian</text>
<text x="340.0" y="114.0" text-anchor="middle" class="km-amber">memtotal_mb &gt;= 256</text>
<text x="340.0" y="133.0" text-anchor="middle" class="km-amber">0 &lt; http_port &lt; 65536</text>
<rect x="500" y="30" width="200" height="50" rx="7" class="k-green"/>
<text x="600.0" y="59.5" text-anchor="middle" class="kb-green">pass: real plays run</text>
<rect x="500" y="120" width="200" height="50" rx="7" class="k-red"/>
<text x="600.0" y="149.5" text-anchor="middle" class="kb-red">fail: stop, nothing changed</text>
<path d="M452 80 L496 56" class="ar-green"/>
<path d="M452 120 L496 144" class="ar-red"/>
</g>
</svg>
<figcaption>Fail early, with a clear message, before anything changes.</figcaption>
</figure>

A failed assertion stops the host with a clear message, which is far better than failing halfway through a change with a half-configured server. Other useful controls: `any_errors_fatal: true` on a play stops **every** host as soon as one fails, and `max_fail_percentage` stops when too many fail (Chapter 12).

## Hands on

1. Give each team member an SSH public key in `team`, and install it in the same loop with `ansible.posix.authorized_key`.
2. Number the team in a message with `loop_control: index_var: i`.
3. Make the rescue in `menu.yml` also append a line to a log file on control (`delegate_to: localhost`, `lineinfile`).
4. Add `any_errors_fatal: true` to the log check play. What happens to web1 now?
5. Explain the difference between `ignore_errors` and `block` with `rescue`.

## Summary

- `when` runs a task only where its condition is true; conditions use facts, groups, variables and registered results, without braces.
- `loop` repeats a task over a list; `when` is checked per item; `dict2items` turns a dictionary into a list; `loop_control: label` keeps output readable. Prefer modules that take lists.
- `until` with `retries` and `delay` waits for something to become true; `wait_for` waits for ports and files.
- `failed_when` and `changed_when` define what failure and change mean; `fail` stops a host on purpose; `ignore_errors` carries on.
- `block`, `rescue` and `always` handle failures like try, catch and finally.
- `assert` checks assumptions before anything changes.
