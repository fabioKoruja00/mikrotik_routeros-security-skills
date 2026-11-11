# IoT checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Modbus gateway

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Gateway without security rules | `/iot modbus print` | `disabled=no` with `disable-security-rules=yes` (the default): any host reaching `tcp-port` (502) reads and writes the serial devices. `disabled=no` is only possible on a model with the serial port; elsewhere enabling fails with "Couldn't acquire hardware port" | CRITICAL |
| Rules too wide | `/iot modbus security-rules print proplist=ip-range,allowed-function-codes,disabled` | with rules enforced, an `ip-range` beyond the SCADA hosts, or write function codes (5, 6, 15, 16) allowed to hosts that only read | HIGH |
| Port reachable | `/ip firewall filter print stats where chain=input` | following the rule order, TCP 502 accepted from a WAN or customer interface | HIGH |

## 2. MQTT

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Broker without TLS | `/iot mqtt brokers print proplist=name,address,port,ssl,certificate,username,auto-connect` | `ssl=no` to a broker outside the local network: credentials and payloads in clear | MEDIUM |
| Broker without credentials | `/iot mqtt brokers print count-only where password=""` | count of brokers with no password; combined with a public broker address, anyone can impersonate the device's topics | MEDIUM |
| Subscription that runs code | `/iot mqtt subscriptions print proplist=broker,topic,qos` + `/iot mqtt subscriptions print count-only where on-message!=""` | subscriptions with an `on-message` script on a wildcard topic or a broker others can publish to: whoever publishes there triggers code on the router. Do not read the script body | MEDIUM |

## 3. LoRa gateway

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Forwarding to unplanned servers | `/iot lora print proplist=name,disabled,network,forward,servers,spoof-gps` + `/iot lora servers print proplist=name,address,port,protocol,ssl,certificate` | an enabled gateway forwarding to a server nobody declared, or over the unencrypted UDP packet forwarder (`ssl=no`) across the Internet, where a server `key` set (`/iot lora servers print count-only where key!=""`) also travels unprotected | MEDIUM |
| Join filters open | `/iot lora joineui print proplist=name,type,logging` + `/iot lora netid print proplist=name,type,logging` | no JoinEUI or NetID filter on a private network gateway: it relays every device in range to the configured servers | LOW |

## 4. Bluetooth and Wiliot

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Scanner without whitelist | `/iot bluetooth scanners print proplist=type,filter-policy,disabled` + `/iot bluetooth whitelist print proplist=address,device,disabled` | enabled scanner accepting every advertiser where only known tags matter: unrelated devices feed the data pipeline | LOW |
| Peripheral keys stored | `/iot bluetooth peripheral-devices print proplist=name,address,address-type,persist` + `/iot bluetooth peripheral-devices print count-only where mtik-key!=""` | keys for tags nobody recognises; never print `mtik-key` | LOW |
| Wiliot server without TLS | `/iot wiliot servers print proplist=name,address,port,ssl,certificate` | `ssl=no` to a cloud endpoint. The list ships with a factory entry; judge it only where Wiliot tags are in use | LOW |
</content>
