# Multi-Tier Web & DB Automation Deployment

An Ansible project that provisions a two-tier stack — an Nginx web server and a
MariaDB database server — on Rocky Linux, as a capstone for an Ansible
automation course.

## What it does

- **Facts-driven tuning** (`tasks/facts.yaml`): reads `ansible_facts` (CPU count,
  total RAM) and derives values used later, such as the Nginx
  `worker_processes` count and an InnoDB buffer-pool size for MariaDB.
- **Jinja2 templating** (`templates/index.html.j2`): renders a status page per
  host showing its hostname, IP, OS, CPU/RAM, and the facts-derived values
  above.
- **File & config management**: creates the web content directory, deploys the
  template with `ansible.builtin.template`, and edits existing config files
  in place with `ansible.builtin.lineinfile` (hiding the Nginx version banner,
  binding MariaDB to all interfaces).
- **Modular tasks**: `site.yaml` is a thin entry point that pulls in
  `tasks/facts.yaml`, `tasks/packages.yaml`, `tasks/config.yaml`, and
  `tasks/firewall.yaml` via `include_tasks`, so each concern lives in its own
  file.
- **Firewall**: ensures `firewalld` is installed and running, then opens the
  ports each tier needs (80/443 on web, 3306 on db) via `ansible.posix.firewalld`.

## Layout

```
ansible-enterprise-deploy/
├── ansible.cfg
├── inventory/
│   └── hosts.yaml        # web (node1) and db (node2) groups
├── group_vars/
│   ├── web.yml            # web_packages, firewall_ports for the web group
│   └── db.yml              # db_packages, firewall_ports for the db group
├── templates/
│   └── index.html.j2
├── tasks/
│   ├── facts.yaml
│   ├── packages.yaml
│   ├── config.yaml
│   └── firewall.yaml
├── site.yaml
└── README.md
```

## Requirements

- Rocky Linux 8/9/10 targets, reachable over SSH as a `become`-capable user.
- The `ansible.posix` collection (`ansible-galaxy collection install ansible.posix`)
  for the `firewalld` module.

## Usage

```bash
# check connectivity
ansible all -m ping

# dry run
ansible-playbook site.yaml --check --diff

# real run
ansible-playbook site.yaml
```
