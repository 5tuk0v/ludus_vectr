# Ansible Role: VECTR ([Ludus](https://ludus.cloud))

An Ansible Role that installs [VECTR](https://github.com/SecurityRiskAdvisors/VECTR) on Ubuntu hosts.

This role:
- Downloads the latest VECTR release from GitHub
- Configures the VECTR environment with customizable variables
- Deploys VECTR using Docker Compose
- Displays access credentials upon successful deployment

> [!WARNING]
> VECTR is a containerized application composed of multiple Docker services. According to the official documentation, the host system should have at least 4 CPU cores and 8 GB of RAM available to ensure stable performance.

## Requirements

- Ubuntu 22.04 LTS (Jammy)

> [!NOTE]
> This role currently targets Ubuntu 22.04 LTS as it is the officially supported distribution in the VECTR documentation. Other Debian-based distributions may work but have not been tested.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Installation directory for VECTR
ludus_vectr_install_path: "/opt/vectr"

# Hostname for accessing VECTR (must be resolvable via DNS or hosts file)
# VECTR requires HTTPS and will redirect HTTP requests to the configured hostname
ludus_vectr_hostname: "vectr.ludus.domain"

# HTTPS port for VECTR access
ludus_vectr_port: 8081

# Container logging level (DEBUG, INFO, WARN, ERROR, FATAL)
ludus_vectr_container_log_level: "WARN"

# MongoDB data directory
ludus_vectr_data_dir: "/var/data/"

# PostgreSQL database configuration
ludus_vectr_postgres_password: "vectrtest"
ludus_vectr_postgres_user: "vectr"
ludus_vectr_postgres_db: "vectr"

# Redis password
ludus_vectr_redis_password: "ChangeMeRedis"

# Security keys (change these for production use!)
ludus_vectr_data_key: "ChangeMeNow"
ludus_vectr_jws_key: "ChangeMeJWSKey123456"
ludus_vectr_jwe_key: "ChangeMeJWEKey123456"

# Docker Compose project name
ludus_vectr_compose_project_name: "vectr"

# Certificate profile (usercert, internal)
ludus_vectr_certprofile: "internal"
```

### Custom SSL Certificates

By default, VECTR uses self-signed certificates (`ludus_vectr_certprofile: "internal"`). To use custom SSL certificates:

1. Set `ludus_vectr_certprofile: "usercert"` in your role variables
2. After deployment, manually edit `/opt/vectr/.env` and add:
   ```
   VECTR_SSL_CRT=CERT-WITH-ESCAPED-NEWLINES\n-GOES-HERE
   VECTR_SSL_KEY=PRIVATE-KEY-WITH-ESCAPED-NEWLINES\n-GOES-HERE
   ```
3. Restart the containers: `cd /opt/vectr && docker compose restart`

## Dependencies

- `geerlingguy.docker`

## Example Playbook

```yaml
- hosts: vectr_hosts
  roles:
    - 5tuk0v.ludus_vectr
  vars:
    ludus_vectr_hostname: "vectr.mylab.local"
    ludus_vectr_postgres_password: "MySecurePassword123!"
```

## Example Ludus Range Config

```yaml
ludus:
  - vm_name: "{{ range_id }}-vectr-ubuntu2204-x64"
    hostname: "{{ range_id }}-vectr"
    template: ubuntu-22.04-x64-server-template
    vlan: 10
    ip_last_octet: 10
    ram_gb: 8
    cpus: 4
    linux: true
    roles:
      - 5tuk0v.ludus_vectr
    role_vars:
      ludus_vectr_hostname: "vectr.{{ range_id }}.local"
      ludus_vectr_port: 8081
      ludus_vectr_postgres_password: "MySecurePassword123!"
      ludus_vectr_redis_password: "MySecureRedis456!"
      ludus_vectr_data_key: "MySecureDataKey456!"
      ludus_vectr_jws_key: "MySecureJWSKey789!"
      ludus_vectr_jwe_key: "MySecureJWEKey012!"
```

## License

GPLv3

## Author Information

This role was created by [5tuk0v](https://github.com/5tuk0v), for [Ludus](https://ludus.cloud/).

- GitHub: [@5tuk0v](https://github.com/5tuk0v)
- Twitter/X: [@stuk0v_](https://twitter.com/stuk0v_)
