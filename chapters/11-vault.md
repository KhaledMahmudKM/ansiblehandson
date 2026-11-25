# Chapter 11: Vault and Secrets

Northwind's web site is about to become a shop. The new *Orders* application on the web servers needs a database password and a token for the payment provider's API, and an `appadmin` account with a password for the developers. These are **secrets**. Written in plain `group_vars`, they would end up in Git, in every backup, on every screen the project is shown on, and on every laptop that clones it. **Ansible Vault** encrypts them, so the project can still be shared and versioned, and decrypts them only in memory while a playbook runs.

> **Try it:** DockerLab, *Ansible lab 10, Vault and secrets*. Same topology. The finished project is in `examples/ch11`; its vault password is `Lab-Vault-2026`.

## The problem

```
grep -rn -E 'password|token' group_vars/
group_vars/web.yml:12:appconfig_db_password: Tr0ub4dor&3
group_vars/web.yml:13:appconfig_api_token: sk-live-51Habcdef0123456789
```

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A plain-text secret in group_vars spreads everywhere the project goes: the Git server and its history, backups, colleagues&#x27; and contractors&#x27; laptops, screen shares and CI logs." xmlns="http://www.w3.org/2000/svg">
<style>
.a11a{ font-family:var(--sans),sans-serif; }
.a11a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a11a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a11a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a11a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a11a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a11a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a11a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a11a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a11a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a11a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a11a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a11a .s{ fill:var(--text-dim); font-size:13px; }
.a11a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a11a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a11a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a11a .c{ fill:var(--text-faint); font-size:12.5px; }
.a11a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a11a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a11a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a11a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11a-a); }
.a11a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11a-a); marker-start:url(#a11a-s); }
.a11a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a11a-g); }
.a11a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a11a-a); }
.a11a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a11a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a11a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a11a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a11a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a11a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11a-tri); }
.a11a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11a-tri); }
.a11a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11a-open); }
.a11a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11a-open); }
.a11a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11a-dia); }
.a11a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11a-odia); }
.a11a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11a-dia); marker-end:url(#a11a-open); }
.a11a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11a-solid); }
.a11a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a11a-open); }
.a11a .fill{ fill:var(--text); stroke:none; }
.a11a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a11a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a11a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a11a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a11a .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a11a .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a11a .ks-red{ fill:#A32D2D; font-size:13px; }
.a11a .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a11a .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11a .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a11a .kb-red, :root[data-theme="dark"] .a11a .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a11a .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a11a .kh-red{ fill:#F09595; }
.a11a .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a11a-red); }
.a11a .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11a-red); }
.a11a .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a11a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a11a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a11a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a11a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a11a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a11a .kb-gray, :root[data-theme="dark"] .a11a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a11a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a11a .kh-gray{ fill:#B4B2A9; }
.a11a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a11a-gray); }
.a11a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11a-gray); }
.a11a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a11a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a11a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a11a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a11a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a11a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a11a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a11a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a11a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a11a-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a11a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a11a">
<rect x="200" y="80" width="320" height="74" rx="6" class="k-red"/>
<text x="210" y="100" text-anchor="start" class="kh-red">group_vars/web.yml</text>
<text x="212.0" y="122" text-anchor="start" class="m">appconfig_db_password: Tr0ub4dor&amp;3</text>
<text x="212.0" y="140" text-anchor="start" class="m">appconfig_api_token: sk-live-51H...</text>
<rect x="20" y="20" width="160" height="56" rx="7" class="k-gray"/>
<text x="100.0" y="43.5" text-anchor="middle" class="kb-gray">Git server</text>
<text x="100.0" y="61.5" text-anchor="middle" class="ks-gray">and all its history</text>
<path d="M198 120 L180 48" class="ad-red"/>
<rect x="20" y="150" width="160" height="56" rx="7" class="k-gray"/>
<text x="100.0" y="173.5" text-anchor="middle" class="kb-gray">backups</text>
<text x="100.0" y="191.5" text-anchor="middle" class="ks-gray">kept for years</text>
<path d="M198 120 L180 178" class="ad-red"/>
<rect x="540" y="20" width="160" height="56" rx="7" class="k-gray"/>
<text x="620.0" y="43.5" text-anchor="middle" class="kb-gray">every clone</text>
<text x="620.0" y="61.5" text-anchor="middle" class="ks-gray">laptops, contractors</text>
<path d="M522 120 L540 48" class="ad-red"/>
<rect x="540" y="150" width="160" height="56" rx="7" class="k-gray"/>
<text x="620.0" y="173.5" text-anchor="middle" class="kb-gray">screens, CI logs</text>
<text x="620.0" y="191.5" text-anchor="middle" class="ks-gray">shared and recorded</text>
<path d="M522 120 L540 178" class="ad-red"/>
</g>
</svg>
<figcaption>A secret in plain text is a secret shared with everyone who ever touches the project.</figcaption>
</figure>

