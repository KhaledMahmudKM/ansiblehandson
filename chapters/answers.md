# Answers to Chapter Questions

Answers and short explanations for the **Check your understanding** questions at the end of each chapter. Try the questions first; each section links back to them.

## Chapter 1: How Ansible Works

**1. A. Servers slowly becoming different through manual changes**

A quick fix on one server that never reaches the others makes identical servers subtly different.

**2. C. Nothing is installed on managed nodes; Ansible uses SSH and the node's Python**

No daemon to install, patch or monitor on the managed nodes.

**3. B. An SSH server, Python 3 and an account Ansible can log in to**

Ansible itself lives only on the control node.

**4. D. The module, packed with its arguments as one Python file (AnsiballZ)**

It runs the module with the host's Python, reads its JSON result and removes the temporary folder.

**5. B. Running an operation again changes nothing that is already in the desired state**

It makes playbooks safe to run repeatedly, and turns them into drift detectors.

**6. D. Terraform**

Terraform creates VMs, networks and cloud resources; Ansible configures what runs on them.

**7. A. A curated set of collections such as community.general and ansible.posix**

ansible-core is the engine and the ansible.builtin modules.

**8. C. It falls back to the service command and init scripts, because the containers have no systemd**

The module detects the missing systemd and uses the classic init scripts transparently.

