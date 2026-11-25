# Chapter 1: How Ansible Works

Northwind is a small coffee company with a big ambition: an online shop for its beans. Today its web site runs on two web servers, `web1` and `web2`, with a database server, `db1`, behind them. One person, you, looks after all three. Every change means logging in to each server in turn and typing the same commands, and hoping you typed them the same way each time.

This book follows Northwind's infrastructure as it grows, from those three servers to a load-balanced, firewalled stack with an internal API and a routed network between its sites. Every step is automated with **Ansible**, and every example runs on the **DockerLab** platform. This chapter explains what Ansible is, how it works, and how the book is organized around the labs.

## The problem: servers configured by hand

Configuring servers by hand works for one server. It stops working at three, for reasons that have nothing to do with skill:

- **Repetition.** The same steps on every server, each a chance for a typo.
- **Drift.** A quick fix on one server, never applied to the others, makes "identical" servers subtly different. Six months later nobody knows which difference matters.
- **No record.** The configuration lives in people's heads and shell histories. A new colleague, or you on a bad day, can't rebuild a server from them.
- **No review.** A command typed into a production server is reviewed by nobody, before or after.

<figure class="diagram">
<svg viewBox="0 0 720 330" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Two ways to configure three servers. By hand: an administrator logs in to web1, web2 and db1 in turn and types commands, and the servers drift apart: web2 got a quick fix, db1 missed an update. With Ansible: playbooks kept in Git describe the desired state, and the control node applies the same description to every server, so they match." xmlns="http://www.w3.org/2000/svg">
<style>
.a1a{ font-family:var(--sans),sans-serif; }
.a1a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a1a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a1a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a1a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a1a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a1a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a1a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a1a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a1a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a1a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a1a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a1a .s{ fill:var(--text-dim); font-size:13px; }
.a1a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a1a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a1a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a1a .c{ fill:var(--text-faint); font-size:12.5px; }
.a1a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a1a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a1a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a1a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1a-a); }
.a1a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1a-a); marker-start:url(#a1a-s); }
.a1a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a1a-g); }
.a1a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-a); }
.a1a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a1a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a1a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a1a .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a1a .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a1a .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1a-tri); }
.a1a .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-tri); }
.a1a .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1a-open); }
.a1a .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-open); }
.a1a .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1a-dia); }
.a1a .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1a-odia); }
.a1a .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1a-dia); marker-end:url(#a1a-open); }
.a1a .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1a-solid); }
.a1a .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a1a-open); }
.a1a .fill{ fill:var(--text); stroke:none; }
.a1a .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a1a .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a1a .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a1a .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a1a .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a1a .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a1a .ks-teal{ fill:#0F6E56; font-size:13px; }
.a1a .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a1a .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1a .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a1a .kb-teal, :root[data-theme="dark"] .a1a .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a1a .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a1a .kh-teal{ fill:#5DCAA5; }
.a1a .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a1a-teal); }
.a1a .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-teal); }
.a1a .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a1a .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a1a .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a1a .ks-amber{ fill:#854F0B; font-size:13px; }
.a1a .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a1a .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1a .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a1a .kb-amber, :root[data-theme="dark"] .a1a .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a1a .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a1a .kh-amber{ fill:#EF9F27; }
.a1a .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a1a-amber); }
.a1a .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-amber); }
.a1a .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a1a .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a1a .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a1a .ks-purple{ fill:#534AB7; font-size:13px; }
.a1a .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a1a .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1a .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a1a .kb-purple, :root[data-theme="dark"] .a1a .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a1a .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a1a .kh-purple{ fill:#AFA9EC; }
.a1a .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a1a-purple); }
.a1a .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-purple); }
.a1a .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a1a .k-red{ fill:#FCEBEB; stroke:#E24B4A; stroke-width:1.6; }
.a1a .kb-red{ fill:#791F1F; font-size:13px; font-weight:700; }
.a1a .ks-red{ fill:#A32D2D; font-size:13px; }
.a1a .km-red{ fill:#791F1F; font-size:12.5px; font-family:var(--mono),monospace; }
.a1a .kh-red{ fill:#A32D2D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1a .k-red{ fill:#791F1F; stroke:#F09595; }
:root[data-theme="dark"] .a1a .kb-red, :root[data-theme="dark"] .a1a .km-red{ fill:#FCEBEB; }
:root[data-theme="dark"] .a1a .ks-red{ fill:#F7C1C1; }
:root[data-theme="dark"] .a1a .kh-red{ fill:#F09595; }
.a1a .ar-red{ stroke:#E24B4A; stroke-width:2; fill:none; marker-end:url(#a1a-red); }
.a1a .ad-red{ stroke:#E24B4A; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-red); }
.a1a .ln-red{ stroke:#E24B4A; stroke-width:2; fill:none; }
.a1a .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a1a .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a1a .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a1a .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a1a .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1a .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a1a .kb-gray, :root[data-theme="dark"] .a1a .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a1a .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a1a .kh-gray{ fill:#B4B2A9; }
.a1a .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a1a-gray); }
.a1a .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1a-gray); }
.a1a .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a1a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a1a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a1a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a1a-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a1a-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a1a-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a1a-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a1a-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a1a-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a1a-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a1a-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a1a-red" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#E24B4A"/></marker><marker id="a1a-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a1a">
<text x="180" y="22" text-anchor="middle" class="h">by hand</text>
<text x="540" y="22" text-anchor="middle" class="h">with Ansible</text>
<rect x="100" y="40" width="160" height="44" rx="7" class="k-gray"/>
<text x="180.0" y="57.5" text-anchor="middle" class="kb-gray">you, over SSH</text>
<text x="180.0" y="75.5" text-anchor="middle" class="ks-gray">one server at a time</text>
<path d="M180 84 L75 172" class="ad-gray"/>
<rect x="20" y="176" width="110" height="54" rx="7" class="k-teal"/>
<line x1="30" y1="185" x2="46" y2="185" class="ln-teal"/>
<line x1="30" y1="190" x2="46" y2="190" class="ln-teal"/>
<text x="83.0" y="198" text-anchor="middle" class="kb-teal">web1</text>
<text x="75.0" y="217" text-anchor="middle" class="ks-teal">v1.2, 3 users</text>
<path d="M180 84 L190 172" class="ad-gray"/>
<rect x="135" y="176" width="110" height="54" rx="7" class="k-amber"/>
<line x1="145" y1="185" x2="161" y2="185" class="ln-amber"/>
<line x1="145" y1="190" x2="161" y2="190" class="ln-amber"/>
<text x="198.0" y="198" text-anchor="middle" class="kb-amber">web2</text>
<text x="190.0" y="217" text-anchor="middle" class="ks-amber">v1.2 + quick fix</text>
<path d="M180 84 L305 172" class="ad-gray"/>
<rect x="250" y="176" width="110" height="54" rx="7" class="k-red"/>
<line x1="260" y1="185" x2="276" y2="185" class="ln-red"/>
<line x1="260" y1="190" x2="276" y2="190" class="ln-red"/>
<text x="313.0" y="198" text-anchor="middle" class="kb-red">db1</text>
<text x="305.0" y="217" text-anchor="middle" class="ks-red">v1.1, 2 users</text>
<text x="180" y="260" text-anchor="middle" class="s">the same job, done three times,</text>
<text x="180" y="278" text-anchor="middle" class="s">three slightly different ways:</text>
<text x="180" y="300" text-anchor="middle" class="kh-red">drift</text>
<line x1="360" y1="30" x2="360" y2="310" class="zone"/>
<rect x="420" y="40" width="120" height="44" rx="7" class="k-purple"/>
<text x="480.0" y="57.5" text-anchor="middle" class="kb-purple">playbooks</text>
<text x="480.0" y="75.5" text-anchor="middle" class="ks-purple">in Git</text>
<rect x="570" y="35" width="130" height="54" rx="7" class="dark"/>
<text x="635.0" y="58" text-anchor="middle" class="on">control</text>
<text x="635.0" y="76" text-anchor="middle" class="on" style="fill:var(--bg);font-weight:400">applies them</text>
<path d="M540 62 L566 62" class="ar-purple"/>
<path d="M635 90 L435 172" class="ar"/>
<rect x="385" y="176" width="100" height="54" rx="7" class="k-teal"/>
<line x1="395" y1="185" x2="411" y2="185" class="ln-teal"/>
<line x1="395" y1="190" x2="411" y2="190" class="ln-teal"/>
<text x="443.0" y="198" text-anchor="middle" class="kb-teal">web1</text>
<text x="435.0" y="217" text-anchor="middle" class="ks-teal">desired state</text>
<path d="M635 90 L545 172" class="ar"/>
<rect x="495" y="176" width="100" height="54" rx="7" class="k-teal"/>
<line x1="505" y1="185" x2="521" y2="185" class="ln-teal"/>
<line x1="505" y1="190" x2="521" y2="190" class="ln-teal"/>
<text x="553.0" y="198" text-anchor="middle" class="kb-teal">web2</text>
<text x="545.0" y="217" text-anchor="middle" class="ks-teal">desired state</text>
<path d="M635 90 L655 172" class="ar"/>
<rect x="605" y="176" width="100" height="54" rx="7" class="k-teal"/>
<line x1="615" y1="185" x2="631" y2="185" class="ln-teal"/>
<line x1="615" y1="190" x2="631" y2="190" class="ln-teal"/>
<text x="663.0" y="198" text-anchor="middle" class="kb-teal">db1</text>
<text x="655.0" y="217" text-anchor="middle" class="ks-teal">desired state</text>
<text x="560" y="260" text-anchor="middle" class="s">one description, applied to all,</text>
<text x="560" y="278" text-anchor="middle" class="s">checked and reviewed:</text>
<text x="560" y="300" text-anchor="middle" class="kh-teal">no drift</text>
</g>
</svg>
<figcaption>Configuration by hand drifts; configuration as code converges.</figcaption>
</figure>

The answer is **infrastructure as code**: describe what each server should look like in text files, keep those files in version control, and let a tool make the servers match. The files become the documentation, the record of every change, and the way to build a new server in minutes. Ansible is one of the most widely used tools for doing this.

## What Ansible is

Ansible is an open source **automation engine**, written in Python and maintained by Red Hat and a large community. You describe the **desired state** of your machines in YAML files; Ansible connects to them and makes the necessary changes. It configures servers (packages, files, users, services), deploys applications, and orchestrates multi-step jobs across many machines, such as a rolling update behind a load balancer.

Three design choices set Ansible apart:

1. **Agentless.** Nothing is installed on the managed machines. Ansible connects over **SSH**, the protocol you already use to log in, and needs only Python on the other side.
2. **Push-based.** You run Ansible from one machine, the **control node**, and it pushes changes out when you decide. Nothing polls in the background.
3. **Declarative and idempotent.** You say *what* you want ("nginx is installed and running"), not *how* to get there. Ansible checks first and changes only what is wrong, so running the same playbook twice is safe.

## The control node and the managed nodes

Every Ansible setup has two kinds of machine:

| | Control node | Managed nodes |
|---|---|---|
| What it is | The machine you run Ansible on | The machines Ansible configures |
| Needs | Ansible (and its Python), an SSH client, your project files | An SSH server, Python 3, an account Ansible can log in to |
| In this book | `control`, 10.0.0.10 | `web1`, `web2`, `db1`, and later `lb`, `web3`, `api1` and routers |

<figure class="diagram">
<svg viewBox="0 0 720 330" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The control node holds Ansible, the inventory and the playbooks. It pushes tasks over SSH to the managed nodes web1, web2 and db1, which need only an SSH server and Python 3. No agent runs on the managed nodes." xmlns="http://www.w3.org/2000/svg">
<style>
.a1b{ font-family:var(--sans),sans-serif; }
.a1b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a1b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a1b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a1b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a1b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a1b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a1b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a1b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a1b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a1b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a1b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a1b .s{ fill:var(--text-dim); font-size:13px; }
.a1b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a1b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a1b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a1b .c{ fill:var(--text-faint); font-size:12.5px; }
.a1b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a1b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a1b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a1b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1b-a); }
.a1b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1b-a); marker-start:url(#a1b-s); }
.a1b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a1b-g); }
.a1b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a1b-a); }
.a1b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a1b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a1b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a1b .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a1b .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a1b .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1b-tri); }
.a1b .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1b-tri); }
.a1b .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1b-open); }
.a1b .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1b-open); }
.a1b .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1b-dia); }
.a1b .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1b-odia); }
.a1b .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1b-dia); marker-end:url(#a1b-open); }
.a1b .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1b-solid); }
.a1b .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a1b-open); }
.a1b .fill{ fill:var(--text); stroke:none; }
.a1b .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a1b .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a1b .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a1b .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a1b .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a1b .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a1b .ks-blue{ fill:#185FA5; font-size:13px; }
.a1b .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a1b .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1b .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a1b .kb-blue, :root[data-theme="dark"] .a1b .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a1b .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a1b .kh-blue{ fill:#85B7EB; }
.a1b .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a1b-blue); }
.a1b .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1b-blue); }
.a1b .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a1b .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a1b .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a1b .ks-purple{ fill:#534AB7; font-size:13px; }
.a1b .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a1b .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1b .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a1b .kb-purple, :root[data-theme="dark"] .a1b .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a1b .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a1b .kh-purple{ fill:#AFA9EC; }
.a1b .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a1b-purple); }
.a1b .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1b-purple); }
.a1b .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a1b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a1b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a1b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a1b-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a1b-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a1b-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a1b-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a1b-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a1b-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a1b-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a1b">
<rect x="20" y="20" width="260" height="290" rx="10" class="k-purple"/>
<text x="150" y="44" text-anchor="middle" class="kb-purple">control node</text>
<rect x="40" y="60" width="220" height="48" rx="7" class="pnl"/>
<text x="150.0" y="79.5" text-anchor="middle" class="b">ansible-core 2.19</text>
<text x="150.0" y="97.5" text-anchor="middle" class="c">the engine, modules</text>
<rect x="40" y="120" width="220" height="48" rx="7" class="pnl"/>
<text x="150.0" y="139.5" text-anchor="middle" class="b">inventory</text>
<text x="150.0" y="157.5" text-anchor="middle" class="c">which hosts, which groups</text>
<rect x="40" y="180" width="220" height="48" rx="7" class="pnl"/>
<text x="150.0" y="199.5" text-anchor="middle" class="b">playbooks, roles</text>
<text x="150.0" y="217.5" text-anchor="middle" class="c">what they should look like</text>
<rect x="40" y="240" width="220" height="48" rx="7" class="pnl"/>
<text x="150.0" y="259.5" text-anchor="middle" class="b">SSH client + key</text>
<text x="150.0" y="277.5" text-anchor="middle" class="c">to log in without a password</text>
<rect x="460" y="30" width="240" height="80" rx="10" class="k-blue"/>
<text x="520" y="56" text-anchor="middle" class="kb-blue">web1</text>
<text x="520" y="76" text-anchor="middle" class="ks-blue">10.0.0.11</text>
<rect x="580" y="42" width="110" height="26" rx="4" class="pnl"/>
<text x="635.0" y="59.5" text-anchor="middle" class="m">sshd :22</text>
<rect x="580" y="74" width="110" height="26" rx="4" class="pnl"/>
<text x="635.0" y="91.5" text-anchor="middle" class="m">python3</text>
<path d="M282 165 C 360 165, 380 70, 456 70" class="ar"/>
<rect x="460" y="130" width="240" height="80" rx="10" class="k-blue"/>
<text x="520" y="156" text-anchor="middle" class="kb-blue">web2</text>
<text x="520" y="176" text-anchor="middle" class="ks-blue">10.0.0.12</text>
<rect x="580" y="142" width="110" height="26" rx="4" class="pnl"/>
<text x="635.0" y="159.5" text-anchor="middle" class="m">sshd :22</text>
<rect x="580" y="174" width="110" height="26" rx="4" class="pnl"/>
<text x="635.0" y="191.5" text-anchor="middle" class="m">python3</text>
<path d="M282 165 C 360 165, 380 170, 456 170" class="ar"/>
<rect x="460" y="230" width="240" height="80" rx="10" class="k-blue"/>
<text x="520" y="256" text-anchor="middle" class="kb-blue">db1</text>
<text x="520" y="276" text-anchor="middle" class="ks-blue">10.0.0.21</text>
<rect x="580" y="242" width="110" height="26" rx="4" class="pnl"/>
<text x="635.0" y="259.5" text-anchor="middle" class="m">sshd :22</text>
<rect x="580" y="274" width="110" height="26" rx="4" class="pnl"/>
<text x="635.0" y="291.5" text-anchor="middle" class="m">python3</text>
<path d="M282 165 C 360 165, 380 270, 456 270" class="ar"/>
<text x="370" y="152" text-anchor="middle" class="a">SSH</text>
<text x="370" y="196" text-anchor="middle" class="c">push</text>
</g>
</svg>
<figcaption>Agentless: the managed nodes need only what a Linux server usually has already.</figcaption>
</figure>

