---
name: iot
description: "Read-only audit of the RouterOS v7 iot package: MQTT brokers and the scripts their subscriptions run, the Modbus TCP-to-serial gateway and its security rules, LoRa gateway forwarding and servers, Bluetooth scanners, advertisers and peripheral keys, Wiliot servers. Use this skill whenever a MikroTik device has the iot package installed, is a KNOT, LtAP or LoRa gateway, talks MQTT or Modbus, or the audit mentions /iot, MQTT, Modbus, LoRa or Bluetooth on RouterOS, without changing configuration."
---

# IoT

The iot package bridges field devices to the network. A Modbus gateway with its security rules off lets any host that reaches TCP 502 write coils on the serial side; an MQTT subscription runs a script whenever someone publishes to the topic.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **No bare `print` on `/iot mqtt brokers`, `/iot mqtt subscriptions`, `/iot lora servers` or `/iot bluetooth peripheral-devices`.** They hold `password`, the `on-message` script body, the server `key` and the `mtik-key`. Use the `proplist` in the checks and `count-only` for presence.
- **Menus exist without the hardware.** LoRa, Bluetooth and the Modbus serial port only work on models that have them; a configuration there on a device without the radio or port is inert, so lower the severity and say why.
- **Modbus ships with `disable-security-rules=yes`.** The rule list is ignored until that flag is `no`, so rules present do not mean filtering.
- Scripts and scheduler in general are in `scripts-storage`.
</content>