Anyone who can read the project can read the secrets: a colleague, a contractor, the Git server, a backup tape, a screen share. Deleting them later doesn't help, because Git keeps history. Secrets must be encrypted **in the files** from the start.

## Split variables from secrets

The usual convention keeps two files per group, in a **folder** named after the group:

- `group_vars/web/vars.yml`: ordinary values, readable by everyone, plus a **reference** to each secret;
- `group_vars/web/vault.yml`: the secrets themselves, each named with a `vault_` prefix, **encrypted**.

```
mkdir -p group_vars/web
mv group_vars/web.yml group_vars/web/vars.yml
cat >> group_vars/web/vars.yml <<'EOF'
appconfig_db_host: db1
appconfig_db_user: orders
appconfig_db_password: "{{ vault_db_password }}"
appconfig_api_token: "{{ vault_api_token }}"
appconfig_admin_password: "{{ vault_admin_password }}"
appconfig_admin_salt: "{{ vault_admin_salt }}"
EOF
cat > group_vars/web/vault.yml <<'EOF'
---
vault_db_password: Tr0ub4dor&3
vault_api_token: sk-live-51Habcdef0123456789
vault_admin_password: Correct-Horse-Battery-9
vault_admin_salt: nOrthWiNdSaLt26
EOF
ansible-vault encrypt group_vars/web/vault.yml
```

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The vars and vault split. group_vars/web/vars.yml is plain and readable: appconfig_db_password refers to vault_db_password. group_vars/web/vault.yml holds vault_db_password and the other secrets, encrypted with Ansible Vault. Ansible loads both files for web hosts and resolves the reference." xmlns="http://www.w3.org/2000/svg">
<style>
.a11b{ font-family:var(--sans),sans-serif; }
.a11b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a11b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a11b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a11b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a11b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a11b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a11b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a11b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a11b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a11b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a11b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a11b .s{ fill:var(--text-dim); font-size:13px; }
.a11b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a11b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a11b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a11b .c{ fill:var(--text-faint); font-size:12.5px; }
.a11b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a11b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a11b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a11b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11b-a); }
.a11b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11b-a); marker-start:url(#a11b-s); }
.a11b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a11b-g); }
.a11b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a11b-a); }
.a11b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a11b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a11b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a11b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a11b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a11b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11b-tri); }
.a11b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11b-tri); }
.a11b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11b-open); }
.a11b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11b-open); }
.a11b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11b-dia); }
.a11b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11b-odia); }
.a11b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11b-dia); marker-end:url(#a11b-open); }
.a11b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11b-solid); }
.a11b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a11b-open); }
.a11b .fill{ fill:var(--text); stroke:none; }
.a11b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a11b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a11b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a11b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a11b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a11b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a11b .ks-blue{ fill:#185FA5; font-size:13px; }
.a11b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a11b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a11b .kb-blue, :root[data-theme="dark"] .a11b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a11b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a11b .kh-blue{ fill:#85B7EB; }
.a11b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a11b-blue); }
.a11b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11b-blue); }
.a11b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a11b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a11b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a11b .ks-purple{ fill:#534AB7; font-size:13px; }
.a11b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a11b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a11b .kb-purple, :root[data-theme="dark"] .a11b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a11b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a11b .kh-purple{ fill:#AFA9EC; }
.a11b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a11b-purple); }
.a11b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11b-purple); }
.a11b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a11b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a11b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a11b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a11b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a11b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a11b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a11b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a11b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a11b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a11b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a11b">
<rect x="10" y="30" width="385" height="92" rx="6" class="k-blue"/>
<text x="20" y="50" text-anchor="start" class="kh-blue">group_vars/web/vars.yml  (plain)</text>
<text x="22.0" y="72" text-anchor="start" class="m">appconfig_db_user: orders</text>
<text x="22.0" y="90" text-anchor="start" class="m">appconfig_db_password: "{{ vault_db_password }}"</text>
<text x="22.0" y="108" text-anchor="start" class="m">appconfig_api_token: "{{ vault_api_token }}"</text>
<rect x="415" y="30" width="295" height="92" rx="6" class="k-purple"/>
<text x="425" y="50" text-anchor="start" class="kh-purple">group_vars/web/vault.yml  (encrypted)</text>
<text x="427.0" y="72" text-anchor="start" class="m">$ANSIBLE_VAULT;1.1;AES256</text>
<text x="427.0" y="90" text-anchor="start" class="m">35353430623038313165346432</text>
<text x="427.0" y="108" text-anchor="start" class="m">3539363537646561313061...</text>
<path d="M300 104 C 360 160, 420 160, 470 118" class="ad-purple"/>
<text x="380" y="170" text-anchor="middle" class="kh-purple">refers to</text>
<rect x="20" y="196" width="330" height="50" rx="7" class="pnl"/>
<text x="185.0" y="216.5" text-anchor="middle" class="s">anyone can see WHICH secrets exist,</text>
<text x="185.0" y="234.5" text-anchor="middle" class="s">and where they are used</text>
<rect x="400" y="196" width="300" height="50" rx="7" class="pnl"/>
<text x="550.0" y="216.5" text-anchor="middle" class="s">only the vault password</text>
<text x="550.0" y="234.5" text-anchor="middle" class="s">shows their values</text>
</g>
</svg>
<figcaption>References in the open, values under encryption.</figcaption>
</figure>