The control node holds the **inventory** (which machines exist, in which groups) and the **playbooks** (what they should look like). When you run a playbook, Ansible opens SSH connections to the managed nodes, several at a time, and works through the tasks in order.

## What happens when Ansible runs a task

Each task uses a **module**, a small program that knows how to bring one kind of thing into a wanted state: the `apt` module manages packages, `copy` manages files, `user` manages accounts, `service` manages daemons. For each task on each host, Ansible:

1. Connects to the host over SSH (and reuses the connection for later tasks).
2. Copies the module, packed with its arguments into one Python file, to a temporary folder.
3. Runs it with the host's Python, as your user or, with `become`, through `sudo` as root.
4. Reads the module's answer, a JSON document saying whether anything **changed**, **failed**, or was already **ok**.
5. Deletes the temporary folder.

<figure class="diagram">
<svg viewBox="0 0 720 360" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A sequence diagram of one task. The control node connects to web1 over SSH, makes a temporary folder, copies the module packed as an AnsiballZ Python file, runs it with python3 (through sudo when become is set), receives a JSON result such as changed true, and removes the temporary folder." xmlns="http://www.w3.org/2000/svg">
<style>
.a1c{ font-family:var(--sans),sans-serif; }
.a1c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a1c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a1c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a1c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a1c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a1c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a1c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a1c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a1c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a1c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a1c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a1c .s{ fill:var(--text-dim); font-size:13px; }
.a1c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a1c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a1c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a1c .c{ fill:var(--text-faint); font-size:12.5px; }
.a1c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a1c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a1c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a1c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1c-a); }
.a1c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1c-a); marker-start:url(#a1c-s); }
.a1c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a1c-g); }
.a1c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a1c-a); }
.a1c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a1c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a1c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a1c .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a1c .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a1c .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1c-tri); }
.a1c .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1c-tri); }
.a1c .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1c-open); }
.a1c .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1c-open); }
.a1c .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1c-dia); }
.a1c .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1c-odia); }
.a1c .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1c-dia); marker-end:url(#a1c-open); }
.a1c .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1c-solid); }
.a1c .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a1c-open); }
.a1c .fill{ fill:var(--text); stroke:none; }
.a1c .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a1c .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a1c .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a1c .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a1c .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a1c .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a1c .ks-blue{ fill:#185FA5; font-size:13px; }
.a1c .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a1c .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1c .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a1c .kb-blue, :root[data-theme="dark"] .a1c .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a1c .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a1c .kh-blue{ fill:#85B7EB; }
.a1c .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a1c-blue); }
.a1c .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1c-blue); }
.a1c .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a1c .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a1c .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a1c .ks-purple{ fill:#534AB7; font-size:13px; }
.a1c .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a1c .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1c .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a1c .kb-purple, :root[data-theme="dark"] .a1c .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a1c .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a1c .kh-purple{ fill:#AFA9EC; }
.a1c .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a1c-purple); }
.a1c .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1c-purple); }
.a1c .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
</style>
<defs><marker id="a1c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a1c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a1c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a1c-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a1c-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a1c-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a1c-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a1c-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a1c-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a1c-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker></defs>
<g class="a1c">
<rect x="65.0" y="12" width="150" height="36" rx="4" class="k-purple"/>
<text x="140.0" y="34.5" text-anchor="middle" class="kb-purple">control</text>
<line x1="140" y1="48" x2="140" y2="340" class="gd"/>
<rect x="485.0" y="12" width="150" height="36" rx="4" class="k-blue"/>
<text x="560.0" y="34.5" text-anchor="middle" class="kb-blue">web1</text>
<line x1="560" y1="48" x2="560" y2="340" class="gd"/>
<line x1="145" y1="80" x2="555" y2="80" class="msg"/>
<text x="350.0" y="74" text-anchor="middle" class="s">1. connect over SSH (reused for later tasks)</text>
<line x1="145" y1="124" x2="555" y2="124" class="msg"/>
<text x="350.0" y="118" text-anchor="middle" class="s">2. mkdir ~/.ansible/tmp/ansible-tmp-...</text>
<line x1="145" y1="168" x2="555" y2="168" class="msg"/>
<text x="350.0" y="162" text-anchor="middle" class="s">3. copy AnsiballZ_apt.py (module + arguments)</text>
<line x1="145" y1="212" x2="555" y2="212" class="msg"/>
<text x="350.0" y="206" text-anchor="middle" class="s">4. run it: sudo python3 AnsiballZ_apt.py</text>
<line x1="555" y1="256" x2="145" y2="256" class="ret"/>
<text x="350.0" y="250" text-anchor="middle" class="c">5. {"changed": true, "msg": "..."}</text>
<line x1="145" y1="300" x2="555" y2="300" class="msg"/>
<text x="350.0" y="294" text-anchor="middle" class="s">6. rm -rf the temporary folder</text>
<rect x="555" y="150" width="10" height="120" class="act"/>
</g>
</svg>
<figcaption>Every task follows these steps on every host; nothing stays behind.</figcaption>
</figure>

