# OpenWrt sing-box Gateway

A planned LuCI application for managing sing-box policy routing on OpenWrt, with Hysteria2 outbound support.

> **Project status: planning. There is no installable package yet.**
> Features described here are proposed, not implemented or verified. Do not deploy this project on a router carrying production home-network traffic.

## Why this project exists

Running proxy routing on OpenWrt often means maintaining several separate pieces by hand: sing-box configuration, rule sets, DNS, scheduled update scripts, firewall forwarding, and service startup. When something breaks, it can be difficult to tell whether the cause is a rule, DNS, the tunnel, or the firewall. A configuration mistake can affect every device in a home network.

This project aims to bring those tasks into an OpenWrt-native LuCI interface. It should make configuration easier to inspect, validate settings before applying them, preserve the last known-good rules when an update fails, restore ordinary direct access where appropriate, and show useful status and diagnostic information.

The project is based on a real home-network use case using sing-box and Hysteria2. The goal is to turn that experience into an implementation that others can review, reproduce, and improve.

## What it aims to provide

### sing-box service management

- Show whether sing-box is installed, its version, and its running state.
- Start and stop the service and control startup at boot from LuCI.
- Generate sing-box configuration and validate it before applying changes.
- Show relevant service logs and error details.

### Hysteria2 outbound configuration

- Configure the server address, port, authentication, TLS/SNI, and Salamander obfuscation.
- Configure Brutal congestion-control bandwidth values with clear units.
- Mask sensitive fields and exclude credentials from diagnostic exports.

### Policy routing and rule-based traffic splitting

- Route traffic to proxy, direct, or reject actions based on domains, IP addresses, and CIDR ranges.
- Show rule matching order and let users add their own rules.
- Fetch supported rules from Shadowrocket rule sources and convert them locally to sing-box rule sets. sing-box is the proxy core; Shadowrocket is only an optional rule-data source.
- Store custom rules separately from downloaded upstream rules so updates do not overwrite user settings.
- Make the final policy and behavior during proxy failures explicit. A direct fallback must not be presented as if it still protects traffic privacy.

### Rule updates and rollback

- Support manual updates and configurable scheduled updates.
- Use a staged update flow: download, parse, convert, compile, validate, and atomically replace.
- Keep the currently active rules and report the reason when a network, parsing, or validation step fails.
- Retain the previous valid rules and configuration for rollback.

### DNS, status, and diagnostics

- Show DNS interception and upstream settings to help prevent DNS loops.
- Keep DNS and IPv6 advertisement changes explicit; do not silently replace existing network settings during installation.
- Show service, tunnel, and rule status, with an option to export redacted diagnostics.

### Safe activation and recovery

- Do not take over LAN traffic automatically after installation.
- Apply configuration only after validation succeeds.
- Plan for cleanup of forwarding state, configuration rollback, and a clear disable procedure if the service exits unexpectedly.
- Preserve user configuration and backups by default when uninstalling.

## Design principles

1. **Safe defaults:** installation and upgrades must not silently change the network path for the whole LAN.
2. **Recoverable failures:** failed rule or configuration updates must leave the last known-good version available.
3. **Explainable behavior:** users should be able to inspect rule sources, match order, update results, and failure modes.
4. **Minimize credential exposure:** node credentials must not appear in logs, diagnostic bundles, or version control.
5. **Use OpenWrt conventions:** prefer UCI, procd, rpcd/ubus, LuCI, and native firewall integration.
6. **Separate upstream and local rules:** upstream rules can be updated without overwriting user exceptions.

## Planned milestones

1. Define the supported OpenWrt, sing-box, and firewall versions.
2. Build the package skeleton, LuCI menu, and read-only status page.
3. Implement Hysteria2 configuration, configuration generation, and validation.
4. Add support for selected Shadowrocket rule syntax and user-defined rules.
5. Implement safe updates, status reporting, and rollback.
6. Validate firewall takeover and recovery on isolated test equipment before enabling LAN-wide routing.
7. Complete installation, upgrade, uninstall, recovery, and package-release documentation.

See [PROJECT_PLAN.md](PROJECT_PLAN.md) for detailed scope and acceptance criteria.

## Current status and contributions

The project is in the planning stage. Features, configuration formats, the compatibility matrix, and installable packages have not been released. There are no production installation instructions yet.

Feature requests and discussions about supported OpenWrt versions and design are welcome through GitHub Issues. Before code contributions are accepted, the repository will add a license, contribution guidelines, and a process for reporting security issues.

For substantial changes, please open an Issue first. Changes involving the firewall, default routes, DNS, or automatic traffic takeover should include failure scenarios and a recovery plan.

## License and upstream rule attribution

The project license has not been selected. Repository contents should not yet be treated as software released under a specific open-source license.

The project plans to support rules from [Johnshall/Shadowrocket-ADBlock-Rules-Forever](https://github.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever). That project has its own license and attribution requirements. The planned approach is to fetch and convert rules locally at runtime and identify the source in the interface and documentation. Rule adaptation does not change ownership of, or maintenance responsibility for, the upstream project.