Ansible loads **every** file in `group_vars/web/`, so `vars.yml` finds the `vault_` values as if they were in one file. The point of the indirection: anyone can see **which** secrets exist and where they are used, with `grep`, in code review, in `git diff`, without being able to read them.

## What encryption looks like

`ansible-vault encrypt` asks for a **vault password** twice and encrypts the file in place with **AES-256**:

```
head -3 group_vars/web/vault.yml
$ANSIBLE_VAULT;1.1;AES256
35353430623038313165346432306561373036616136633432633266313866666135393566373334
3539363537646561313061343931363335653637666465660a623963303533366464306263393664
```

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="ansible-vault encrypt. The plain YAML file and the vault password go in; a key is derived from the password with PBKDF2 and a random salt; the content is encrypted with AES-256 and protected with an HMAC; the result is a text file starting with $ANSIBLE_VAULT;1.1;AES256 followed by hexadecimal ciphertext." xmlns="http://www.w3.org/2000/svg">
<style>
.a11c{ font-family:var(--sans),sans-serif; }
.a11c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a11c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a11c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a11c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a11c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a11c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a11c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a11c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a11c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a11c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a11c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a11c .s{ fill:var(--text-dim); font-size:13px; }
.a11c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a11c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a11c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a11c .c{ fill:var(--text-faint); font-size:12.5px; }
.a11c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a11c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a11c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a11c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11c-a); }
.a11c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11c-a); marker-start:url(#a11c-s); }
.a11c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a11c-g); }
.a11c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a11c-a); }
.a11c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a11c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a11c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a11c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a11c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a11c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11c-tri); }
.a11c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11c-tri); }
.a11c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11c-open); }
.a11c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11c-open); }
.a11c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11c-dia); }
.a11c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11c-odia); }
.a11c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11c-dia); marker-end:url(#a11c-open); }
.a11c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11c-solid); }
.a11c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a11c-open); }
.a11c .fill{ fill:var(--text); stroke:none; }
.a11c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a11c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a11c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a11c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a11c .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a11c .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a11c .ks-amber{ fill:#854F0B; font-size:13px; }
.a11c .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a11c .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11c .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a11c .kb-amber, :root[data-theme="dark"] .a11c .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a11c .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a11c .kh-amber{ fill:#EF9F27; }
.a11c .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a11c-amber); }
.a11c .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11c-amber); }
.a11c .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a11c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a11c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a11c .ks-purple{ fill:#534AB7; font-size:13px; }
.a11c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a11c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a11c .kb-purple, :root[data-theme="dark"] .a11c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a11c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a11c .kh-purple{ fill:#AFA9EC; }
.a11c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a11c-purple); }
.a11c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11c-purple); }
.a11c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a11c .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a11c .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a11c .ks-green{ fill:#3B6D11; font-size:13px; }
.a11c .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a11c .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11c .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a11c .kb-green, :root[data-theme="dark"] .a11c .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a11c .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a11c .kh-green{ fill:#97C459; }
.a11c .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a11c-green); }
.a11c .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11c-green); }
.a11c .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a11c .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a11c .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a11c .ks-red{ fill:#A32D2D; font-size:13px; }
.a11c .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a11c .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11c .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a11c .kb-red, :root[data-theme="dark"] .a11c .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a11c .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a11c .kh-red{ fill:#F09595; }
.a11c .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a11c-red); }
.a11c .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11c-red); }
.a11c .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a11c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a11c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a11c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a11c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a11c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a11c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a11c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a11c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a11c-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a11c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a11c-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a11c-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a11c">
<rect x="20" y="30" width="210" height="92" rx="6" class="k-red"/>
<text x="30" y="50" text-anchor="start" class="kh-red">vault.yml (plain)</text>
<text x="32.0" y="72" text-anchor="start" class="m">vault_db_password:</text>
<text x="47.2" y="90" text-anchor="start" class="m">Tr0ub4dor&amp;3</text>
<text x="32.0" y="108" text-anchor="start" class="m">vault_api_token: ...</text>
<rect x="20" y="150" width="210" height="46" rx="7" class="k-amber"/>
<text x="125.0" y="169.0" text-anchor="middle" class="kb-amber">vault password</text>
<text x="125.0" y="186.0" text-anchor="middle" class="km-amber">Lab-Vault-2026</text>
<rect x="280" y="70" width="170" height="90" rx="7" class="k-purple"/>
<text x="365.0" y="100.5" text-anchor="middle" class="kb-purple">ansible-vault</text>
<text x="365.0" y="119.5" text-anchor="middle" class="ks-purple">PBKDF2 key + salt</text>
<text x="365.0" y="138.5" text-anchor="middle" class="ks-purple">AES-256 + HMAC</text>
<path d="M232 80 L276 100" class="ar"/>
<path d="M232 172 L276 140" class="ar-amber"/>
<rect x="490" y="40" width="210" height="128" rx="6" class="k-green"/>
<text x="500" y="60" text-anchor="start" class="kh-green">vault.yml (encrypted)</text>
<text x="502.0" y="82" text-anchor="start" class="m">$ANSIBLE_VAULT;1.1;AES256</text>
<text x="502.0" y="100" text-anchor="start" class="m">3535343062303831</text>
<text x="502.0" y="118" text-anchor="start" class="m">3165346432306561</text>
<text x="502.0" y="136" text-anchor="start" class="m">3730366161366334</text>
<text x="502.0" y="154" text-anchor="start" class="m">...</text>
<path d="M452 115 L486 115" class="ar-green"/>
</g>
</svg>
<figcaption>The encrypted file is plain text again, but meaningless without the password.</figcaption>
</figure>