Nothing stays behind on the managed node. That is what **agentless** means: no daemon to install, patch, secure or monitor, and nothing to break when the control node is switched off. Chapter 2 shows these steps happening, with `-vvv`.

## Declarative, idempotent automation

A shell script is **imperative**: a list of commands to run. Run it twice and it runs every command twice, adding a second copy of a line to a file or failing because a user already exists. An Ansible task is **declarative**: it describes an end state, and the module works out whether anything needs doing.

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A shell script compared with an Ansible task. The script runs useradd and echo commands every time: the second run fails because the user exists and appends a duplicate line. The task declares a user deploy that is present: the first run reports changed, the second reports ok and changes nothing." xmlns="http://www.w3.org/2000/svg">
<style>
.a1d{ font-family:var(--sans),sans-serif; }
.a1d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a1d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a1d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a1d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a1d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a1d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a1d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a1d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a1d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a1d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a1d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a1d .s{ fill:var(--text-dim); font-size:13px; }
.a1d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a1d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a1d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a1d .c{ fill:var(--text-faint); font-size:12.5px; }
.a1d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a1d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a1d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a1d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1d-a); }
.a1d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1d-a); marker-start:url(#a1d-s); }
.a1d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a1d-g); }
.a1d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a1d-a); }
.a1d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a1d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a1d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a1d .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a1d .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a1d .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1d-tri); }
.a1d .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1d-tri); }
.a1d .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1d-open); }
.a1d .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1d-open); }
.a1d .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1d-dia); }
.a1d .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1d-odia); }
.a1d .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1d-dia); marker-end:url(#a1d-open); }
.a1d .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1d-solid); }
.a1d .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a1d-open); }
.a1d .fill{ fill:var(--text); stroke:none; }
.a1d .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a1d .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a1d .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a1d .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a1d .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a1d .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a1d .ks-teal{ fill:#0F6E56; font-size:13px; }
.a1d .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a1d .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1d .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a1d .kb-teal, :root[data-theme="dark"] .a1d .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a1d .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a1d .kh-teal{ fill:#5DCAA5; }
.a1d .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a1d-teal); }
.a1d .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1d-teal); }
.a1d .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a1d .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a1d .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a1d .ks-coral{ fill:#993C1D; font-size:13px; }
.a1d .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a1d .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1d .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a1d .kb-coral, :root[data-theme="dark"] .a1d .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a1d .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a1d .kh-coral{ fill:#F0997B; }
.a1d .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a1d-coral); }
.a1d .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1d-coral); }
.a1d .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
</style>
<defs><marker id="a1d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a1d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a1d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a1d-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a1d-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a1d-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a1d-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a1d-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a1d-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a1d-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker></defs>
<g class="a1d">
<text x="180" y="22" text-anchor="middle" class="h">imperative: a script</text>
<text x="540" y="22" text-anchor="middle" class="h">declarative: a task</text>
<rect x="20" y="36" width="320" height="70" rx="6" class="k-coral"/>
<text x="32.0" y="56" text-anchor="start" class="m">useradd -m deploy</text>
<text x="32.0" y="74" text-anchor="start" class="m">echo 'PermitRootLogin no' \</text>
<text x="70.0" y="92" text-anchor="start" class="m">&gt;&gt; /etc/ssh/sshd_config</text>
<rect x="380" y="36" width="320" height="88" rx="6" class="k-teal"/>
<text x="392.0" y="56" text-anchor="start" class="m">- name: The deploy account</text>
<text x="407.2" y="74" text-anchor="start" class="m">ansible.builtin.user:</text>
<text x="422.4" y="92" text-anchor="start" class="m">name: deploy</text>
<text x="422.4" y="110" text-anchor="start" class="m">state: present</text>
<text x="360" y="150" text-anchor="middle" class="c">1st run</text>
<text x="180" y="150" text-anchor="middle" class="ks-coral">adds the user, adds the line</text>
<text x="540" y="150" text-anchor="middle" class="ks-teal">changed: user created</text>
<text x="360" y="188" text-anchor="middle" class="c">2nd run</text>
<text x="180" y="188" text-anchor="middle" class="ks-coral">useradd fails: user exists;</text>
<text x="540" y="188" text-anchor="middle" class="ks-teal">ok: already present,</text>
<text x="180" y="214" text-anchor="middle" class="ks-coral">the line is now there twice</text>
<text x="540" y="214" text-anchor="middle" class="ks-teal">nothing to do</text>
<rect x="20" y="240" width="320" height="44" rx="7" class="k-coral"/>
<text x="180.0" y="266.5" text-anchor="middle" class="kb-coral">says HOW: steps to run</text>
<rect x="380" y="240" width="320" height="44" rx="7" class="k-teal"/>
<text x="540.0" y="266.5" text-anchor="middle" class="kb-teal">says WHAT: the state to reach</text>
</g>
</svg>
<figcaption>Idempotency: running a declarative task again changes nothing that is already right.</figcaption>
</figure>

