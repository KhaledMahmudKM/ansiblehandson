# Concise Book on Ansible, Hands-on Edition

A concise, diagram-rich guide to Ansible for third- and fourth-year undergraduates: the control node and managed nodes, ad-hoc commands and modules, inventories, playbooks, variables and facts, Jinja2 templates, conditions, loops and error handling, handlers and tags, roles and collections, Vault, rolling updates, network automation with FRR, and a hardened, verified web stack.

**Read it online:** https://khaledmahmudkm.github.io/ansiblehandson/

The hands-on companion to the shorter [Concise Book on Ansible](https://github.com/khaledmahmudkm/conciseansible): one project, followed lab by lab, with every command tested.

## The running example: Northwind

Every chapter adds to one project for one company, Northwind: from a coffee shop's web site on two web servers and a database server, to a load-balanced *Northwind Orders* application, three routers linking its sites, and a final stack with an internal API, SSH hardening, host firewalls and a verification playbook.

The [`examples/`](examples/) folder holds the project as it stands at the end of each chapter. Every command and playbook was run against the DockerLab topology the chapter names (ansible-core 2.19, as on DockerLab's Debian 13 control node).

## How this repository works

* `chapters/` holds one Markdown file per chapter, with diagrams embedded as SVG, and `chapters.json` for the table of contents.
* `book.json` holds the title, author and palette (`harbor`).
* `index.html` loads the shared reader, [bookengine](https://github.com/khaledmahmudkm/bookengine) 1.0.6, from jsDelivr. When opened from a local web server, it uses `../bookengine-git/` instead if that folder exists.
* `examples/` holds the Northwind project, chapter by chapter, and the DockerLab routing lab for Chapter 13.

## Licence

Text and diagrams © 2026 Khaled Mahmud. The example playbooks may be reused freely.
