# Examples: the Northwind project, chapter by chapter

Each folder is the project as it stands at the **end** of that chapter of the *Concise Book on Ansible, Hands-on Edition*. Every playbook here was run against the DockerLab topology the chapter names.

| Folder | Chapter | DockerLab lab | Hosts |
|---|---|---|---|
| `ch04` | Inventory and configuration | Ansible lab 3 | web1, web2, db1 |
| `ch05` | Playbooks | Ansible lab 4 | web1, web2, db1 |
| `ch06` | Variables and facts | Ansible lab 5 | web1, web2, db1 |
| `ch07` | Templates | Ansible lab 6 | web1, web2, db1 |
| `ch08` | Conditions, loops and errors | Ansible lab 7 | web1, web2, db1 |
| `ch09` | Handlers and tags | Ansible lab 8 | web1, web2, db1 |
| `ch10` | Roles and collections | Ansible lab 9 | web1, web2, db1 |
| `ch11` | Vault and secrets | Ansible lab 10 | web1, web2, db1 |
| `ch12` | Rolling updates | Ansible lab 11 | lb, web1, web2, web3 |
| `ch13` | Network automation with FRR | `ch13/ansible-13-frr-ospf.json` | r1, r2, r3 |
| `ch14` | Final project: a secure web stack | Ansible lab 12 | lb, web1, web2, api1 |

Chapters 2 and 3 use ad-hoc commands only.

## Using a folder

In control's console, after deploying the lab:

```
setup-keys web1 web2 db1                 # the hosts of that lab
git clone https://github.com/khaledmahmudkm/ansiblehandson /tmp/book
cp -r /tmp/book/examples/ch07 ~/northwind && cd ~/northwind
ansible all -m ping
ansible-playbook site.yml
```

## Vault password

`ch11/group_vars/web/vault.yml` and `ch12/group_vars/web/vault.yml` are encrypted with the vault password **`Lab-Vault-2026`**, a lab-only password published here on purpose. Their `ansible.cfg` reads it from `~/.vault_pass`:

```
printf 'Lab-Vault-2026\n' > ~/.vault_pass && chmod 600 ~/.vault_pass
```

Never publish a real vault password, and never commit a password file.

## The routing lab

`ch13/ansible-13-frr-ospf.json` is a DockerLab lab file: import it, Deploy, then follow its instructions or Chapter 13. It uses the Ansible labs' `ansible-node` image for the routers, with *router* and *privileged* on; Ansible installs FRR.
