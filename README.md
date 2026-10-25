<div align="center">

<img src="snap/gui/rustfs.png" alt="RustFS logo" width="128" />

# RustFS

**S3-compatible object storage.**

[![rustfs](https://snapcraft.io/rustfs/badge.svg)](https://snapcraft.io/rustfs)
[![Base: core26](https://img.shields.io/badge/Base-core26-2C8C3C?logo=canonical&logoColor=white)](https://snapcraft.io/core26)
![Confinement: strict](https://img.shields.io/badge/Confinement-strict-2C8C3C)
[![Maintainer: sidaliteir](https://img.shields.io/badge/Maintainer-sidaliteir-2C8C3C?logo=snapcraft&logoColor=white)](https://snapcraft.io/publisher/sidaliteir)
![Architectures: amd64](https://img.shields.io/badge/Architectures-amd64-2C8C3C?logo=linux&logoColor=white)

[Getting Started](#getting-started) · [Configure](#configure) · [Limitations](#limitations) · [Build](#build) · [Support](#support) · [License](#license)

Unofficial snap packaging for [RustFS](https://github.com/rustfs/rustfs), an object storage server with a web console.

</div>

---

## Getting Started

<a href="https://snapcraft.io/rustfs"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install rustfs
sudo cat /var/snap/rustfs/common/credentials/access-key
sudo cat /var/snap/rustfs/common/credentials/secret-key
```

The service starts automatically with generated credentials. Use them at [http://127.0.0.1:9001/rustfs/console/](http://127.0.0.1:9001/rustfs/console/). The S3 endpoint is `http://127.0.0.1:9000`.

## Configure

| Setting | Accepted values | Default |
| --- | --- | --- |
| `volumes` | Space-separated paths; RustFS brace expansion supported | `/var/snap/rustfs/common/data` |
| `address` | `host:port` or `:port`; port `1`–`65535` | `127.0.0.1:9000` |
| `console-enable` | `true`, `false` | `true` |
| `console-address` | `host:port` or `:port`; port `1`–`65535` | `127.0.0.1:9001` |
| `server-domains` | Comma-separated domain names | Unset |
| `region` | S3 region string | Unset |

```bash
sudo snap get rustfs
sudo snap set rustfs address=:9000 console-address=:9001
```

Changes restart the running service. For an external disk:

```bash
sudo snap connect rustfs:removable-media
sudo mkdir -p /mnt/disk1/rustfs
sudo snap set rustfs volumes=/mnt/disk1/rustfs
```

For TLS, put `rustfs_cert.pem` and `rustfs_key.pem` in `/var/snap/rustfs/common/tls`, then run `sudo snap restart rustfs`. Alternatively, use an HTTPS reverse proxy. To rotate credentials, replace the two files above and restart.

## Limitations

- Service storage paths must be inside the snap's data directory or connected `/mnt` and `/media` mounts; arbitrary host paths are unavailable.
- Multiple drives must use separate mounted disks and require `sudo snap connect rustfs:mount-observe`.
- Back up data and credentials before updates. `snap revert` does not roll back stored data or its format; keep cluster nodes on the same version.

## Build

```bash
git clone https://github.com/sidalit/rustfs-snap.git
cd rustfs-snap
sudo snap install snapcraft --classic
snapcraft pack
```

## Support

- **Installation or snap issues:** [open a packaging issue](https://github.com/sidalit/rustfs-snap/issues).
- **Application bugs:** [report them upstream](https://github.com/rustfs/rustfs/issues).

## License

Snap packaging: [MIT](https://github.com/sidalit/rustfs-snap/blob/main/LICENSE).
