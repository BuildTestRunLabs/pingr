pingr is ping with a live TUI and an MTR-style path trace. Single static binary, IPv4, Linux only (also runs in WSL2).

## Install

Pick the file for your CPU: `x86_64` for most PCs and servers, `aarch64` for ARM (Raspberry Pi 4/5, Ampere, Graviton). Check with `uname -m`.

```sh
v=VERSION
arch=$(uname -m)   # x86_64 or aarch64
f=pingr-$v-$arch-unknown-linux-musl
curl -LO https://github.com/BuildTestRunLabs/pingr/releases/download/$v/$f.tar.gz
curl -LO https://github.com/BuildTestRunLabs/pingr/releases/download/$v/$f.tar.gz.sha256
sha256sum -c $f.tar.gz.sha256
tar xzf $f.tar.gz
sudo install $f/pingr /usr/local/bin/
pingr --version
```

### Verify the build (optional)

Each archive has a signed build attestation that proves it was built by this repository's release workflow from the tagged source. With the GitHub CLI:

```sh
gh attestation verify $f.tar.gz --repo BuildTestRunLabs/pingr
```

## Permissions

pingr uses an unprivileged ICMP socket, which most current distros (Ubuntu 20.04+, Debian 11+, Fedora, Arch) allow by default. If it exits with "no ICMP socket privilege", either allow ping sockets for everyone:

```sh
sudo sysctl -w net.ipv4.ping_group_range="0 2147483647"
echo 'net.ipv4.ping_group_range=0 2147483647' | sudo tee /etc/sysctl.d/99-ping.conf
```

or give only this binary the capability:

```sh
sudo setcap cap_net_raw+ep /usr/local/bin/pingr
```

## Usage

```sh
pingr 1.1.1.1                  # live TUI, q to quit
pingr -t example.com           # MTR-style route trace
pingr -I eth0 1.1.1.1          # send from one interface
pingr -q --rounds 5 1.1.1.1    # no TUI, five lines, then exit
pingr --help                   # all options
```

See the [README](https://github.com/BuildTestRunLabs/pingr#readme) and [reading the results](https://github.com/BuildTestRunLabs/pingr/blob/main/docs/reading-results.md) for details.
