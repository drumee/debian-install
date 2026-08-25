# Drumee — Debian install

Install Drumee directly on a Debian server or virtual machine, from Debian
packages — no container. This is the path for a production instance.

- **Website:** [drumee.com](https://drumee.com)
- **Documentation:** [docs.drumee.com](https://docs.drumee.com/introduction/)

> Debian-family distributions only. For a containerised install see
> [drumee/docker-hosted](https://github.com/drumee/docker-hosted).

---

## Requirements

| | Minimum |
|---|---|
| OS | Debian 11 or newer |
| RAM | 8 GB |
| CPU | 2 GHz, 2 cores |
| Domain | A dedicated internet domain name, with access to its DNS zone **and** its glue records |
| Network | At least one public IP address — IPv4 required, IPv6 recommended |

Storage layout matters here:

- Put `DRUMEE_DATA_DIR` (the file store) and the server root on **different**
  partitions.
- Give `DRUMEE_DB_DIR` its own partition, at least 100 GB, on the fastest disk
  you have — SSD or NVMe if the instance will see heavy read/write.
- The domain name **cannot be shared** with any other application, and the
  database server should not be shared either.

The installer pulls in and configures nginx, MariaDB, Prosody, Jitsi Meet,
Node.js, BIND 9, Redis, Postfix, OpenDKIM, LibreOffice, GraphicsMagick and
FFmpeg, so start from a clean machine rather than one already running a web or
mail server.

## 1. Point your domain at the server

Drumee runs BIND and is authoritative for its own domain, so both the records
and the glue records have to be in place before installation.

In your DNS zone, replacing `example.org` with your domain — if you have no IPv6
address, fill in the IPv4 rows only:

| Name | Type | Target |
|---|---|---|
| `example.org` | A | your IPv4 address |
| `example.org` | AAAA | your IPv6 address |
| `ns1.example.org` | A | your IPv4 address |
| `ns1.example.org` | AAAA | your IPv6 address |
| `ns2.example.org` | A | your IPv4 address |
| `ns2.example.org` | AAAA | your IPv6 address |

Then at your registrar:

1. Set the domain's name servers to `ns1.example.org` and `ns2.example.org`.
2. Add both as **glue records**.
3. Wait for propagation.

Confirm before going further:

```console
nslookup example.org
```

## 2. Configure

```console
git clone https://github.com/drumee/debian-hosted.git
cd debian-hosted
cp env.sh drumee.sh
```

Edit `drumee.sh`. The installer validates these and refuses to start if any are
wrong:

| Variable | Required | Notes |
|---|---|---|
| `DRUMEE_DOMAIN_NAME` | yes | Must match the DNS records above |
| `PUBLIC_IP4` | yes | Validated as a dotted IPv4 address |
| `PUBLIC_IP6` | no | Validated if set |
| `ADMIN_EMAIL` | yes | Becomes the admin account and receives the setup link |
| `ACME_EMAIL_ACCOUNT` | no | For the ACME certificate; defaults to `ADMIN_EMAIL` |
| `DRUMEE_DB_DIR` | yes | **The directory must already exist** |
| `DRUMEE_DATA_DIR` | yes | **The directory must already exist** |
| `STORAGE_BACKUP` | no | rsync backup destination |
| `DRUMEE_DESCRIPTION` | no | Free text shown on the login page |

Two things that will stop the install cold:

- `DRUMEE_DB_DIR` and `DRUMEE_DATA_DIR` are checked for existence — create them
  first, with enough space.
- The installer refuses to place them under system paths (`/usr`, `/etc`,
  `/var`-adjacent system directories, `/root`, and so on).

## 3. Install

Run as **root** — `su`, not `sudo`:

```console
./install
```

The installer fetches the Drumee package list from `app.drumee.com`, installs
`drumee-infra`, `drumee-schemas`, `drumee-server-pod`, `drumee-ui-pod` and
`drumee-static`, then provisions the database, requests certificates and starts
the services.

When it finishes, a setup link is sent to `ADMIN_EMAIL`. Open it, set the admin
password, and the instance is live.

### Options

```console
./install --env-file=/path/to/other.sh   # use a different env file
./reinstall                              # purge the Drumee packages and install again
```

`reinstall` removes `drumee-ui-pod`, `drumee-server-pod`, `drumee-schemas`,
`drumee-infra` and `drumee-static` before reinstalling. It does **not** delete
your data or database directories.

If `drumee-infra` is already installed the script stops rather than reinstalling
over a working system; pass `--force-infra-install=yes` if that is really what
you want.

## Other ways to install

| Path | Repository |
|---|---|
| Docker | [drumee/docker-hosted](https://github.com/drumee/docker-hosted) |
| Synology NAS | [drumee/synology-hosted](https://github.com/drumee/synology-hosted) |
| Local development environment | [drumee/starter-kit](https://github.com/drumee/starter-kit) |

## License

AGPL-3.0 — see [LICENSE](LICENSE).

## Contributing

See the org [CONTRIBUTING guide](https://github.com/drumee/.github/blob/main/CONTRIBUTING.md).
Questions and self-hosting help: [Discussions](https://github.com/orgs/drumee/discussions).
