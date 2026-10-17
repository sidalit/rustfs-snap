# RustFS snap

[![rustfs](https://snapcraft.io/rustfs/badge.svg)](https://snapcraft.io/rustfs)

Snap packaging for [RustFS](https://github.com/rustfs/rustfs), an open-source, S3-compatible, high-performance object storage server.

Unofficial build, not affiliated with or endorsed by the RustFS project.

## Build

```bash
snapcraft pack
```

## Install and try

<a href="https://snapcraft.io/rustfs"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install rustfs   # or a local build: ./rustfs_1.0.2_*.snap --dangerous
```

The service starts right away and stores data in `/var/snap/rustfs/common/data`:

| What | Address |
|---|---|
| S3 API | `http://127.0.0.1:9000` |
| Web console | `http://127.0.0.1:9001/rustfs/console/` |

Random credentials are generated at installation, instead of upstream's well-known defaults:

```bash
sudo cat /var/snap/rustfs/common/credentials/access-key
sudo cat /var/snap/rustfs/common/credentials/secret-key
```

Use them to log in to the console, or with any S3 client:

```bash
aws --endpoint-url http://127.0.0.1:9000 s3 mb s3://my-bucket
```

The `rustfs` command runs RustFS's own tools, for example `rustfs --version` or `sudo rustfs info`.

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
| `volumes` | `/var/snap/rustfs/common/data` | Data folders, separated by spaces. `{1...4}` expands to several drives |
| `address` | `127.0.0.1:9000` | Address of the S3 API. Use `:9000` for all networks |
| `console-enable` | `true` | Serve the web console |
| `console-address` | `127.0.0.1:9001` | Address of the web console |
| `server-domains` | not set | Domains for virtual-host-style requests, separated by commas |
| `region` | not set | Region reported to S3 clients |

Changes apply right away. The service restarts by itself.

```bash
sudo snap get rustfs
sudo snap set rustfs address=:9000 console-address=:9001
sudo snap unset rustfs region
```

### Open the service to your network

```bash
sudo snap set rustfs address=:9000 console-address=:9001
```

Your firewall must allow the ports (for example `sudo ufw allow 9000,9001/tcp`).

### HTTPS

Place a certificate and its key in `/var/snap/rustfs/common/tls`, named `rustfs_cert.pem` and `rustfs_key.pem`, then restart the service. RustFS then serves the S3 API and console over HTTPS.

### Credentials

To change the credentials, replace the content of the two files in `/var/snap/rustfs/common/credentials`, then restart the service.

## Storage on other drives

The service can only use folders in `/var/snap/rustfs/common` and, with the `removable-media` connection, disks mounted under `/media` or `/mnt`:

```bash
sudo snap connect rustfs:removable-media
sudo mkdir -p /mnt/disk1/rustfs
sudo snap set rustfs volumes=/mnt/disk1/rustfs
```

### Erasure coding across several disks

With several volumes, RustFS checks that each one is a separate disk, which needs the `mount-observe` connection:

```bash
sudo snap connect rustfs:mount-observe
sudo snap set rustfs volumes='/mnt/disk{1...4}/rustfs'
```

RustFS refuses several volumes on the same physical disk.

## Known limitations

### Folders outside /var/snap/rustfs/common, /media and /mnt

The service cannot use other folders, such as `/data` or `/srv`. Mount the disks under `/mnt` instead.

### Multiple drives need a manual connection

`mount-observe` is not connected automatically. Without it, a multi-drive setup fails to start with `local endpoint cross-device mount validation failed: Permission denied`.

### Upgrades and reverts

Data lives in `/var/snap/rustfs/common`, which all revisions share, so `snap revert` restores the previous release but not the data. A newer release may update the on-disk format, which an older release might not read. Back up important data before major upgrades.

For a cluster of several servers, refresh them together, as RustFS warns against mixing versions:

```bash
sudo snap refresh --hold rustfs   # on each server, then refresh them all at the same time
```

### openat2 messages in the kernel log

The snap sandbox does not allow the `openat2` system call, so the snap makes RustFS use its fallback for kernels without it. The kernel log shows `seccomp` audit messages for `openat2` when the service starts. They are harmless.

### Update check

RustFS's update check is turned off, as updates come from the Snap Store.

## Update to a new upstream release

Change `version` and `source-tag` in `snap/snapcraft.yaml`, check the console release in the `console` part, then rebuild.

## License

The packaging is MIT-licensed (see `LICENSE`). RustFS and its console are Apache-2.0-licensed; their dependencies carry their own licenses.