[Back to the questions](01-how-ansible-works.md#check-your-understanding)

## Chapter 2: First Contact

**1. D. It logs in to every host many times per run, so typing passwords is impractical**

A key pair lets Ansible log in non-interactively, as often as it needs.

**2. B. ~/.ssh/authorized_keys**

ssh-copy-id appends your public key there; the private key never leaves control.

**3. D. all and ungrouped**

all contains every host; ungrouped holds hosts in no other group.

**4. A. Name resolution, SSH, the key and Python on the host all work**

It is not an ICMP ping: Ansible logs in and runs a small Python program.

**5. C. command**

command runs a program on each host; it always reports CHANGED because it can't know.

**6. D. all:!db**

Quote it so the shell leaves the ! alone.

**7. B. How many hosts Ansible works on in parallel (default 5)**

With -f 1 hosts run one after another; with -f 3 three run at once.

**8. C. Runs the module as another user, root by default, through sudo**

Log in as an ordinary user, become root only where needed; -K asks for a sudo password.

[Back to the questions](02-first-contact.md#check-your-understanding)

## Chapter 3: Modules and Idempotency

**1. A. command runs a program without a shell; shell runs the line in /bin/sh, so pipes work**

Prefer command, or better a dedicated module; both always report changed.

**2. C. SUCCESS / ok, with changed false**

The file is already as described, so nothing is done: idempotency.

**3. D. Report what would change, with diffs, without changing anything**

A dry run with diffs, which makes Ansible a drift detector.

**4. B. state=absent**

absent deletes the path, recursively for a folder.

**5. C. To run apt update only when the cache is older than that many seconds, keeping reruns fast**

Without it, every run would refresh the package lists.

**6. A. The group list replaces the user's current extra groups**

Like usermod -G versus usermod -aG.

**7. A. Adds the line at the end of the file**

If a line matches, the last match is replaced; otherwise the line is appended.

**8. C. stat**

stat only reports; file would create or change the path.

[Back to the questions](03-modules.md#check-your-understanding)

## Chapter 4: Inventory and Configuration

**1. B. ./ansible.cfg, in the project folder**

Only the first one found is read: $ANSIBLE_CONFIG, ./ansible.cfg, ~/.ansible.cfg, /etc/ansible/ansible.cfg.

**2. C. Settings that differ from the defaults, and where each came from**

It names the source: the file or an environment variable.

**3. A. A group whose members are other groups**

Everything in the listed groups is in linux.

**4. A. web:&production**

:& is intersection; : is union; :! is exclusion.

**5. C. host_vars/web2.yml**

The more specific level wins, and the host is the most specific.

**6. B. Gives the address to connect to, keeping the inventory name**

The host keeps its name, db1, in patterns and variables.

**7. D. Hosts web1 and web2**

A range; web[01:20] would be twenty hosts.

**8. B. A plugin or script that builds the host list at run time from a source such as a cloud API**

It outputs the same JSON as ansible-inventory --list; the rest of the project does not change.

[Back to the questions](04-inventory.md#check-your-understanding)

## Chapter 5: Playbooks

**1. A. A list of plays; each play targets hosts and lists tasks, each calling one module**

Playbook, play, task, module: each contains the next.

**2. C. Unquoted, YAML reads it as a number, which can turn into a different mode**

Quote modes like "0750", and values that start with {{.

**3. B. Each task runs on all hosts of the play before the next task starts**

That is why output is grouped by task.

**4. D. The playbook is idempotent: the servers already match the description**

A second run that reports changes usually hides a command or shell task that should be a module.

**5. B. Parses the playbook without connecting to any host**

Use --check --diff to preview changes on the hosts.

**6. D. Preview with --check --diff, apply to one host with --limit, look, then apply to all**

Look before you leap; each step is cheap.

**7. A. db1's remaining tasks are skipped; other hosts carry on**

A failure stops that host, not the play.

**8. C. A literal block: the indented lines below are kept exactly, newlines included**

>- would fold the lines into one.

[Back to the questions](05-playbooks.md#check-your-understanding)

## Chapter 6: Variables and Facts

**1. D. A value starting with {{ would otherwise be read by YAML as a dictionary**

Quote any value that starts with {{.

**2. B. Extra variables given with -e**

Handy for one-off runs, risky as a habit because nothing records them.

**3. D. ansible_facts['eth0']['ipv4']['address']**

The default route goes out through nat0, so default_ipv4 is the internet interface.

**4. A. The task's whole result: stdout, stdout_lines, rc, changed, failed and more**

Later tasks read fields such as disk.stdout_lines.

**5. C. So the recap stays honest: otherwise the task always counts as changed**

command cannot know whether it changed anything.

**6. D. db1's variables and facts**

Its facts are present only if they were gathered earlier in the run.

**7. B. To gather every host's facts, so hostvars['db1'] has them later**

The second play targets only web servers and would not gather db1's facts.

**8. C. http-port**

A hyphen is subtraction in Jinja2; use lowercase and underscores.

[Back to the questions](06-variables-facts.md#check-your-understanding)

## Chapter 7: Templates

**1. A. On the control node**

It renders on control with each host's variables and facts, then writes the result to the host.

**2. C. {{ }} prints an expression; {% %} holds logic such as if, for and set**

{# #} marks a comment that is not copied to the output.

**3. D. {% for day, hours in opening_hours.items() %}**

.items() gives key and value pairs.

**4. B. Removes the newline right after a {% %} block tag**

So {% for %} and {% endfor %} lines leave no empty lines behind; {%- and -%} strip more.

**5. C. The other hosts' facts were not gathered in this run**

Gather the missing facts with delegate_facts, or use fact caching.

**6. A. Run the task once, on control, with the first host's variables**

That is how report.yml writes one fleet report on control.

**7. A. Checks the new file before it replaces the old one; on failure the real file is untouched**

%s is the new file's temporary name.

**8. C. As data in group_vars, with the template only describing the form**

Change the data, not the files: the template stays the same.

[Back to the questions](07-templates.md#check-your-understanding)

## Chapter 8: Conditions, Loops and Errors

**1. B. As a Jinja2 expression without braces**

when: "'web' in group_names"; a list of conditions must all be true.

**2. C. For each item on each host**

That is how bob is skipped on db1 and carol on the web servers.

**3. A. Removes dave again, with his home folder (remove: true)**

The list is the truth; leavers disappear from every server.

**4. A. Turns a dictionary into a list of key and value items, for loop**

loop takes lists; item.key and item.value give the entries.

**5. C. One transaction is faster than calling apt once per package**

Don't loop where a module takes a list.

**6. B. failed_when: errors.rc > 1**

failed_when replaces Ansible's rule: only exit codes above 1 are real errors.

**7. D. rescued=1 and failed=0**

The failure was handled; always tasks run either way.

**8. B. Running on unsupported hosts or invalid settings, failing before anything changes**

Better to stop with a clear message than fail halfway through a change.

[Back to the questions](08-conditions-loops-errors.md#check-your-understanding)

## Chapter 9: Handlers and Tags

**1. A. Once, at the end of the play, only if a notifying task changed something**

Several notifications still run the handler once.

**2. C. Nothing: no change, no notification, no reload**

Users are interrupted only when something actually changed.

**3. B. Reload rereads the configuration gracefully with the same master PID; restart stops and starts, dropping connections**

Prefer reload where the service and the change allow it.

**4. D. Answer one notification topic with several handlers, in order**

Northwind checks the configuration with nginx -t, then reloads.

**5. B. So a changed configuration is applied before the check, instead of at the end of the play**

Otherwise the check would test the old configuration.

**6. D. The handler never runs: the file and the running service disagree, and the next run won't notice**

--force-handlers or force_handlers = True prevents it.

**7. A. Tasks tagged content, plus those tagged always**

Fact gathering also runs.

**8. C. Runs a task only when one of its other tags is asked for by name**

Right for dangerous jobs, such as tags: [never, purge] run with --tags purge.

[Back to the questions](09-handlers-tags.md#check-your-understanding)

## Chapter 10: Roles and Collections

**1. D. defaults/main.yml**

defaults have the lowest priority, so group_vars and host_vars override them; vars holds internal constants.

**2. B. So variables from different roles never clash; ansible-lint checks it**

Roles from many authors end up in one project.

**3. D. The role's own templates/ folder**

Each part of a role is found by its folder name.

**4. A. Through host_vars/web2.yml, which overrides the role defaults**

Data stays outside the role; that is what makes it reusable.

**5. C. webserver's defaults are loaded only in plays that use the webserver role**

In the Every server play, web1 has no webserver_port of its own.

**6. D. include_role is dynamic, decided at run time, so it works in loops and under when; import_role is static**

roles: at play level runs before the play's tasks.

**7. B. A distribution format, namespace.collection, for modules, roles and plugins**

List the ones a project needs in requirements.yml.

**8. C. The project passes the most demanding set of good-practice rules**

Profiles range from min to production.

[Back to the questions](10-roles.md#check-your-understanding)

## Chapter 11: Vault and Secrets

**1. A. They spread to every clone, backup and screen share, and Git keeps them in history**

Secrets must be encrypted in the files from the start.

**2. C. Ordinary values plus references such as appconfig_db_password: "{{ vault_db_password }}"**

Anyone can see which secrets exist and where they are used, without reading them.

**3. D. AES-256**

The key is derived from the vault password, and an HMAC protects against tampering.

**4. B. ansible-vault edit**

edit decrypts into the editor and re-encrypts on save.

**5. C. Inside the project, where it would be committed**

ansible.cfg points to ~/.vault_pass, outside the project.

**6. A. Hides the task's arguments and results from the output**

Encryption protects files, not output; no_log protects the output.

**7. A. A random salt would give a new hash every run, so the user task would always report changed**

The salt is kept in the vault too.

**8. C. Rekey the vault and change the secrets themselves**

Rekeying protects the file, not secrets that may already have been read.

[Back to the questions](11-vault.md#check-your-understanding)

## Chapter 12: Rolling Updates

**1. B. So the rollout can take out exactly one server by changing only its own file**

force: false creates each file once and leaves later changes to the rollout.

**2. C. A group and a host with the same name confuse Ansible**

Ansible warns about it.

**3. A. Runs the whole play on one host at a time**

The PLAY line appears once per batch.

**4. A. Stops the whole rollout as soon as any host in a batch fails**

A bad version never spreads past the first failing server.

**5. C. On lb, with web2's variables**

So inventory_hostname still names web2, and the task writes web2.conf on lb.

**6. B. A background loop requesting the page through lb, which got 200 for every request**

Watching from the users' side is the real test.

**7. D. Marked down in the load balancer, because its post_tasks never ran**

Customers keep being served by the other servers.

**8. B. A canary: one server first, then all the rest at once**

The first server proves the version before the rest follow.

[Back to the questions](12-rolling-updates.md#check-your-understanding)

## Chapter 13: Network Automation with FRR

**1. A. So a routing mistake cannot cut off Ansible's own access: management is out of band**

OSPF must never run on eth0.

**2. C. It stays up whatever happens to the links, and doubles as the OSPF router ID**

The verification pings go from loopback to loopback.

**3. B. Like servers: SSH and Python, apt, lineinfile, a template and the service module**

FRR runs on Linux, so the familiar modules apply.

**4. D. /etc/frr/daemons and /etc/frr/frr.conf**

daemons chooses which daemons start; frr.conf holds the configuration.

**5. B. A typo in frr.conf replacing the running configuration**

FRR parses the new file before it is put in place.

**6. D. OSPF needs time to converge; neighbours must reach Full**

A verification must wait for convergence, not check once.

**7. A. From the router's own data: the length of ospf_interfaces**

The expected state comes from the same data that built the configuration, so the test grows with the network.

**8. C. Recomputes routes: r1 reaches 2.2.2.2 via r3 at a higher cost**

verify.yml then reports the lost neighbour, as monitoring should.

[Back to the questions](13-network-automation.md#check-your-understanding)

## Chapter 14: Final Project: a Secure Web Stack

**1. D. web1 and web2 only**

Least privilege: the API serves only the web servers.

**2. B. Setting it first would cut Ansible's own SSH connection halfway through the run**

Allow rules first, then drop everything else.

**3. D. From the web servers' facts, looked up for each host in firewall_allow**

subelements('hosts') makes one rule per port and host.

**4. A. With --limit, other hosts' facts would be missing; this gathers them and stores them as those hosts' facts**

It is what lets site.yml --limit web2 repair one server.

**5. C. It owns a whole file, leaving the package's file alone, and the first value set wins**

validate: sshd -t -f %s checks the file before it is used.

**6. D. The whole stack is idempotent, so it can run every night to undo drift**

That is the test of a good playbook.

**7. B. A firewall that allows everything would pass every positive test; blocked ports must be proved blocked**

The two ignored results are the expected failures, judged by the final assert.

**8. C. site.yml --limit web2, then verify.yml to prove the repair**

The playbooks are the truth; servers are brought back to it.

[Back to the questions](14-final-project.md#check-your-understanding)
