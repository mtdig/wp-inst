# WordPress Ansible Installatie

Uitbreiding SEL Opdracht 3: Installeer een database en web server. En zo ...

Automatische installatie van een complete WordPress stack met MariaDB, Apache en beveiligingsconfiguratie.

## Wat wordt er geïnstalleerd?

- **MariaDB** - Database server
- **Apache + PHP** - Webserver met PHP ondersteuning
- **WordPress** - CMS
- **UFW** - Firewall (poorten 22, 80, 443, 3306)
- **fail2ban** - Bescherming tegen brute-force aanvallen
- **SSH hardening** - Alleen key-based authenticatie
- **fastfetch** - Systeeminfo bij login

## Getting started

### NixOS

```bash
nix develop
```

Dit installeert automatisch Python en alle dependencies.

### Andere systemen

Vereisten:
- Python 3.10+
- uv (Blazingly fast Python package manager)

```bash
# Installeer uv (indien nog niet aanwezig)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Installeer dependencies
uv sync
```

## Configuratie

### Inventory

Pas `inventory.yml` aan met je eigen hosts:

```yaml
all:
  hosts:
    jouw_server:
      ansible_host: 192.168.1.100
      ansible_user: gebruiker
      ansible_python_interpreter: /usr/bin/python3
```

### Vault (wachtwoorden)

Maak een `vault.yml` bestand aan in de root van het project:

```yaml
---
db_wp_password: "jouw_database_wachtwoord"
wp_admin_password: "jouw_wordpress_admin_wachtwoord"
```

> **Let op:** Dit bestand staat in `.gitignore` en moet op elke machine handmatig aangemaakt worden.

### Playbook variabelen

Pas indien nodig de variabelen aan in `playbooks/site.yml`:

```yaml
vars:
  wp_domain: jouw-domein.nl
  wp_path: /var/www/wordpress
  wp_db_name: wordpress
  wp_db_user: wpuser
```

## Uitvoeren

```bash
# Volledige installatie
uv run ansible-playbook playbooks/site.yml

# Alleen specifieke host
uv run ansible-playbook playbooks/site.yml --limit jouw_server

# Dry-run (check mode)
uv run ansible-playbook playbooks/site.yml --check

# Met verbose output
uv run ansible-playbook playbooks/site.yml -v
```

## Structuur

```
.
├── ansible.cfg          # Ansible configuratie
├── inventory.yml        # Host definities
├── vault.yml            # Wachtwoorden (niet in git)
├── playbooks/
│   └── site.yml         # Hoofdplaybook
└── roles/
    ├── common/          # SSH, UFW, fail2ban, fastfetch
    ├── mariadb/         # Database server
    └── wordpress/       # Apache, PHP, WordPress
```

## Na installatie

Na succesvolle installatie verschijnt een bericht met de URL om WordPress te configureren via de webbrowser.