The header names the format and cipher; the rest is the ciphertext, in hexadecimal. The key is derived from your vault password (PBKDF2 with a random salt), and an HMAC protects the file against tampering. The encrypted file is safe to commit. It is only as strong as the vault password, so choose a long one.

## Working with encrypted files

| Command | Does |
|---|---|
| `ansible-vault create FILE` | create a new encrypted file in your editor |
| `ansible-vault encrypt FILE` | encrypt an existing file in place |
| `ansible-vault view FILE` | show the plain text |
| `ansible-vault edit FILE` | decrypt into your editor, re-encrypt on save; the plain text never touches the disk |
| `ansible-vault decrypt FILE` | decrypt in place (rarely what you want) |
| `ansible-vault rekey FILE` | re-encrypt with a new vault password |
| `ansible-vault encrypt_string VALUE --name VAR` | encrypt one value, to paste into a plain file |

```
ansible-vault view group_vars/web/vault.yml
ansible-vault edit group_vars/web/vault.yml
```

## Using the secrets: an application role

A new role, **appconfig**, writes the application's settings file and creates its admin account:

```yaml
---
- name: Folder for the application's settings
  ansible.builtin.file:
    path: /etc/{{ appconfig_name }}
    state: directory
    owner: root
    mode: "0750"

- name: Settings, with the secrets in them
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/{{ appconfig_name }}/app.conf
    owner: root
    mode: "0600"
  no_log: true

- name: An admin account with a password from the vault
  ansible.builtin.user:
    name: "{{ appconfig_admin_user }}"
    shell: /bin/bash
    password: "{{ appconfig_admin_password | password_hash('sha512', appconfig_admin_salt) }}"
  no_log: true
```

```
# roles/appconfig/templates/app.conf.j2
# {{ ansible_managed }}
[database]
host = {{ appconfig_db_host }}
user = {{ appconfig_db_user }}
password = {{ appconfig_db_password }}

[api]
token = {{ appconfig_api_token }}
```

and the web play applies it after the web server:

```yaml
- name: Web servers
  hosts: web
  become: true
  roles:
    - webserver
    - appconfig
```

Without the vault password, Ansible refuses to start, because it cannot decrypt the variables:

