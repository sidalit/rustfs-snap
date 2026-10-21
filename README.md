# RustFS snap

[![rustfs](https://snapcraft.io/rustfs/badge.svg)](https://snapcraft.io/rustfs)

Snap packaging for [RustFS](https://github.com/rustfs/rustfs), an open-source, S3-compatible object storage server.

Unofficial build, not affiliated with or endorsed by the RustFS project.

## Build

```bash
snapcraft pack
```

## Install and try

<a href="https://snapcraft.io/rustfs"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install rustfs
```

The service starts right away. The S3 API listens on `127.0.0.1:9000`; open http://127.0.0.1:9001/rustfs/console/ for the web console.

Random credentials are generated at installation:

```bash
sudo cat /var/snap/rustfs/common/credentials/access-key
sudo cat /var/snap/rustfs/common/credentials/secret-key
```

Use them in the console or an S3 client. The `rustfs` command also exposes upstream tools, such as `rustfs --version`.

## Background service

```bash
sudo snap logs rustfs       # view logs
sudo snap restart rustfs    # restart
sudo snap stop --disable rustfs
sudo snap start --enable rustfs
```

### Settings

| Setting | Default | Description |
|---|---|---|
| `volumes` | `/var/snap/rustfs/common/data` | Space-separated data folders; `{1...4}` expands to several drives |
| `address` | `127.0.0.1:9000` | S3 API address; `:9000` listens on all networks |
| `console-enable` | `true` | Enable the web console |
| `console-address` | `127.0.0.1:9001` | Web console address |
| `server-domains` | not set | Comma-separated domains for virtual-host-style requests |
| `region` | not set | Region reported to S3 clients |

Changes restart a running service. `snap unset rustfs <setting>` restores its default.

```bash
sudo snap get rustfs
sudo snap set rustfs address=:9000 console-address=:9001
```

For HTTPS, place `rustfs_cert.pem` and `rustfs_key.pem` in `/var/snap/rustfs/common/tls`, then restart the service. To change credentials, replace the corresponding files under `/var/snap/rustfs/common/credentials` and restart.

## Data and backups

Data is stored in `/var/snap/rustfs/common/data`. To use a disk mounted under `/media` or `/mnt`:

```bash
sudo snap connect rustfs:removable-media
sudo mkdir -p /mnt/disk1/rustfs
sudo snap set rustfs volumes=/mnt/disk1/rustfs
```

For erasure coding, use separate mounted disks and connect `mount-observe`:

```bash
sudo snap connect rustfs:mount-observe
sudo snap set rustfs volumes='/mnt/disk{1...4}/rustfs'
```

Back up data and credentials before upgrading. `snap revert` restores the program, not its data or storage format. Keep cluster nodes on the same RustFS version.

## Known limitations

- Service storage is limited to snap data directories and connected drives under `/media` or `/mnt`; arbitrary paths such as `/srv` are unavailable.
- Multiple drives require the manual `mount-observe` connection and separate disks.
- The snap uses RustFS's legacy filesystem fallback when confinement blocks `openat2`; kernel audit messages may appear.
- Updates use snapd; the daemon's upstream update check is disabled.

## Update to a new upstream release

Change `version` in `snap/snapcraft.yaml`, check the console release and checksum, then rebuild.

## Automation

Builds, publishing, and dependency updates use [snap-automation](https://github.com/sidalit/snap-automation).

## License

The packaging is MIT-licensed (see `LICENSE`). RustFS and its console are Apache-2.0-licensed; their dependencies carry their own licenses.
