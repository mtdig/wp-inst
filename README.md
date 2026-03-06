# WordPress Ansible Installatie

Uitbreiding SEL Opdracht 3: Installeer een database en web server. En zo ...

Automatische installatie van een complete WordPress stack met MariaDB, Apache en beveiligingsconfiguratie.

## Introductie

[Ansible](https://docs.ansible.com/) is een open-source automatiseringstool voor configuratiebeheer, applicatie-deployment en task automation. In tegenstelling tot andere tools zoals Puppet of Chef, werkt Ansible **agentless**: er hoeft geen software geïnstalleerd te worden op de doelservers.

Ansible maakt verbinding via **SSH** en voert taken uit door Python-modules naar de remote host te kopiëren en daar uit te voeren. Dit maakt het eenvoudig om te beginnen — als je via SSH kunt inloggen, kan Ansible het beheren.

De configuratie wordt geschreven in **YAML** bestanden (playbooks) die beschrijven welke taken uitgevoerd moeten worden. Ansible is **idempotent**: je kunt een playbook meerdere keren uitvoeren en alleen wijzigingen worden doorgevoerd.

Meer informatie: [Ansible Documentation](https://docs.ansible.com/ansible/latest/index.html)

## Installatie overzicht

### Role: common (basis beveiliging)

1. **SSH hardening**
   - `PasswordAuthentication no`
   - `PubkeyAuthentication yes`
   - `PermitRootLogin prohibit-password`
   - `ChallengeResponseAuthentication no`

2. **Packages installeren**
   - `ufw` (firewall)
   - `fail2ban` (brute-force bescherming)
   - `fastfetch` (systeeminfo)

3. **UFW firewall configuratie**
   - Standaard: inkomend verkeer deny, uitgaand allow
   - Poort 22 (SSH) open
   - Poort 80 (HTTP) open
   - Poort 443 (HTTPS) open
   - Poort 3306 (MySQL) open

4. **fail2ban configuratie**
   - 3 pogingen voor ban
   - 1 uur ban tijd
   - Localhost en ansible host uitgezonderd

5. **fastfetch toevoegen aan .bashrc**
   - Voor root en ansible user

### Role: mariadb (database)

1. **MariaDB server installeren**
   - `mariadb-server`
   - `python3-pymysql` (voor Ansible)

2. **Service starten en enablen**

3. **WordPress database aanmaken**
   - Database: `wordpress`
   - Authenticatie via unix_socket

4. **WordPress database user aanmaken**
   - User: `wpuser`
   - Volledige rechten op wordpress database

### Role: wordpress (webserver)

1. **Apache en PHP installeren**
   - `apache2`
   - `php`, `php-mysql`, `php-curl`, `php-gd`, `php-xml`, `php-mbstring`
   - `libapache2-mod-php`

2. **Apache modules enablen**
   - `rewrite`
   - `ssl`

3. **Self-signed SSL certificaat genereren**
   - Geldig voor 365 dagen
   - Opgeslagen in `/etc/ssl/certs/` en `/etc/ssl/private/`

4. **WordPress downloaden en uitpakken**
   - Van wordpress.org
   - Naar `/var/www/wordpress`

5. **wp-config.php aanmaken**
   - Database credentials
   - Security keys

6. **Apache virtual host configureren**
   - HTTP redirect naar HTTPS
   - SSL met self-signed certificaat

7. **Default site uitschakelen**

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

# Zonder common role (skip SSH hardening, UFW, fail2ban)
uv run ansible-playbook playbooks/site.yml -e skip_common=true

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
