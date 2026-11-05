# OpenWrt sing-box Gateway LuCI Application — Project Plan

## 1. Goal

Assess whether a safe, recoverable LuCI application for sing-box policy routing on OpenWrt is needed, then build it only if comparison with existing projects demonstrates a concrete gap. The original use case is a home-network setup using sing-box with Hysteria2.

The application must not depend on Shadowrocket at runtime. sing-box is the proxy core; Shadowrocket may be one source of rule data.

## 2. Current project decision

A prior-art review found substantial overlap with HomeProxy, PassWall2, OpenWrt-momo, openwrt-singbox, and singbox-openwrt-config. The current repository contains documentation only and no application code.

**Implementation is on hold pending an isolated, requirement-based comparison.** Use an existing project or contribute upstream if it meets the requirements. Continue this repository only if a specific unmet need is demonstrated and its intended audience is clear.

See [docs/PRIOR_ART_REVIEW.md](docs/PRIOR_ART_REVIEW.md) for the preliminary comparison.

## 3. Requirements to compare

Evaluate candidate projects on an isolated OpenWrt VM or test device. Record actual behavior, not only feature claims.

- Hysteria2 manual and subscription setup.
- Salamander obfuscation and the required upload/download bandwidth settings.
- sing-box route ordering, proxy/direct/reject actions, and custom domain/IP rules.
- Rule provider updates, validation, failure handling, and rollback.
- DNS behavior for both IPv4 and IPv6, including recovery when the proxy is offline.
- LAN-wide versus per-device activation and safe shutdown behavior.
- OpenWrt releases, architectures, firewall versions, memory/storage use, license, release cadence, and maintainability.
- Whether a project's maintainers are open to contributions for any missing capability.

## 4. Conditional product scope

If the comparison demonstrates a unique gap, a possible MVP is a focused OpenWrt LuCI manager for the supported sing-box + Hysteria2 use case.

### Potential capabilities

1. **Service management**
   - Detect sing-box installation/version and display service state.
   - Start, stop, and enable the service at boot.
   - Generate configuration and validate it before activation.

2. **Hysteria2 outbound**
   - Configure server, port, authentication, TLS/SNI, and Salamander.
   - Expose bandwidth parameters with clear units and a warning to use realistic VPS capacity.
   - Mask secrets and redact diagnostic exports.

3. **Policy routing**
   - Support proxy, direct, and reject actions based on domains, IP addresses, and CIDR ranges.
   - Display rule precedence and let users maintain local exceptions separately from upstream rules.
   - Make final and failure policies explicit. Direct fallback preserves connectivity but may send traffic outside the tunnel; it is not a privacy-preserving fail-closed mode.

4. **Rule adaptation and updates**
   - Fetch upstream Shadowrocket rule files by URL without maintaining an upstream fork or copying the full file into this repository.
   - Initially support a documented subset such as DOMAIN, DOMAIN-SUFFIX, DOMAIN-KEYWORD, IP-CIDR, IP-CIDR6, and supported action markers.
   - Report unsupported or ambiguous lines with counts and locations; never silently discard them.
   - Convert supported matches to sing-box rule-set source JSON and compile to SRS using a compatible sing-box version.
   - Download, parse, convert, compile, and validate in a staging location; replace active files atomically only after every step succeeds.
   - Keep the previous active rules on failure. Show source, timestamp, digest, converter/core versions, and update result.

5. **DNS, diagnostics, and recovery**
   - Show DNS interception and upstream relationships to help avoid loops.
   - Do not silently overwrite LAN IPv6 RA/DHCPv6 DNS advertisements.
   - Provide service/tunnel/rule status and redacted diagnostic export.
   - Remove only firewall state owned by the application on stop/uninstall; preserve user data by default.

These capabilities are a proposal, not implemented behavior.

## 5. Safety requirements if development proceeds

- Installation must not enable LAN takeover.
- Applying network-wide settings must be explicit and have a clear recovery path.
- Candidate configuration must pass sing-box validation before replacing the active version.
- Rule update failure must leave the last known-good rules active.
- Service exit must not leave blackhole routes or stale forwarding rules.
- A failed restart after an update must restore the previous configuration and rules.
- Protect router management and local-network traffic from proxy routing loops.
- Do not place credentials, private domains, or home addresses in Git history, logs, screenshots, or diagnostics.
- Verify all recovery claims through isolated failure tests before documenting them as implemented.

## 6. Possible architecture, only if justified

Use standard OpenWrt mechanisms: UCI for persistent settings; LuCI for the interface; rpcd/ubus for bounded privileged operations; procd for service lifecycle; sing-box for proxy routing; and native firewall integration. Keep rule parsing, configuration generation, update, validation, and rollback outside the UI code so each component can be reviewed and tested independently.

Avoid adding a custom daemon or new data format unless the prior-art comparison demonstrates a need that cannot be met by existing OpenWrt and sing-box mechanisms.

## 7. Conditional milestones

### Phase 0: Prior-art decision

- Complete the isolated candidate comparison.
- Select an existing project, prepare an upstream contribution, or document the unique unmet need for this repository.
- Confirm target OpenWrt releases, architectures, sing-box source/version, firewall, and license constraints.

**Acceptance:** a written decision identifies the chosen path and evidence for it. No product code is written before this gate.

### Phase 1: Non-invasive package skeleton

- Add LuCI navigation, access controls, and a read-only status view.
- Add package metadata, UCI defaults, and service lifecycle scaffolding.
- Do not change traffic routing.

**Acceptance:** install/remove and status reporting work without affecting network traffic.

### Phase 2: Hysteria2 configuration and validation

- Add the required Hysteria2 fields and secret handling.
- Generate a candidate configuration and validate before apply.
- Verify node connectivity and service restart in an isolated environment.

**Acceptance:** invalid configuration is rejected; secrets are absent from logs; stopping the service leaves the test device with a usable direct path.

### Phase 3: Rule adapters and local exceptions

- Implement only the supported Shadowrocket syntax, with fixtures and parser tests.
- Convert and compile sing-box rule sets.
- Keep custom user rules independent and apply the documented precedence.

**Acceptance:** unsupported syntax is reported, update output is reproducible, and custom rules have the expected effect.

### Phase 4: Update lifecycle and rollback

- Add manual and scheduled updates, staging, validation, atomic replacement, and rollback.
- Show history and errors in LuCI.

**Acceptance:** simulated network, parse, compile, storage, and restart failures preserve a working version.

### Phase 5: Limited traffic takeover

- Add explicit LAN/per-device activation only if required by the proven gap.
- Provide preflight checks, a timeout-based rollback, and a clear disable action.
- Test one client before any broader rollout.

**Acceptance:** the client retains direct access and router management after node or service failure; no DHCP conflict, DNS loop, or unrecoverable route remains.

### Phase 6: Package and release

- Document installation, upgrade, uninstall, recovery, supported targets, privacy, and upstream attribution.
- Build and validate packages across the supported matrix.
- Publish a preview only after clean-device setup and failure recovery are verified.

## 8. Open items

- Decide whether this repository should continue after the prior-art comparison.
- Confirm a project license and contribution/security-reporting process before accepting code.
- Verify sing-box rule-set format compatibility for the selected core versions.
- Confirm attribution and license obligations for locally converted or distributed upstream rules.
- Verify the actual router's IPv4/IPv6, DNS, rule order, and firewall requirements if the project continues.

## 9. Status

This is a planning document. No application code has been implemented, no installable package is available, and LAN-wide takeover is not enabled by this project. All proposed behavior requires implementation and isolated validation before being described as complete.
