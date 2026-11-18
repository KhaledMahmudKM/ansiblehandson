# Glossary

**Ad-hoc command.** A single module run on a set of hosts from the command line with `ansible`, without a playbook.

**Agentless.** Needing no software installed on managed machines: Ansible uses SSH and the machine's Python.

**ansible.cfg.** The configuration file Ansible reads from the current folder, the home folder or `/etc/ansible`; the first found wins.

**ansible-lint.** A tool that checks playbooks and roles against good practice, with profiles from `min` to `production`.

**ansible_managed.** A variable holding a standard comment that marks a templated file as managed by Ansible.

**AnsiballZ.** The packed Python file Ansible copies to a host to run a module.

**Assert.** A module that checks conditions and stops the host with a message when one is false.

**AWX.** The open source web interface and API for running Ansible on schedules with credentials and logs; upstream of Ansible Automation Platform.

**Become.** Running a task as another user, usually root, through `sudo`.

**Block.** A group of tasks sharing keywords, with optional `rescue` and `always` sections for error handling.

**Canary.** A first, small batch of a rollout, used to test a change on one server before the rest.

**Check mode.** A dry run (`--check`) in which modules report what they would change without changing it.

**Collection.** A distribution format for Ansible content, named `namespace.collection`, holding modules, roles and plugins.

**Control node.** The machine Ansible runs on, holding the inventory and playbooks.

**Convergence.** In routing, the point where every router agrees on the network's routes; in configuration management, bringing systems to the desired state.

**Default (role).** A variable in a role's `defaults/main.yml`, with the lowest precedence, meant to be overridden.

**Delegation.** Running a task on another host (`delegate_to`) while using the current host's variables.

**Desired state.** The description of how a system should be, which modules compare with how it is.

**Diff mode.** Showing the changes a task makes to files (`--diff`).

**Drift.** Differences that accumulate between systems, or between a system and its description, through manual changes.

**Dynamic inventory.** An inventory built at run time from a source such as a cloud API, by a plugin or script.

**Extra variables.** Variables given with `-e` on the command line; they override all others.

**Facts.** Information gathered from each host by the `setup` module: system, hardware, network and more.

**Filter.** A Jinja2 function applied with `|` that transforms a value, such as `default`, `join` or `to_nice_yaml`.

**Forks.** The number of hosts Ansible works on in parallel (default 5).

**FQCN (fully qualified collection name).** A module's full name, such as `ansible.builtin.copy`.

**FRR (FRRouting).** An open source routing suite for Linux, with daemons for OSPF, BGP and other protocols.

**Group.** A named set of hosts in the inventory; groups can contain groups.

**group_vars, host_vars.** Folders of YAML variables for each group and each host, beside the inventory.

**Handler.** A task that runs only when notified by a changed task, once, at the end of the play.

**Health check.** A request, such as to `/health`, that tells whether a service works.

**Host pattern.** An expression selecting hosts: a group, a host, wildcards, unions (`:`), intersections (`:&`), exclusions (`:!`).

**hostvars.** A magic variable holding every host's variables and facts.

**Idempotency.** The property that running an operation again changes nothing that is already in the desired state.

**Infrastructure as code.** Describing infrastructure in versioned, reviewable text files that tools apply.

**Inventory.** The list of managed hosts and their groups and variables, in INI or YAML, or built dynamically.

**Jinja2.** The templating language Ansible uses for variables, expressions and templates.

**Least privilege.** Giving each user, process or host only the access its job needs.

**Load balancer.** A server that shares requests among several backends.

**Lookup.** A plugin that reads data on the control node, such as a file, an environment variable or a secret store.

**Loop.** Repeating a task for each item of a list, with the item in `item`.

**Magic variables.** Variables Ansible defines itself, such as `inventory_hostname`, `group_names`, `groups` and `hostvars`.

**Managed node.** A machine Ansible configures.

**Module.** A unit of work Ansible copies to a host and runs, such as `apt`, `copy` or `service`.

**network_cli.** A connection type that sends CLI commands to network devices over SSH, without Python on the device.

**no_log.** A task keyword that hides its arguments and results from the output.

**Notify.** A task keyword naming the handler to trigger when the task changes something.

**OSPF.** A link-state routing protocol in which routers flood link information and compute shortest paths.

**Play.** A mapping of hosts to tasks and roles, with keywords such as `become` and `serial`.

**Playbook.** A YAML file containing one or more plays.

**Precedence.** The order that decides which value wins when a variable is set in several places.

**Register.** Saving a task's result in a variable.

**Resource module.** A network module that manages one resource, such as interfaces or OSPF, from structured data.

**Role.** A reusable package of tasks, handlers, templates, files, variables and metadata for one job.

**Rolling update.** Updating servers in batches, removing each from service while it changes.

**Serial.** A play keyword that runs the play on batches of hosts in turn.

**site.yml.** The conventional name of the playbook that configures the whole infrastructure.

**Source of truth.** The authoritative system holding the data that describes the infrastructure, such as Git or NetBox.

**Tag.** A label on tasks, blocks, roles or plays that lets `--tags` and `--skip-tags` select them.

**Task.** One call to a module, with a name and parameters.

**Template.** A Jinja2 file rendered with a host's variables and written to the host.

**Validate.** A `copy` and `template` parameter that checks a new file with a command before it replaces the old one.

**Vault.** Ansible's feature for encrypting files and values with AES-256.

**YAML.** The data format of playbooks, inventories and variables: lists, dictionaries and scalars, nested by indentation.