This property, **idempotency**, is the idea everything else in Ansible builds on. Because a playbook is safe to run again, you can run it after every change, on a new server, or every night to put back whatever someone changed by hand. The output tells you which tasks *changed* something, which makes Ansible a drift detector as well as a configuration tool.

## The building blocks

Ansible's vocabulary is small, and each term contains the previous one:

| Term | What it is | Chapter |
|---|---|---|
| **Module** | A unit of work: install a package, copy a file, start a service | 3 |
| **Task** | One call to a module, with a name and arguments | 3, 5 |
| **Play** | A list of tasks for a group of hosts | 5 |
| **Playbook** | A YAML file holding one or more plays | 5 |
| **Inventory** | The hosts and their groups, with variables | 4 |
| **Variables and facts** | Data that makes plays reusable; facts are discovered from each host | 6 |
| **Template** | A file with blanks, filled in per host | 7 |
| **Handler** | A task that runs only when notified by a change | 9 |
| **Role** | A packaged, reusable unit: tasks, templates, handlers and defaults for one job | 10 |
| **Collection** | A distribution format for modules, roles and plugins | 10 |

<figure class="diagram">
<svg viewBox="0 0 720 330" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Ansible&#x27;s building blocks, nested. A collection contains roles and modules. A playbook contains plays; each play targets hosts from the inventory and contains tasks; each task calls one module. Roles package tasks, handlers, templates and defaults. Variables and facts feed plays, tasks and templates." xmlns="http://www.w3.org/2000/svg">
<style>
.a1e{ font-family:var(--sans),sans-serif; }
.a1e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a1e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a1e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a1e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a1e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a1e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a1e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a1e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a1e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a1e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a1e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a1e .s{ fill:var(--text-dim); font-size:13px; }
.a1e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a1e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .c{ fill:var(--text-faint); font-size:12.5px; }
.a1e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a1e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a1e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a1e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1e-a); }
.a1e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1e-a); marker-start:url(#a1e-s); }
.a1e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a1e-g); }
.a1e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-a); }
.a1e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a1e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a1e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a1e .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a1e .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a1e .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1e-tri); }
.a1e .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-tri); }
.a1e .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1e-open); }
.a1e .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-open); }
.a1e .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1e-dia); }
.a1e .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1e-odia); }
.a1e .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1e-dia); marker-end:url(#a1e-open); }
.a1e .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1e-solid); }
.a1e .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a1e-open); }
.a1e .fill{ fill:var(--text); stroke:none; }
.a1e .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a1e .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a1e .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a1e .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a1e .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a1e .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a1e .ks-blue{ fill:#185FA5; font-size:13px; }
.a1e .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1e .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a1e .kb-blue, :root[data-theme="dark"] .a1e .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a1e .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a1e .kh-blue{ fill:#85B7EB; }
.a1e .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a1e-blue); }
.a1e .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-blue); }
.a1e .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a1e .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a1e .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a1e .ks-teal{ fill:#0F6E56; font-size:13px; }
.a1e .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1e .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a1e .kb-teal, :root[data-theme="dark"] .a1e .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a1e .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a1e .kh-teal{ fill:#5DCAA5; }
.a1e .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a1e-teal); }
.a1e .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-teal); }
.a1e .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a1e .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a1e .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a1e .ks-amber{ fill:#854F0B; font-size:13px; }
.a1e .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1e .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a1e .kb-amber, :root[data-theme="dark"] .a1e .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a1e .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a1e .kh-amber{ fill:#EF9F27; }
.a1e .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a1e-amber); }
.a1e .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-amber); }
.a1e .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a1e .k-purple{ fill:#EEEDFE; stroke:#7F77DD; stroke-width:1.6; }
.a1e .kb-purple{ fill:#3C3489; font-size:13px; font-weight:700; }
.a1e .ks-purple{ fill:#534AB7; font-size:13px; }
.a1e .km-purple{ fill:#3C3489; font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .kh-purple{ fill:#534AB7; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1e .k-purple{ fill:#3C3489; stroke:#AFA9EC; }
:root[data-theme="dark"] .a1e .kb-purple, :root[data-theme="dark"] .a1e .km-purple{ fill:#EEEDFE; }
:root[data-theme="dark"] .a1e .ks-purple{ fill:#CECBF6; }
:root[data-theme="dark"] .a1e .kh-purple{ fill:#AFA9EC; }
.a1e .ar-purple{ stroke:#7F77DD; stroke-width:2; fill:none; marker-end:url(#a1e-purple); }
.a1e .ad-purple{ stroke:#7F77DD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-purple); }
.a1e .ln-purple{ stroke:#7F77DD; stroke-width:2; fill:none; }
.a1e .k-green{ fill:#EAF3DE; stroke:#639922; stroke-width:1.6; }
.a1e .kb-green{ fill:#27500A; font-size:13px; font-weight:700; }
.a1e .ks-green{ fill:#3B6D11; font-size:13px; }
.a1e .km-green{ fill:#27500A; font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .kh-green{ fill:#3B6D11; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1e .k-green{ fill:#27500A; stroke:#97C459; }
:root[data-theme="dark"] .a1e .kb-green, :root[data-theme="dark"] .a1e .km-green{ fill:#EAF3DE; }
:root[data-theme="dark"] .a1e .ks-green{ fill:#C0DD97; }
:root[data-theme="dark"] .a1e .kh-green{ fill:#97C459; }
.a1e .ar-green{ stroke:#639922; stroke-width:2; fill:none; marker-end:url(#a1e-green); }
.a1e .ad-green{ stroke:#639922; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-green); }
.a1e .ln-green{ stroke:#639922; stroke-width:2; fill:none; }
.a1e .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a1e .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a1e .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a1e .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a1e .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1e .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a1e .kb-gray, :root[data-theme="dark"] .a1e .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a1e .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a1e .kh-gray{ fill:#B4B2A9; }
.a1e .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a1e-gray); }
.a1e .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1e-gray); }
.a1e .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a1e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a1e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a1e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a1e-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a1e-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a1e-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a1e-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a1e-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a1e-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a1e-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a1e-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a1e-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#7F77DD"/></marker><marker id="a1e-green" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#639922"/></marker><marker id="a1e-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a1e">
<rect x="20" y="20" width="470" height="290" rx="12" class="k-gray"/>
<text x="40" y="44" text-anchor="start" class="kb-gray">playbook  site.yml</text>
<rect x="40" y="60" width="205" height="230" rx="10" class="k-blue"/>
<text x="52" y="82" text-anchor="start" class="kb-blue">play: Web servers</text>
<text x="52" y="100" text-anchor="start" class="km-blue">hosts: web</text>
<rect x="54" y="114" width="177" height="40" rx="7" class="pnl"/>
<text x="142.5" y="129.5" text-anchor="middle" class="b">task: apt</text>
<text x="142.5" y="147.5" text-anchor="middle" class="c">calls one module</text>
<rect x="54" y="164" width="177" height="40" rx="7" class="pnl"/>
<text x="142.5" y="179.5" text-anchor="middle" class="b">task: template</text>
<text x="142.5" y="197.5" text-anchor="middle" class="c">calls one module</text>
<rect x="54" y="214" width="177" height="40" rx="7" class="pnl"/>
<text x="142.5" y="229.5" text-anchor="middle" class="b">task: service</text>
<text x="142.5" y="247.5" text-anchor="middle" class="c">calls one module</text>
<rect x="265" y="60" width="205" height="230" rx="10" class="k-blue"/>
<text x="277" y="82" text-anchor="start" class="kb-blue">play: Database</text>
<text x="277" y="100" text-anchor="start" class="km-blue">hosts: db</text>
<rect x="279" y="114" width="177" height="40" rx="7" class="pnl"/>
<text x="367.5" y="129.5" text-anchor="middle" class="b">task: apt</text>
<text x="367.5" y="147.5" text-anchor="middle" class="c">calls one module</text>
<rect x="279" y="164" width="177" height="40" rx="7" class="pnl"/>
<text x="367.5" y="179.5" text-anchor="middle" class="b">task: template</text>
<text x="367.5" y="197.5" text-anchor="middle" class="c">calls one module</text>
<rect x="279" y="220" width="177" height="54" rx="7" class="k-purple"/>
<text x="367.5" y="242.5" text-anchor="middle" class="kb-purple">role: database</text>
<text x="367.5" y="260.5" text-anchor="middle" class="ks-purple">tasks, templates ...</text>
<rect x="520" y="20" width="180" height="62" rx="7" class="k-teal"/>
<text x="610.0" y="46.5" text-anchor="middle" class="kb-teal">inventory</text>
<text x="610.0" y="64.5" text-anchor="middle" class="ks-teal">hosts and groups</text>
<rect x="520" y="100" width="180" height="62" rx="7" class="k-amber"/>
<text x="610.0" y="126.5" text-anchor="middle" class="kb-amber">variables, facts</text>
<text x="610.0" y="144.5" text-anchor="middle" class="ks-amber">data for plays</text>
<rect x="520" y="180" width="180" height="62" rx="7" class="k-green"/>
<text x="610.0" y="206.5" text-anchor="middle" class="kb-green">modules</text>
<text x="610.0" y="224.5" text-anchor="middle" class="ks-green">apt, copy, user ...</text>
<rect x="520" y="260" width="180" height="50" rx="7" class="k-green"/>
<text x="610.0" y="280.5" text-anchor="middle" class="kb-green">collections</text>
<text x="610.0" y="298.5" text-anchor="middle" class="ks-green">ship modules, roles</text>
<path d="M518 51 L494 51" class="ar-teal"/>
<path d="M518 131 L494 131" class="ar-amber"/>
<path d="M518 211 L494 211" class="ar-green"/>
</g>
</svg>
<figcaption>Modules do the work; tasks call them; plays aim them at hosts; roles and collections package them.</figcaption>
</figure>

## Ansible and its relatives

Ansible is not the only tool of its kind. **Puppet** and **Chef** also manage configuration, but use agents on every node that pull their configuration from a server, and their own languages. **Salt** can do both. **Terraform** (and OpenTofu) is often used *with* Ansible: it creates infrastructure, the virtual machines, networks and cloud resources, while Ansible configures what runs on it. Shell scripts remain useful for small, one-off jobs, but they are imperative and not idempotent.

| Tool | Model | Agent on nodes | Language | Best at |
|---|---|---|---|---|
| Ansible | push, over SSH | no | YAML | configuration, deployment, orchestration |
| Puppet, Chef | pull from a server | yes | own DSL, Ruby | large fleets kept in a state continuously |
| Terraform | API calls to providers | no | HCL | creating cloud and virtual infrastructure |
| Shell scripts | run commands | no | Bash | one-off tasks on one machine |

## Versions and what's installed

Ansible comes in two layers. **ansible-core** is the engine and the built-in modules (`ansible.builtin.*`). The **ansible** package adds a curated set of **collections**, such as `community.general` and `ansible.posix`, with hundreds more modules. The DockerLab control node runs Debian 13 with its `ansible` package: ansible-core 2.19, with the collections, plus `ansible-lint`. The commands in this book were tested with exactly that combination.

```
$ ansible --version
ansible [core 2.19.x]
  config file = /home/student/northwind/ansible.cfg
  ...
  python version = 3.13.x
```

## The labs and the running story

The book is designed to be read alongside DockerLab's Ansible labs. Each lab deploys a small topology of containers: a **control** node, a Debian 13 machine with Ansible, and **managed nodes**, plain Debian 13 servers with Python 3 and an SSH server. You work in control's console; the managed nodes are yours to configure.

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A DockerLab Ansible lab: the control container at 10.0.0.10 and managed nodes web1 at 10.0.0.11, web2 at 10.0.0.12 and db1 at 10.0.0.21, all cabled to the lan switch, 10.0.0.0/24, on eth0. Each container also has a nat0 interface to the internet for installing packages." xmlns="http://www.w3.org/2000/svg">
<style>
.a1f{ font-family:var(--sans),sans-serif; }
.a1f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a1f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a1f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a1f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a1f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a1f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a1f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a1f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a1f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a1f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a1f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a1f .s{ fill:var(--text-dim); font-size:13px; }
.a1f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a1f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a1f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a1f .c{ fill:var(--text-faint); font-size:12.5px; }
.a1f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a1f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a1f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a1f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1f-a); }
.a1f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1f-a); marker-start:url(#a1f-s); }
.a1f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a1f-g); }
.a1f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a1f-a); }
.a1f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a1f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a1f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a1f .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a1f .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a1f .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1f-tri); }
.a1f .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1f-tri); }
.a1f .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1f-open); }
.a1f .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1f-open); }
.a1f .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1f-dia); }
.a1f .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1f-odia); }
.a1f .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1f-dia); marker-end:url(#a1f-open); }
.a1f .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1f-solid); }
.a1f .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a1f-open); }
.a1f .fill{ fill:var(--text); stroke:none; }
.a1f .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a1f .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a1f .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a1f .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a1f .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a1f .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a1f .ks-blue{ fill:#185FA5; font-size:13px; }
.a1f .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a1f .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1f .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a1f .kb-blue, :root[data-theme="dark"] .a1f .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a1f .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a1f .kh-blue{ fill:#85B7EB; }
.a1f .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a1f-blue); }
.a1f .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1f-blue); }
.a1f .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a1f .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a1f .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a1f .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a1f .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a1f .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1f .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a1f .kb-gray, :root[data-theme="dark"] .a1f .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a1f .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a1f .kh-gray{ fill:#B4B2A9; }
.a1f .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a1f-gray); }
.a1f .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1f-gray); }
.a1f .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a1f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a1f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a1f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a1f-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a1f-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a1f-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a1f-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a1f-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a1f-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a1f-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a1f">
<rect x="285" y="16" width="150" height="54" rx="7" class="dark"/>
<text x="360.0" y="39" text-anchor="middle" class="on">control</text>
<text x="360.0" y="57" text-anchor="middle" class="on" style="fill:var(--bg);font-weight:400">10.0.0.10</text>
<rect x="250" y="120" width="220" height="34" rx="17" class="k-gray"/>
<text x="360" y="142" text-anchor="middle" class="kb-gray">switch lan  10.0.0.0/24</text>
<line x1="360" y1="70" x2="360" y2="120" class="lk"/>
<text x="372" y="100" text-anchor="start" class="c">eth0</text>
<line x1="360" y1="154" x2="115" y2="210" class="lk"/>
<rect x="60" y="210" width="110" height="54" rx="7" class="k-blue"/>
<line x1="70" y1="219" x2="86" y2="219" class="ln-blue"/>
<line x1="70" y1="224" x2="86" y2="224" class="ln-blue"/>
<text x="123.0" y="232" text-anchor="middle" class="kb-blue">web1</text>
<text x="115.0" y="251" text-anchor="middle" class="ks-blue">10.0.0.11  student</text>
<line x1="360" y1="154" x2="360" y2="210" class="lk"/>
<rect x="305" y="210" width="110" height="54" rx="7" class="k-blue"/>
<line x1="315" y1="219" x2="331" y2="219" class="ln-blue"/>
<line x1="315" y1="224" x2="331" y2="224" class="ln-blue"/>
<text x="368.0" y="232" text-anchor="middle" class="kb-blue">web2</text>
<text x="360.0" y="251" text-anchor="middle" class="ks-blue">10.0.0.12  student</text>
<line x1="360" y1="154" x2="605" y2="210" class="lk"/>
<rect x="550" y="210" width="110" height="54" rx="7" class="k-blue"/>
<line x1="560" y1="219" x2="576" y2="219" class="ln-blue"/>
<line x1="560" y1="224" x2="576" y2="224" class="ln-blue"/>
<text x="613.0" y="232" text-anchor="middle" class="kb-blue">db1</text>
<text x="605.0" y="251" text-anchor="middle" class="ks-blue">10.0.0.21  student</text>
<rect x="20" y="16" width="230" height="54" rx="7" class="pnl"/>
<text x="135.0" y="38.5" text-anchor="middle" class="s">you work in control's console</text>
<text x="135.0" y="56.5" text-anchor="middle" class="c">double-click it in DockerLab</text>
<rect x="470" y="16" width="230" height="54" rx="7" class="pnl"/>
<text x="585.0" y="38.5" text-anchor="middle" class="s">every node: nat0, the internet</text>
<text x="585.0" y="56.5" text-anchor="middle" class="c">apt can install packages</text>
</g>
</svg>
<figcaption>The topology of DockerLab's Ansible labs 1 to 10.</figcaption>
</figure>

