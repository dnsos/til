## Different angles on security

The `Dockerfile` used by Kamal should make sure to:

- keep the base image patched - i.e. if a vulnerability is discovered in Ruby 3.4.7, a bump to Ruby 3.4.8 might solve that (hypothetical example)
- use `apt-get update` to keep installed packages up to date with every build (therefore not every build is reproducible, but the security advantage outweighs that)

The host server that runs the Docker containers should be kept up to date.

We should use something like `unattended-upgrades` that is configured to frequently bring in security releases. Minor version bumps should potentially be avoided.

This makes sure that system dependencies (e.g. OpenSSL, Docker or even the Kernel) are secure.

Automatic reboots at an appropriate time should be allowed. If the app is behind a load balancer with multiple app servers, their reboots should be staggered, so that there is always an app instance available.

Updates that require a reboot are not _that_ frequent.