```
ansible-playbook site.yml
[ERROR]: Attempting to decrypt but no vault secrets found
ansible-playbook site.yml --ask-vault-pass
```

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="At run time. On control, ansible-playbook reads the vault password from a prompt or a password file, decrypts vault.yml in memory only, and renders app.conf for web1 and web2. Each web server receives its file, owned by root with mode 0600. db1 never receives the web secrets." xmlns="http://www.w3.org/2000/svg">
<style>
.a11d{ font-family:var(--sans),sans-serif; }
.a11d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a11d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a11d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a11d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a11d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a11d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a11d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a11d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a11d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a11d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a11d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a11d .s{ fill:var(--text-dim); font-size:13px; }
.a11d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a11d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a11d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a11d .c{ fill:var(--text-faint); font-size:12.5px; }
.a11d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a11d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a11d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a11d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11d-a); }
.a11d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11d-a); marker-start:url(#a11d-s); }
.a11d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a11d-g); }
.a11d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a11d-a); }
.a11d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a11d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a11d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a11d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a11d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a11d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11d-tri); }
.a11d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11d-tri); }
.a11d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11d-open); }
.a11d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11d-open); }
.a11d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11d-dia); }
.a11d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11d-odia); }
.a11d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11d-dia); marker-end:url(#a11d-open); }
.a11d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11d-solid); }
.a11d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a11d-open); }
.a11d .fill{ fill:var(--text); stroke:none; }
.a11d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a11d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a11d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a11d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a11d .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a11d .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a11d .ks-blue{ fill:#185FA5; font-size:13px; }
.a11d .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a11d .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11d .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a11d .kb-blue, :root[data-theme="dark"] .a11d .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a11d .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a11d .kh-blue{ fill:#85B7EB; }
.a11d .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a11d-blue); }
.a11d .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11d-blue); }
.a11d .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a11d .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a11d .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a11d .ks-amber{ fill:#854F0B; font-size:13px; }
.a11d .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a11d .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11d .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a11d .kb-amber, :root[data-theme="dark"] .a11d .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a11d .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a11d .kh-amber{ fill:#EF9F27; }
.a11d .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a11d-amber); }
.a11d .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11d-amber); }
.a11d .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a11d .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a11d .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a11d .ks-purple{ fill:#534AB7; font-size:13px; }
.a11d .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a11d .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11d .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a11d .kb-purple, :root[data-theme="dark"] .a11d .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a11d .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a11d .kh-purple{ fill:#AFA9EC; }
.a11d .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a11d-purple); }
.a11d .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11d-purple); }
.a11d .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a11d .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a11d .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a11d .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a11d .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a11d .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11d .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a11d .kb-gray, :root[data-theme="dark"] .a11d .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a11d .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a11d .kh-gray{ fill:#B4B2A9; }
.a11d .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a11d-gray); }
.a11d .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11d-gray); }
.a11d .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a11d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a11d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a11d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a11d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a11d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a11d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a11d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a11d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a11d-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a11d-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a11d-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a11d-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a11d">
<rect x="20" y="20" width="330" height="220" rx="10" class="k-purple"/>
<text x="185" y="44" text-anchor="middle" class="kb-purple">control</text>
<rect x="36" y="60" width="140" height="56" rx="7" class="k-amber"/>
<text x="106.0" y="83.5" text-anchor="middle" class="km-amber">~/.vault_pass</text>
<text x="106.0" y="101.5" text-anchor="middle" class="ks-amber">or a prompt</text>
<rect x="196" y="60" width="140" height="56" rx="7" class="k-purple"/>
<text x="266.0" y="83.5" text-anchor="middle" class="km-purple">vault.yml</text>
<text x="266.0" y="101.5" text-anchor="middle" class="ks-purple">encrypted</text>
<rect x="60" y="150" width="250" height="70" rx="7" class="pnl"/>
<text x="185.0" y="171.5" text-anchor="middle" class="b">decrypted in memory only</text>
<text x="185.0" y="189.5" text-anchor="middle" class="c">app.conf rendered per host</text>
<text x="185.0" y="207.5" text-anchor="middle" class="m">no_log: true</text>
<path d="M106 118 L140 146" class="ar-amber"/>
<path d="M266 118 L230 146" class="ar-purple"/>
<rect x="420" y="30" width="280" height="70" rx="7" class="k-blue"/>
<text x="560.0" y="60.5" text-anchor="middle" class="kb-blue">web1: /etc/orders/app.conf</text>
<text x="560.0" y="78.5" text-anchor="middle" class="km-blue">owner root, mode 0600</text>
<path d="M312 185 C 370 185, 370 65, 416 65" class="ar"/>
<rect x="420" y="120" width="280" height="70" rx="7" class="k-blue"/>
<text x="560.0" y="150.5" text-anchor="middle" class="kb-blue">web2: /etc/orders/app.conf</text>
<text x="560.0" y="168.5" text-anchor="middle" class="km-blue">owner root, mode 0600</text>
<path d="M312 185 C 370 185, 370 155, 416 155" class="ar"/>
<rect x="420" y="210" width="280" height="40" rx="7" class="k-gray"/>
<text x="560.0" y="234.5" text-anchor="middle" class="ks-gray">db1: never receives the web secrets</text>
</g>
</svg>
<figcaption>Secrets are decrypted on control, in memory, and sent only where they are needed.</figcaption>
</figure>

With `--ask-vault-pass`, Ansible asks once, decrypts in memory on control, and sends each secret only to the hosts that use it. On web1 the file is `0600`, owned by root:

```
ssh web1 cat /etc/orders/app.conf
cat: /etc/orders/app.conf: Permission denied
ssh web1 sudo cat /etc/orders/app.conf | head -4
# Ansible managed
[database]
host = db1
user = orders
```

Secrets are only as safe as the files they end up in. Encrypting them at rest in the project means nothing if the deployed file is world-readable. Note also that once `group_vars/web/` holds vault data, even an ad-hoc command on a web server needs the vault password, because it loads the host's variables.

## Keeping secrets out of the output

Encryption protects the **files**, not the **output**. A `debug` message, a failed task's error, `-v`, a diff, a CI log or a terminal recording can all print a secret. `no_log: true` hides a task's arguments and results:

```yaml
    - name: A careless debug message
      ansible.builtin.debug:
        msg: "connecting with {{ appconfig_db_password }}"

    - name: The same, with no_log
      ansible.builtin.debug:
        msg: "connecting with {{ appconfig_db_password }}"
      no_log: true
```

```
ok: [web1] => {
    "msg": "connecting with Tr0ub4dor&3"
}
ok: [web1]
```

The second task shows only its status. With `-v`, Ansible says why: *the output has been hidden due to the fact that 'no_log: true' was specified for this result*.

