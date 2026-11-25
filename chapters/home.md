# Concise Book on Ansible, Hands-on Edition

A concise, diagram-rich guide to automating servers and networks with Ansible, built around one growing project and tested on DockerLab.

<img src="cover.svg" alt="Cover Image" width="100%" >

Configuring one server by hand is easy. Configuring ten the same way, keeping them the same for years, changing them safely while users depend on them, and proving they are right, is a different job. **Ansible** does that job: you describe what your machines should look like in plain YAML files, and Ansible connects over SSH and makes it so, idempotently, with nothing installed on the machines.

This book covers Ansible from the first `ping` to a hardened, verified production-style stack: the control node and managed nodes, ad-hoc commands and modules, inventories, playbooks, variables and facts, Jinja2 templates, conditions, loops and error handling, handlers and tags, roles and collections, Vault, rolling updates behind a load balancer, network automation with FRR routers, and a final project with SSH hardening, host firewalls and a verification playbook. It is written for third- and fourth-year undergraduates who know basic Linux.

> **Two Ansible books.** This hands-on edition follows one project through DockerLab's labs, with every command tested. For a shorter overview of Ansible, see the [Concise Book on Ansible](https://khaledmahmudkm.github.io/conciseansible/).

---

## The running example: Northwind

Every chapter adds to one project for one company. **Northwind** starts as a coffee shop with a web site on two web servers and a database server, configured by hand. Chapter by chapter, its operations team (you) automates everything: the servers' baseline, the web site and its templates, the team's accounts, encrypted application secrets, then a load-balanced *Northwind Orders* application updated with no downtime, routers linking its sites, and finally a secured stack with an internal API, firewalls and automated verification.

The project as it stands at the end of each chapter is in the [`examples/` folder](https://github.com/khaledmahmudkm/ansiblehandson/tree/main/examples) of this book's repository, and every command and playbook in the book was run against the lab topology it describes.

---

## The labs: DockerLab

The book is designed to be read with **DockerLab**, a Docker-based lab platform. Its Ansible labs deploy a **control** node with Ansible (Debian 13, ansible-core 2.19) and plain Debian 13 **managed nodes**, so every example runs exactly as printed:

| Chapters | DockerLab lab |
|---|---|
| 2 to 11 | Ansible labs 1 to 10: control, web1, web2, db1 |
| 12 | Ansible lab 11: control, lb, web1, web2, web3 |
| 13 | the routing lab in `examples/ch13`: control and three FRR routers |
| 14 | Ansible lab 12: control, lb, web1, web2, api1 |

> Everything here runs inside lab containers. Before running playbooks on real servers, read Chapter 5's advice: check, diff, one host first.

---

## How this book is organized

**Part I, Foundations.** How Ansible works; first contact with key-based SSH, an inventory and ad-hoc commands; modules and idempotency; inventories and project configuration.

**Part II, Playbooks.** Playbooks; variables and facts; Jinja2 templates; conditions, loops and error handling; handlers and tags.

**Part III, Reuse and Security.** Roles and collections; Vault and secrets.

**Part IV, Orchestration.** Rolling updates with no downtime; network automation with FRR and OSPF; the final project, a secure web stack.

Each chapter ends with **Check your understanding**, eight multiple-choice questions. The **Appendix** has a quick reference, a **Glossary**, and **Answers to Chapter Questions** with every answer and a short explanation.

---

## How to read it

Read the chapters in order: each builds on the project of the one before. Each chapter starts with the DockerLab lab to deploy, and ends with **Hands on** exercises and a summary. Diagrams use colour for meaning: blue for web servers, teal for load balancers and routers, coral for the API, purple for the control side and roles, amber for data, green for success and red for failures and secrets.

---

© 2026 Khaled Mahmud
