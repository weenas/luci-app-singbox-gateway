# Prior-Art Review: OpenWrt LuCI and sing-box

**Reviewed:** 2026-11-05  
**Purpose:** Decide whether this project should build a new generic LuCI application for sing-box policy routing or reuse/contribute to an existing project.

This is a documentation-based review of public repositories, not an installation test or security audit. Features and maintenance status can change, so verify upstream before adoption.

## Summary

There is substantial prior art. Existing projects already cover most of the proposed scope: LuCI management, sing-box lifecycle, Hysteria2, policy routing, rule sets, scheduled updates, diagnostics, and in some cases validation and rollback.

This repository has no application code yet. **Pause work on a second generic LuCI sing-box gateway until an isolated comparison identifies a concrete unmet requirement.** Prefer using an existing project or contributing the missing behavior upstream. Continue this repository only if a specific gap and target audience are demonstrated.

## Projects reviewed

| Project | Documented capabilities | Relevance and caveats |
| --- | --- | --- |
| [ImmortalWrt HomeProxy](https://github.com/immortalwrt/homeproxy) | Sing-box proxy platform; Hysteria/Hysteria2, Salamander obfuscation, upload/download Mbps settings, routing and access controls. GPL-2.0; large repository and ongoing activity. | Closest established option for the Hysteria2 use case. Verify firmware targets, workflow, and failure behavior. Bandwidth fields alone do not prove every desired congestion-control behavior. |
| [OpenWrt PassWall2](https://github.com/Openwrt-Passwall/openwrt-passwall2) | Multi-core LuCI proxy platform; sing-box and Hysteria2, transparent proxy, domain/IP routing, DNS controls, subscriptions, per-device rules, and failover. | Mature and broad. Evaluate complexity, footprint, and exact sing-box behavior. |
| [OpenWrt-momo](https://github.com/nikkinikki-org/OpenWrt-momo) | LuCI for sing-box transparent proxy; Redirect/TPROXY/TUN, IPv4/IPv6, access control, profile editor, scheduled restart; OpenWrt 24.10+ and sing-box 1.12+. GPL-3.0. | Strong existing transparent-proxy implementation. Check whether its profile workflow exposes the Hysteria2 settings required here. |
| [openwrt-singbox](https://github.com/aminmokhtari94/openwrt-singbox) | OpenWrt-native Go daemon and LuCI app using UCI, rpcd/ubus, procd; subscriptions/groups, DNS, routing/rule sets, TProxy, per-device modes, TUN, kill switch, OpenWrt 24.10/25.12 packages, CI and QEMU testing. Source includes Hysteria2 URI import and outbound rendering. | Very close architectural overlap. The reviewed source search did not establish Salamander or explicit Hysteria bandwidth controls in its UI. Verify its license and test it before reuse; no root LICENSE file was found in the reviewed contents. |
| [singbox-openwrt-config](https://github.com/ssharkkky/singbox-openwrt-config) | LuCI native/managed/profile modes; rule-set providers and scheduled refresh; validation, pending state, health checks, rollback, Hysteria2 URI import, and disposable-VM tests. | Strong overlap with the planned safe configuration/update/rollback workflow. Check its transparent-routing boundary and license; no root license file was evident in reviewed contents. |
| [LAS / luci-app-singbox](https://github.com/DmitriyStroganov/LAS) | LuCI management, routing rules, GeoSite presets, subscription import, dashboard, and configuration validation. | Explicitly lists Hysteria2 as unsupported, so less suitable for this node protocol, but relevant to UI and routing design. |

## Evidence

- HomeProxy Hysteria UI fields and Salamander option: [server.js](https://github.com/immortalwrt/homeproxy/blob/master/htdocs/luci-static/resources/view/homeproxy/server.js); Hysteria settings are rendered by [generate_client.uc](https://github.com/immortalwrt/homeproxy/blob/master/root/etc/homeproxy/scripts/generate_client.uc).
- openwrt-singbox features: [README](https://github.com/aminmokhtari94/openwrt-singbox#readme); Hysteria2 parsing in [subscription.go](https://github.com/aminmokhtari94/openwrt-singbox/blob/main/singbox-manager/src/internal/subscription/subscription.go) and outbound generation in [render.go](https://github.com/aminmokhtari94/openwrt-singbox/blob/main/singbox-manager/src/internal/render/render.go).
- singbox-openwrt-config [README](https://github.com/ssharkkky/singbox-openwrt-config#readme) documents configuration modes, rule providers, validation, rollback, and VM tests.
- Momo [README](https://github.com/nikkinikki-org/OpenWrt-momo#readme), PassWall2 [README](https://github.com/Openwrt-Passwall/openwrt-passwall2#readme), and LAS [protocol table](https://github.com/DmitriyStroganov/LAS#supported-protocols).

## Preliminary recommendation

1. Pause implementation of a generic gateway UI; the feature overlap is too high to justify the present scope.
2. Compare candidates on an isolated OpenWrt VM or test device: Hysteria2 setup, Salamander, required bandwidth values, custom-rule precedence, rule updates, DNS, per-device/LAN activation, and recovery when sing-box or the proxy stops.
3. Use an existing project if it meets the requirements. If one critical behavior is missing, propose it upstream first.
4. Continue this repository only if testing demonstrates a concrete gap maintainers will not address, or a deliberately different audience/workflow. Safe and reversible Hysteria2 activation is a possible niche to evaluate, not a proven differentiator.
5. Before adapting code, check the candidate's license, contribution policy, release cadence, supported targets, and security/recovery behavior.

## Decision status

**Open.** No implementation has started. No existing project has been selected and no narrowed differentiator has been approved. Revisit the project plan after the isolated comparison.