<figure class="diagram">
<svg viewBox="0 0 720 240" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Where secrets leak during a run, and the defence for each: task output and debug messages (no_log), verbose -v output (no_log), --diff of templated files (diff: false), command lines visible in ps and shell history (pass secrets in files or stdin, not arguments), and deployed files (owner root, mode 0600)." xmlns="http://www.w3.org/2000/svg">
<style>
.a11e{ font-family:var(--sans),sans-serif; }
.a11e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a11e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a11e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a11e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a11e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a11e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a11e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a11e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a11e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a11e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a11e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a11e .s{ fill:var(--text-dim); font-size:13px; }
.a11e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a11e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a11e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a11e .c{ fill:var(--text-faint); font-size:12.5px; }
.a11e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a11e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a11e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a11e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11e-a); }
.a11e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11e-a); marker-start:url(#a11e-s); }
.a11e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a11e-g); }
.a11e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a11e-a); }
.a11e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a11e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a11e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a11e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a11e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a11e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11e-tri); }
.a11e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11e-tri); }
.a11e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11e-open); }
.a11e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11e-open); }
.a11e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11e-dia); }
.a11e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11e-odia); }
.a11e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11e-dia); marker-end:url(#a11e-open); }
.a11e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11e-solid); }
.a11e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a11e-open); }
.a11e .fill{ fill:var(--text); stroke:none; }
.a11e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a11e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a11e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a11e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a11e .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a11e .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a11e .ks-teal{ fill:#0F6E56; font-size:13px; }
.a11e .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a11e .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11e .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a11e .kb-teal, :root[data-theme="dark"] .a11e .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a11e .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a11e .kh-teal{ fill:#5DCAA5; }
.a11e .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a11e-teal); }
.a11e .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11e-teal); }
.a11e .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a11e .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a11e .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a11e .ks-red{ fill:#A32D2D; font-size:13px; }
.a11e .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a11e .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11e .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a11e .kb-red, :root[data-theme="dark"] .a11e .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a11e .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a11e .kh-red{ fill:#F09595; }
.a11e .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a11e-red); }
.a11e .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11e-red); }
.a11e .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
</style>
<defs><marker id="a11e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a11e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a11e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a11e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a11e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a11e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a11e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a11e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a11e-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a11e-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker></defs>
<g class="a11e">
<text x="180" y="22" text-anchor="middle" class="kh-red">leak</text>
<text x="540" y="22" text-anchor="middle" class="kh-teal">defence</text>
<rect x="20" y="34" width="320" height="32" rx="5" class="k-red"/>
<text x="180.0" y="54.5" text-anchor="middle" class="ks-red">debug messages, task results</text>
<rect x="380" y="34" width="320" height="32" rx="5" class="k-teal"/>
<text x="540.0" y="54.5" text-anchor="middle" class="km-teal">no_log: true</text>
<path d="M342 50 L376 50" class="ag"/>
<rect x="20" y="74" width="320" height="32" rx="5" class="k-red"/>
<text x="180.0" y="94.5" text-anchor="middle" class="ks-red">-v, -vvv output, CI logs</text>
<rect x="380" y="74" width="320" height="32" rx="5" class="k-teal"/>
<text x="540.0" y="94.5" text-anchor="middle" class="km-teal">no_log: true</text>
<path d="M342 90 L376 90" class="ag"/>
<rect x="20" y="114" width="320" height="32" rx="5" class="k-red"/>
<text x="180.0" y="134.5" text-anchor="middle" class="ks-red">--diff of a templated file</text>
<rect x="380" y="114" width="320" height="32" rx="5" class="k-teal"/>
<text x="540.0" y="134.5" text-anchor="middle" class="km-teal">diff: false on the task</text>
<path d="M342 130 L376 130" class="ag"/>
<rect x="20" y="154" width="320" height="32" rx="5" class="k-red"/>
<text x="180.0" y="174.5" text-anchor="middle" class="ks-red">command lines: ps, shell history</text>
<rect x="380" y="154" width="320" height="32" rx="5" class="k-teal"/>
<text x="540.0" y="174.5" text-anchor="middle" class="km-teal">files or stdin, not arguments</text>
<path d="M342 170 L376 170" class="ag"/>
<rect x="20" y="194" width="320" height="32" rx="5" class="k-red"/>
<text x="180.0" y="214.5" text-anchor="middle" class="ks-red">the deployed file itself</text>
<rect x="380" y="194" width="320" height="32" rx="5" class="k-teal"/>
<text x="540.0" y="214.5" text-anchor="middle" class="km-teal">owner root, mode "0600"</text>
<path d="M342 210 L376 210" class="ag"/>
</g>
</svg>
<figcaption>Encryption at rest is half the job; the output is the other half.</figcaption>
</figure>

Put `no_log: true` on every task that handles a secret, as the appconfig role does. Its cost: when such a task fails, you don't see why. Debug with it temporarily removed, on a test server, and never paste the output anywhere. `--diff` can also show secrets in templated files; `diff: false` on a task turns that off.

## A vault password file