Things to know about the lab containers:

- You are **student** everywhere, with the password `student`. On the managed nodes, student may use `sudo` without a password, as automation accounts often can.
- The containers have no `systemd`. Services are started with the classic `service` command and init scripts, and Ansible's `service` module falls back to these automatically.
- Each container's lab cable is `eth0`. The containers also have an internet connection (`nat0`), so packages install with `apt`. When a playbook needs a host's lab address, it uses `ansible_facts['eth0']`.
- **Redeploy** gives you fresh servers. It also resets control, so keep your project: copy it out with *Files*, or keep it in Git.

The story follows Northwind through three stages, and each chapter adds a piece to one growing project folder, `~/northwind`:

<figure class="diagram">
<svg viewBox="0 0 720 400" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Northwind&#x27;s infrastructure in three stages. Stage one, chapters 2 to 11: web1, web2 and db1 for the coffee shop&#x27;s web site. Stage two, chapter 12: a load balancer in front of web1, web2 and web3 for the Orders application. Stage three, chapter 14: the load balancer, web1 and web2, and an internal API server api1, each with a firewall, plus in chapter 13 three FRR routers linking the company&#x27;s sites." xmlns="http://www.w3.org/2000/svg">
<style>
.a1g{ font-family:var(--sans),sans-serif; }
.a1g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.a1g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.a1g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.a1g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.a1g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.a1g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.a1g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.a1g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.a1g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.a1g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.a1g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.a1g .s{ fill:var(--text-dim); font-size:13px; }
.a1g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.a1g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.a1g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.a1g .c{ fill:var(--text-faint); font-size:12.5px; }
.a1g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.a1g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.a1g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.a1g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1g-a); }
.a1g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#a1g-a); marker-start:url(#a1g-s); }
.a1g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#a1g-g); }
.a1g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-a); }
.a1g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.a1g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.a1g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
.a1g .u{ stroke:var(--text-dim); stroke-width:1.6; fill:none; }
.a1g .ud{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; }
.a1g .gen{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1g-tri); }
.a1g .real{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-tri); }
.a1g .assoc{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1g-open); }
.a1g .dep{ stroke:var(--text-dim); stroke-width:1.6; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-open); }
.a1g .comp{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1g-dia); }
.a1g .agg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1g-odia); }
.a1g .compa{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-start:url(#a1g-dia); marker-end:url(#a1g-open); }
.a1g .msg{ stroke:var(--text-dim); stroke-width:1.6; fill:none; marker-end:url(#a1g-solid); }
.a1g .ret{ stroke:var(--text-dim); stroke-width:1.4; fill:none; stroke-dasharray:5 4; marker-end:url(#a1g-open); }
.a1g .fill{ fill:var(--text); stroke:none; }
.a1g .act{ fill:var(--panel); stroke:var(--text-dim); stroke-width:1.4; }
.a1g .stick{ stroke:var(--text); stroke-width:1.8; fill:none; }
.a1g .note{ fill:var(--panel-hover); stroke:var(--text-faint); stroke-width:1.2; }
.a1g .i{ fill:var(--text-dim); font-size:12.5px; font-style:italic; }
.a1g .k-blue{ fill:#E6F1FB; stroke:#378ADD; stroke-width:1.6; }
.a1g .kb-blue{ fill:#0C447C; font-size:13px; font-weight:700; }
.a1g .ks-blue{ fill:#185FA5; font-size:13px; }
.a1g .km-blue{ fill:#0C447C; font-size:12.5px; font-family:var(--mono),monospace; }
.a1g .kh-blue{ fill:#185FA5; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1g .k-blue{ fill:#0C447C; stroke:#85B7EB; }
:root[data-theme="dark"] .a1g .kb-blue, :root[data-theme="dark"] .a1g .km-blue{ fill:#E6F1FB; }
:root[data-theme="dark"] .a1g .ks-blue{ fill:#B5D4F4; }
:root[data-theme="dark"] .a1g .kh-blue{ fill:#85B7EB; }
.a1g .ar-blue{ stroke:#378ADD; stroke-width:2; fill:none; marker-end:url(#a1g-blue); }
.a1g .ad-blue{ stroke:#378ADD; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-blue); }
.a1g .ln-blue{ stroke:#378ADD; stroke-width:2; fill:none; }
.a1g .k-teal{ fill:#E1F5EE; stroke:#1D9E75; stroke-width:1.6; }
.a1g .kb-teal{ fill:#085041; font-size:13px; font-weight:700; }
.a1g .ks-teal{ fill:#0F6E56; font-size:13px; }
.a1g .km-teal{ fill:#085041; font-size:12.5px; font-family:var(--mono),monospace; }
.a1g .kh-teal{ fill:#0F6E56; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1g .k-teal{ fill:#085041; stroke:#5DCAA5; }
:root[data-theme="dark"] .a1g .kb-teal, :root[data-theme="dark"] .a1g .km-teal{ fill:#E1F5EE; }
:root[data-theme="dark"] .a1g .ks-teal{ fill:#9FE1CB; }
:root[data-theme="dark"] .a1g .kh-teal{ fill:#5DCAA5; }
.a1g .ar-teal{ stroke:#1D9E75; stroke-width:2; fill:none; marker-end:url(#a1g-teal); }
.a1g .ad-teal{ stroke:#1D9E75; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-teal); }
.a1g .ln-teal{ stroke:#1D9E75; stroke-width:2; fill:none; }
.a1g .k-amber{ fill:#FAEEDA; stroke:#BA7517; stroke-width:1.6; }
.a1g .kb-amber{ fill:#633806; font-size:13px; font-weight:700; }
.a1g .ks-amber{ fill:#854F0B; font-size:13px; }
.a1g .km-amber{ fill:#633806; font-size:12.5px; font-family:var(--mono),monospace; }
.a1g .kh-amber{ fill:#854F0B; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1g .k-amber{ fill:#633806; stroke:#EF9F27; }
:root[data-theme="dark"] .a1g .kb-amber, :root[data-theme="dark"] .a1g .km-amber{ fill:#FAEEDA; }
:root[data-theme="dark"] .a1g .ks-amber{ fill:#FAC775; }
:root[data-theme="dark"] .a1g .kh-amber{ fill:#EF9F27; }
.a1g .ar-amber{ stroke:#BA7517; stroke-width:2; fill:none; marker-end:url(#a1g-amber); }
.a1g .ad-amber{ stroke:#BA7517; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-amber); }
.a1g .ln-amber{ stroke:#BA7517; stroke-width:2; fill:none; }
.a1g .k-coral{ fill:#FAECE7; stroke:#D85A30; stroke-width:1.6; }
.a1g .kb-coral{ fill:#712B13; font-size:13px; font-weight:700; }
.a1g .ks-coral{ fill:#993C1D; font-size:13px; }
.a1g .km-coral{ fill:#712B13; font-size:12.5px; font-family:var(--mono),monospace; }
.a1g .kh-coral{ fill:#993C1D; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1g .k-coral{ fill:#712B13; stroke:#F0997B; }
:root[data-theme="dark"] .a1g .kb-coral, :root[data-theme="dark"] .a1g .km-coral{ fill:#FAECE7; }
:root[data-theme="dark"] .a1g .ks-coral{ fill:#F5C4B3; }
:root[data-theme="dark"] .a1g .kh-coral{ fill:#F0997B; }
.a1g .ar-coral{ stroke:#D85A30; stroke-width:2; fill:none; marker-end:url(#a1g-coral); }
.a1g .ad-coral{ stroke:#D85A30; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-coral); }
.a1g .ln-coral{ stroke:#D85A30; stroke-width:2; fill:none; }
.a1g .k-gray{ fill:#F1EFE8; stroke:#888780; stroke-width:1.6; }
.a1g .kb-gray{ fill:#444441; font-size:13px; font-weight:700; }
.a1g .ks-gray{ fill:#5F5E5A; font-size:13px; }
.a1g .km-gray{ fill:#444441; font-size:12.5px; font-family:var(--mono),monospace; }
.a1g .kh-gray{ fill:#5F5E5A; font-size:13px; font-weight:700; }
:root[data-theme="dark"] .a1g .k-gray{ fill:#444441; stroke:#B4B2A9; }
:root[data-theme="dark"] .a1g .kb-gray, :root[data-theme="dark"] .a1g .km-gray{ fill:#F1EFE8; }
:root[data-theme="dark"] .a1g .ks-gray{ fill:#D3D1C7; }
:root[data-theme="dark"] .a1g .kh-gray{ fill:#B4B2A9; }
.a1g .ar-gray{ stroke:#888780; stroke-width:2; fill:none; marker-end:url(#a1g-gray); }
.a1g .ad-gray{ stroke:#888780; stroke-width:2; fill:none; stroke-dasharray:6 4; marker-end:url(#a1g-gray); }
.a1g .ln-gray{ stroke:#888780; stroke-width:2; fill:none; }
</style>
<defs><marker id="a1g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="a1g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="a1g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker><marker id="a1g-tri" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="12" markerHeight="12" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.3"/></marker><marker id="a1g-open" viewBox="0 0 12 12" refX="11" refY="6" markerWidth="11" markerHeight="11" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 1 L11 6 L1 11" fill="none" stroke="var(--text-dim)" stroke-width="1.4"/></marker><marker id="a1g-solid" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="9" markerHeight="9" markerUnits="userSpaceOnUse" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-dim)"/></marker><marker id="a1g-dia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--text-dim)"/></marker><marker id="a1g-odia" viewBox="0 0 16 10" refX="1" refY="5" markerWidth="16" markerHeight="10" markerUnits="userSpaceOnUse" orient="auto"><path d="M1 5 L8 1 L15 5 L8 9 z" fill="var(--bg)" stroke="var(--text-dim)" stroke-width="1.2"/></marker><marker id="a1g-blue" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#378ADD"/></marker><marker id="a1g-teal" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#1D9E75"/></marker><marker id="a1g-amber" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#BA7517"/></marker><marker id="a1g-coral" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#D85A30"/></marker><marker id="a1g-gray" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="#888780"/></marker></defs>
<g class="a1g">
<rect x="20" y="16" width="220" height="368" rx="12" class="k-gray"/>
<text x="130" y="40" text-anchor="middle" class="kb-gray">chapters 2 to 11</text>
<text x="130" y="58" text-anchor="middle" class="ks-gray">the coffee shop site</text>
<rect x="260" y="16" width="220" height="368" rx="12" class="k-gray"/>
<text x="370" y="40" text-anchor="middle" class="kb-gray">chapter 12</text>
<text x="370" y="58" text-anchor="middle" class="ks-gray">Northwind Orders</text>
<rect x="500" y="16" width="220" height="368" rx="12" class="k-gray"/>
<text x="610" y="40" text-anchor="middle" class="kb-gray">chapter 14</text>
<text x="610" y="58" text-anchor="middle" class="ks-gray">the secure stack</text>
<rect x="75" y="80" width="110" height="40" rx="7" class="k-blue"/>
<line x1="85" y1="89" x2="101" y2="89" class="ln-blue"/>
<line x1="85" y1="94" x2="101" y2="94" class="ln-blue"/>
<text x="138.0" y="102" text-anchor="middle" class="kb-blue">web1</text>
<rect x="75" y="140" width="110" height="40" rx="7" class="k-blue"/>
<line x1="85" y1="149" x2="101" y2="149" class="ln-blue"/>
<line x1="85" y1="154" x2="101" y2="154" class="ln-blue"/>
<text x="138.0" y="162" text-anchor="middle" class="kb-blue">web2</text>
<rect x="75" y="200" width="110" height="40" rx="7" class="k-amber"/>
<line x1="85" y1="209" x2="101" y2="209" class="ln-amber"/>
<line x1="85" y1="214" x2="101" y2="214" class="ln-amber"/>
<text x="138.0" y="222" text-anchor="middle" class="kb-amber">db1</text>
<rect x="315" y="80" width="110" height="40" rx="7" class="k-teal"/>
<line x1="325" y1="89" x2="341" y2="89" class="ln-teal"/>
<line x1="325" y1="94" x2="341" y2="94" class="ln-teal"/>
<text x="378.0" y="102" text-anchor="middle" class="kb-teal">lb</text>
<rect x="272" y="190" width="60" height="40" rx="7" class="k-blue"/>
<text x="302.0" y="214.5" text-anchor="middle" class="kb-blue">web1</text>
<path d="M370 120 L302 186" class="ar-teal"/>
<rect x="340" y="190" width="60" height="40" rx="7" class="k-blue"/>
<text x="370.0" y="214.5" text-anchor="middle" class="kb-blue">web2</text>
<path d="M370 120 L370 186" class="ar-teal"/>
<rect x="408" y="190" width="60" height="40" rx="7" class="k-blue"/>
<text x="438.0" y="214.5" text-anchor="middle" class="kb-blue">web3</text>
<path d="M370 120 L438 186" class="ar-teal"/>
<rect x="555" y="80" width="110" height="54" rx="7" class="k-teal"/>
<line x1="565" y1="89" x2="581" y2="89" class="ln-teal"/>
<line x1="565" y1="94" x2="581" y2="94" class="ln-teal"/>
<text x="618.0" y="102" text-anchor="middle" class="kb-teal">lb</text>
<text x="610.0" y="121" text-anchor="middle" class="ks-teal">firewall</text>
<rect x="520" y="160" width="85" height="54" rx="7" class="k-blue"/>
<line x1="530" y1="169" x2="546" y2="169" class="ln-blue"/>
<line x1="530" y1="174" x2="546" y2="174" class="ln-blue"/>
<text x="570.5" y="182" text-anchor="middle" class="kb-blue">web1</text>
<text x="562.5" y="201" text-anchor="middle" class="ks-blue">firewall</text>
<rect x="615" y="160" width="85" height="54" rx="7" class="k-blue"/>
<line x1="625" y1="169" x2="641" y2="169" class="ln-blue"/>
<line x1="625" y1="174" x2="641" y2="174" class="ln-blue"/>
<text x="665.5" y="182" text-anchor="middle" class="kb-blue">web2</text>
<text x="657.5" y="201" text-anchor="middle" class="ks-blue">firewall</text>
<rect x="555" y="240" width="110" height="54" rx="7" class="k-coral"/>
<line x1="565" y1="249" x2="581" y2="249" class="ln-coral"/>
<line x1="565" y1="254" x2="581" y2="254" class="ln-coral"/>
<text x="618.0" y="262" text-anchor="middle" class="kb-coral">api1</text>
<text x="610.0" y="281" text-anchor="middle" class="ks-coral">firewall</text>
<path d="M610 136 L563 156" class="ar-teal"/>
<path d="M610 136 L657 156" class="ar-teal"/>
<path d="M563 216 L590 236" class="ar-coral"/>
<path d="M657 216 L630 236" class="ar-coral"/>
<text x="610" y="320" text-anchor="middle" class="s">+ chapter 13: routers r1, r2, r3</text>
<text x="610" y="340" text-anchor="middle" class="c">linking Northwind's sites</text>
<text x="130" y="290" text-anchor="middle" class="c">playbooks, templates,</text>
<text x="130" y="308" text-anchor="middle" class="c">handlers, roles, vault</text>
<text x="370" y="340" text-anchor="middle" class="c">rolling updates</text>
<text x="370" y="358" text-anchor="middle" class="c">with no downtime</text>
</g>
</svg>
<figcaption>One company, one growing Ansible project: Northwind from three servers to a secured, routed stack.</figcaption>
</figure>

| Chapters | Northwind's infrastructure | DockerLab labs |
|---|---|---|
| 2 to 11 | Two web servers and a database server: the coffee shop's web site, built up from ad-hoc commands to roles with encrypted secrets | Ansible labs 1 to 10 |
| 12 | A load balancer and three web servers: the new *Northwind Orders* application, updated with no downtime | Ansible lab 11 |
| 13 | Three FRR routers linking Northwind's sites, configured and verified from data | the routing lab in [`examples/ch13`](https://github.com/khaledmahmudkm/ansiblehandson/tree/main/examples) |
| 14 | The final stack: load balancer, web servers, an internal API, hardened SSH and host firewalls, with a verification playbook | Ansible lab 12 |

The project as it stands at the end of each chapter is in the [`examples/` folder](https://github.com/khaledmahmudkm/ansiblehandson/tree/main/examples) of this book's repository. If you start reading in the middle, deploy the chapter's lab, give control key login to the servers, and copy that chapter's folder to `~/northwind`.

> **Ansible changes real systems.** Everything in this book runs inside lab containers, where a mistake costs a Redeploy. On real servers, use `--check` before every change, keep playbooks in version control, and test on one host before all of them.

## Summary

- Configuring servers by hand causes repetition, drift, and changes nobody can review or repeat. **Infrastructure as code** describes servers in versioned text files.
- Ansible is **agentless** (SSH and Python on the nodes, nothing installed), **push-based** (you run it from a **control node**), and **declarative** (you describe the desired state).
- For each task, Ansible copies a **module** to the host, runs it, reads its JSON result and cleans up.
- **Idempotency**: tasks check before changing, so a playbook can run again and again safely, and reports only real changes.
- The building blocks nest: modules in tasks, tasks in plays, plays in playbooks, all organized into roles and shared as collections, with an inventory saying where they run.
- The book's running story, Northwind, grows chapter by chapter in one project, `~/northwind`, on DockerLab.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#chapter-1-how-ansible-works).

**1. What is configuration drift?**

- A. Servers slowly becoming different through manual changes
- B. Ansible versions differing between control nodes
- C. Network latency between servers
- D. A YAML indentation error

**2. What does agentless mean for Ansible?**

- A. Ansible runs without a control node
- B. Ansible needs no inventory
- C. Nothing is installed on managed nodes; Ansible uses SSH and the node's Python
- D. Playbooks run without modules

**3. What does a managed node need for Ansible to work?**

- A. Ansible installed
- B. An SSH server, Python 3 and an account Ansible can log in to
- C. A Puppet agent
- D. A web server

**4. For each task on each host, what does Ansible copy to the host?**

- A. The whole playbook
- B. The inventory
- C. An agent that stays installed
- D. The module, packed with its arguments as one Python file (AnsiballZ)

**5. What is idempotency?**

- A. Running tasks in parallel
- B. Running an operation again changes nothing that is already in the desired state
- C. Encrypting secrets
- D. Running tasks in a fixed order

**6. Which tool is typically used with Ansible to create cloud infrastructure, while Ansible configures it?**

- A. Puppet
- B. Chef
- C. Salt
- D. Terraform

**7. What does the ansible package add on top of ansible-core?**

- A. A curated set of collections such as community.general and ansible.posix
- B. A web interface
- C. An agent for managed nodes
- D. A YAML editor

**8. In DockerLab, how does the service module start services in the lab containers?**

- A. Through systemd
- B. It cannot start services
- C. It falls back to the service command and init scripts, because the containers have no systemd
- D. Through Docker restart policies
