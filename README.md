# F3BFS

A distributed POSIX filesystem for biomedical bigdata, extensible and feature-rich.

## Environment

### Build 

- Linux
`libfuse` and `build-essential` are required, in ubuntu/debian:

```
sudo apt install -y libfuse-dev libfuse3-dev build-essential
```

- macOS
```
brew install --cask osxfuse
```

### Runtime
- Linux
`fuse3` and `openssl` are required, in ubuntu/debian:

```
sudo apt-get install -y libfuse3-dev fuse3 libssl-dev
```

- macOS

```
brew install --cask osxfuse
```

In Catalina or former version, you need to load osxfuse into the kernel:

```
/Library/Filesystems/osxfuse.fs/Contents/Resources/load_osxfuse
```

## Installation

> The `install.sh` may fail in macOS Catalina or Big Sur because of the 
> [SIP](https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection). 
> 
> You can just use the `target/release/f3bfs` to mount f3bfs.
> ### Example
> ```
> target/release/f3bfs f3bfs:127.0.0.1:2379 ~/mnt
> ```

### Source code

```bash
git clone https://github.com/Hexilee/f3bfs.git
cd f3bfs
sudo make install
```

## Usage
You need a RocksDB cluster to run f3bfs.

#### TLS
You need ca.crt, client.crt and client.key to access cluster on TLS. 

> It will be convenient to get self-signed certificates by [sign-cert.sh](sign-cert.sh)(based on the [easy-rsa](https://github.com/OpenVPN/easy-rsa)).

You should place them into a directory <cert dir> and execute following docker command.

```bash
docker run -d --device /dev/fuse \
    --cap-add SYS_ADMIN \
    -v <cert dir>:/root/.f3bfs/tls \
    -v <mount point>:/mnt:shared \
    hexilee/f3bfs:0.3.1 --mount-point /mnt --pd-endpoints <endpoints>
```

### Binary

```bash
mkdir <mount point>
mount -t f3bfs f3bfs:<pd endpoints> <mount point>
```

#### TLS

```bash
mount -t f3bfs -o tls=<tls config file> f3bfs:<pd endpoints> <mount point>
```

By default, the tls-config should be located in `~/.f3bfs/tls.toml`, refer to the [tls.toml](config-examples/tls.toml) for detailed configuration.

## Other Custom Mount Options

### `direct_io`

Enable global direct io, to avoid page cache.

```bash
mount -t f3bfs -o direct_io f3bfs:<pd endpoints> <mount point>
```
### `blksize`

The block size, 64KiB by default, could be human-readable.

```bash
mount -t f3bfs -o blksize=512 f3bfs:<pd endpoints> <mount point>
```

### `maxsize`

The quota of fs capacity, could be human-readable.

```bash
mount -t f3bfs -o maxsize=1GiB f3bfs:<pd endpoints> <mount point>
```

## Development

```bash
cargo build
mkdir ~/mnt
RUST_LOG=debug target/debug/f3bfs --mount-point ~/mnt
```

Then you can open another shell and play with f3bfs in `~/mnt`.

Maybe you should enable `user_allow_other` in `/etc/fuse.conf`.

for developing under `FreeBSD`, make sure the following dependencies are met.

```bash
pkg install llvm protobuf pkgconf fusefs-libs3 cmake
```

for now, `user_allow_other` and `auto unmount` does not work for `FreeBSD`, using as `root` and manually `umount` is needed.

## Contribution

### FUSE
There are little docs about FUSE, refer to the [example](https://github.com/cberner/fuser/blob/master/examples/simple.rs) for the meaning of FUSE API.