Typing the vault password on every run gets old, and automation can't type at all. A **password file**, readable only by you and **outside** the project, gives it to Ansible:

```
printf 'Lab-Vault-2026\n' > ~/.vault_pass
chmod 600 ~/.vault_pass
```

```
# ansible.cfg
[defaults]
inventory = inventory
remote_user = student
forks = 10
vault_password_file = ~/.vault_pass
```

Now `ansible-playbook site.yml` runs without a prompt. The password file must never be committed: `ansible.cfg` points to your home folder, not the project. In real setups the "file" is often an executable **script** that fetches the password from a password manager (`pass`, 1Password, Bitwarden) or a CI system's secret store; Ansible runs it and reads its output.

## One encrypted value: encrypt_string

For a single secret, `encrypt_string` encrypts one value to paste among plain variables:

```
ansible-vault encrypt_string --stdin-name vault_smtp_password
Reading plaintext input from stdin. (ctrl-d to end input, twice if your input does not end with a newline)
smtp-Pa55word
vault_smtp_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          36663436333563383466303431306535353562393037636433356264646566663035343465616562
          ...
```

Typing the secret at the prompt, rather than as an argument, keeps it out of your shell history. Inline values are handy for one secret, but a whole `vault.yml` is easier to review, change and rekey.

## Passwords for accounts: password_hash

Linux stores password **hashes**, not passwords, so the `user` module's `password` needs one. The `password_hash` filter makes it from the vault's password:

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="password_hash for an account. The vault password Correct-Horse-Battery-9 and the vault salt nOrthWiNdSaLt26 go into the filter password_hash sha512, producing a crypt string starting with $6$nOrthWiNdSaLt26$, which the user module stores in /etc/shadow. With a fixed salt the hash is the same on every run, so the task reports ok; a random salt would change it every run." xmlns="http://www.w3.org/2000/svg">
<style>
.a11f{ font-family:var(--sans),sans-serif; }
.a11f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a11f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a11f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a11f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a11f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a11f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a11f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a11f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a11f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a11f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a11f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a11f .s{ fill:var(--text-dim); font-size:13px; }
.a11f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a11f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a11f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a11f .c{ fill:var(--text-faint); font-size:12.5px; }
.a11f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a11f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a11f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a11f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11f-a); }
.a11f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a11f-a); marker-start:url(#a11f-s); }
.a11f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a11f-g); }
.a11f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a11f-a); }
.a11f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a11f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a11f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a11f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a11f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a11f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11f-tri); }
.a11f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11f-tri); }
.a11f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11f-open); }
.a11f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a11f-open); }
.a11f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11f-dia); }
.a11f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11f-odia); }
.a11f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a11f-dia); marker-end:url(#a11f-open); }
.a11f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a11f-solid); }
.a11f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a11f-open); }
.a11f .fill{ fill:var(--text); stroke:none; }
.a11f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a11f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a11f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a11f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a11f .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a11f .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a11f .ks-teal{ fill:#0F6E56; font-size:13px; }
.a11f .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a11f .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11f .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a11f .kb-teal, :root[data-theme="dark"] .a11f .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a11f .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a11f .kh-teal{ fill:#5DCAA5; }
.a11f .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a11f-teal); }
.a11f .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11f-teal); }
.a11f .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a11f .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a11f .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a11f .ks-amber{ fill:#854F0B; font-size:13px; }
.a11f .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a11f .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11f .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a11f .kb-amber, :root[data-theme="dark"] .a11f .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a11f .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a11f .kh-amber{ fill:#EF9F27; }
.a11f .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a11f-amber); }
.a11f .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11f-amber); }
.a11f .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a11f .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a11f .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a11f .ks-purple{ fill:#534AB7; font-size:13px; }
.a11f .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a11f .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11f .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a11f .kb-purple, :root[data-theme="dark"] .a11f .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a11f .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a11f .kh-purple{ fill:#AFA9EC; }
.a11f .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a11f-purple); }
.a11f .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11f-purple); }
.a11f .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a11f .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a11f .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a11f .ks-green{ fill:#3B6D11; font-size:13px; }
.a11f .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a11f .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a11f .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a11f .kb-green, :root[data-theme="dark"] .a11f .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a11f .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a11f .kh-green{ fill:#97C459; }
.a11f .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a11f-green); }
.a11f .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a11f-green); }
.a11f .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
</style>
<defs><marker id="a11f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a11f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a11f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a11f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a11f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a11f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a11f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a11f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a11f-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a11f-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a11f-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a11f-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker></defs>
<g class="a11f">
<rect x="20" y="30" width="210" height="50" rx="7" class="k-purple"/>
<text x="125.0" y="59.5" text-anchor="middle" class="km-purple">vault_admin_password</text>
<rect x="20" y="110" width="210" height="50" rx="7" class="k-purple"/>
<text x="125.0" y="139.5" text-anchor="middle" class="km-purple">vault_admin_salt</text>
<rect x="270" y="60" width="190" height="70" rx="7" class="k-amber"/>
<text x="365.0" y="90.0" text-anchor="middle" class="km-amber">password_hash(</text>
<text x="365.0" y="109.0" text-anchor="middle" class="km-amber">'sha512', salt)</text>
<path d="M232 55 L266 85" class="ar"/>
<path d="M232 135 L266 105" class="ar"/>
<rect x="500" y="60" width="200" height="70" rx="7" class="k-green"/>
<text x="600.0" y="90.0" text-anchor="middle" class="km-green">$6$nOrthWiNdSaLt26$...</text>
<text x="600.0" y="109.0" text-anchor="middle" class="ks-green">into /etc/shadow</text>
<path d="M462 95 L496 95" class="ar-green"/>
<text x="360" y="186" text-anchor="middle" class="kh-teal">fixed salt: same hash every run, task reports ok</text>
<text x="360" y="206" text-anchor="middle" class="c">random salt: a new hash every run, always changed</text>
</g>
</svg>
<figcaption>Hash on control, store only the hash, keep the run idempotent.</figcaption>
</figure>

