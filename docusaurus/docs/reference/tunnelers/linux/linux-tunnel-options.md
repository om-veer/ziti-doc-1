
# Options and Modes

## `ziti-edge-tunnel` Environment Variables

`ZITI_TIME_FORMAT=utc` - set the log message time format to UTC timestamp instead of milliseconds since start

`ZITI_LOG=4` - set the log level of the underlying Ziti C SDK, higher is more verbose (level 4 means DEBUG)

`TLSUV_DEBUG=4` - set the log level of the underlying libuv library, higher is more verbose (level 4 means DEBUG)

<!-- Ken will update this when a better C SDK reference becomes available; this is a full URL because Docusaurus resolves relative and absolute URL and file paths at build time, and the Vercel build will always fail because it can't resolve the linked docs sites, e.g. CLANG doxygen site -->
For more information about configuring the underlying Ziti C SDK with environment variables, see [the Ziti C SDK documentation](https://docs.openziti.io/docs/reference/developer/sdk/clang/).

## `ziti-edge-tunnel` Global Options

You can start `ziti-edge-tunnel` with different options, some of the most commonly used options are listed below.

```bash
# Load a single identity.
--identity <identity>
```

```bash
# Load all identities in a dir, ignoring files with a .bak or .original filename suffix.
--identity-dir <dir>
```

```bash
# Set log level, higher is more verbose (default level 3 means INFO).
--verbose N
```

```bash
# Set service polling interval in seconds (default 10).
--refresh N
```

## Run Modes

There are two run modes:

* `run`: transparent proxy with DNS nameserver
* `run-host`: hosting-only without proxy or nameserver

### `run` Mode

`ziti-edge-tunnel run` provides a transparent proxy and nameserver. The nameserver may be configured to be authoritative (the default) or recursive with a command-line option. The OS is automatically configured to treat the nameserver as primary. You may inspect the resulting configuration with these commands.

```bash
resolvectl dns     # inspect the association of tun device and nameserver
resolvectl domain  # inspect the configuration of query routing domains
```

If any interface has a wildcard routing domain configured, `ziti-edge-tunnel` will also configure its tun with a wildcard routing domain. If no other interface has a wildcard routing domain configured, neither will the `ziti-edge-tunnel` tun.

```bash
# Specify the tun interface address and the subnet to which Service domain names are resolved (default 100.64.0.1/10). The nameserver address is always the tun interface address +1, default is 100.64.0.2.
--dns-ip-range <ip range>
```

#### How does `run` configure nameservers?

`ziti-edge-tunnel run` provides a built-in nameserver that will answer queries that exactly match authorized OpenZiti services' intercept domain names, and will respond with a hard-fail `NXDOMAIN` code if the query does not match an authorized service.

You may enable DNS recursion by specifying an upstream nameserver to answer queries for other domain names that are not services' intercept domain names: `ziti-edge-tunnel run --dns-upstream 208.67.222.222`.

`ziti-edge-tunnel` uses the `libsystemd` D-Bus RPC client and will try to configure the OS's resolvers with `systemd-resolved`. If that's not possible for any reason then `ziti-edge-tunnel run` will fall back to shell commands like `resolvectl`. If `resolvectl` fails then `ziti-edge-tunnel run` will attempt to modify `/etc/resolv.conf` directly to install the built-in nameserver as the primary resolver.

If the DNS record exists it returns the answer and sets query status to `NO_ERROR`. If it does not exist then it sends the query to an upstream DNS server if configured. Otherwise, it sets the query status to `REFUSE`. This implies that the caller *should* keep trying to resolve the domain name with other nameservers.

#### System Requirements for Mode `run`

`ziti-edge-tunnel run` requires elevated privileges for managing the `/dev/net/tun` device, running `resolvectl` to assign nameservers and domain routes to the tun interface, and running `ip route` to manage IP routes. This requires running as root because `setcaps` are not inherited in all of these cases, even when the inherit bit is set.

### `run-host` Mode

`ziti-edge-tunnel run-host` is a mode for hosting (listening) for services which does provide service intercepts or DNS. Services configured for 'Bind' will be hosted by the tunneller.

#### System Requirements for Mode `run-host`

`ziti-edge-tunnel run-host` does not require elevated privileges or the above device or socket, only network egress to the servers for which it is hosting Services.