```
SSHPASS='Correct-Horse-Battery-9' sshpass -e ssh appadmin@web1 whoami
appadmin
```

A hash includes a random **salt**, so hashing the same password twice gives two different hashes, and the `user` module would report *changed* on every run. A fixed salt, itself kept in the vault, makes the hash, and the playbook, idempotent.

## Changing the vault password, and real rotation

```
ansible-vault rekey group_vars/web/vault.yml
```

`rekey` re-encrypts with a new vault password; do it when someone who knew the old one leaves, then update `~/.vault_pass`. It only covers whole encrypted files: values made with `encrypt_string` need re-encrypting one by one. And rekeying protects the **file**, not the secrets: if the old vault password might have been used to read them, change the secrets themselves (the database password, the API token), then put the new values in the vault.

## Beyond Vault

Vault keeps secrets in the project, encrypted. Larger organizations often keep them **out** of the project entirely, in a secrets manager such as HashiCorp Vault, AWS Secrets Manager or Azure Key Vault, and fetch them at run time with a **lookup** plugin (`community.hashi_vault.hashi_vault`, `amazon.aws.aws_secret`). Either way, the rules are the same: encrypted at rest, least privilege, never in output, and rotated when people leave.

## Hands on

1. Use two vault IDs, `--vault-id dev@prompt --vault-id prod@~/.vault_prod`, so development and production secrets have different passwords.
2. Write a Git `pre-commit` hook that refuses files containing `password:` unless they start with `$ANSIBLE_VAULT`.
3. Remove `no_log` from the settings task and run with `--diff` on a fresh server. What appears on the screen?
4. Someone who knew the vault password leaves the team. List everything you do, in order.

## Summary

- Secrets in plain variables leak through Git, backups and screens. **Ansible Vault** encrypts them with AES-256.
- Convention: `group_vars/GROUP/vars.yml` references `vault_` variables kept in an encrypted `group_vars/GROUP/vault.yml`.
- `ansible-vault create`, `encrypt`, `view`, `edit`, `rekey`, `encrypt_string` manage encrypted data; secrets are decrypted only in memory.
- Give the password with `--ask-vault-pass` or a `vault_password_file` kept outside the project.
- Protect the output too: `no_log: true` on tasks that handle secrets, and deployed files with tight permissions.
- `password_hash` with a fixed salt from the vault creates account passwords idempotently.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-11-vault-and-secrets).

**1. Why are secrets in plain group_vars a problem even if the repository is private?**

- A. They spread to every clone, backup and screen share, and Git keeps them in history
- B. Ansible can't read them
- C. They slow playbooks down
- D. They break idempotency

**2. In the vars.yml / vault.yml convention, what does vars.yml contain?**

- A. The encrypted values
- B. The vault password
- C. Ordinary values plus references such as appconfig_db_password: "{{ vault_db_password }}"
- D. Nothing

**3. Which cipher does Ansible Vault use?**

- A. DES
- B. ROT13
- C. Base64
- D. AES-256

**4. Which command changes an encrypted file in your editor without writing plain text to disk?**

- A. ansible-vault decrypt
- B. ansible-vault edit
- C. ansible-vault view
- D. ansible-vault rekey

**5. Where must the vault password file never be?**

- A. In your home folder with mode 600
- B. In a password manager
- C. Inside the project, where it would be committed
- D. Behind a script

**6. What does no_log: true do?**

- A. Hides the task's arguments and results from the output
- B. Encrypts the task
- C. Disables logging on the host
- D. Skips the task in check mode

**7. Why does the appconfig role use a fixed salt with password_hash?**

- A. A random salt would give a new hash every run, so the user task would always report changed
- B. For stronger hashes
- C. The user module requires it
- D. To allow decryption

**8. Someone who knew the vault password leaves. What should you do?**

- A. Nothing; the files are encrypted
- B. Delete the vault
- C. Rekey the vault and change the secrets themselves
- D. Only change ~/.vault_pass
