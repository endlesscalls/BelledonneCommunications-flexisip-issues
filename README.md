# Flexisip 2.6.2 in Push Gateway mode: ContactRouteInserter restore / Router lookup fails, and no SNI on outgoing TLS

This repo documents two problems found while running **Flexisip 2.6.2** as a **push gateway** in front of a third-party SIP registrar. Flexisip sends FCM push notifications to wake Android clients that are asleep.

> Status: initial report. Items marked `TODO` will be filled in with config and log excerpts.

---

## Environment

| Component | Version / details |
|---|---|
| Flexisip | 2.6.2 (`TODO: package or build source, e.g. Debian repo / Docker / built from source`) |
| OS | `TODO: distro + version` (VPS) |
| Upstream registrar / PBX | Vodia PBX 70.6 (multi-tenant, one SIP domain per tenant) |
| Client | Custom Android app on Linphone SDK 5.4 |
| Test device | Unihertz Titan 2, Android 16 |
| Push | Firebase Cloud Messaging (HTTP v1), own Firebase project |
| Transport client → Flexisip | `TODO: TLS / TCP / UDP` |
| Transport Flexisip → PBX | TLS |

### Topology

```
Android app (Linphone SDK 5.4)
        │  REGISTER / INVITE (with push params in Contact)
        ▼
Flexisip 2.6.2 — push gateway
        │  forwards REGISTER upstream (ContactRouteInserter)
        │  receives INVITE from PBX → Router lookup → FCM push → fork
        ▼
Vodia PBX 70.6 (registrar for <tenant>.endlesscalls.com)
```

### Relevant configuration

```ini
# TODO: paste the relevant sections of flexisip.conf with secrets removed:
# [global] transports
# [module::Registrar]
# [module::ContactRouteInserter]
# [module::Router] (fallback-route etc.)
# [module::PushNotification] (FCM settings, no keys)
# [module::Forward]
```

---

## Issue 1: ContactRouteInserter restore / Router lookup fails in Push Gateway mode

### Expected behavior

1. The client REGISTERs through Flexisip. Flexisip stores the binding, including push parameters. `ContactRouteInserter` rewrites the Contact and forwards the REGISTER to the PBX.
2. The client app is closed or the phone is asleep.
3. The PBX sends an INVITE to the rewritten Contact, which reaches Flexisip.
4. `ContactRouteInserter` restores the original contact, and `module::Router` finds the stored binding.
5. Flexisip sends an FCM push, the app wakes and re-registers, and the INVITE is forked to it.

### Actual behavior

When the app is closed, the phone **never rings**. The call does show up afterwards in the app as a **missed call**, so something about the call reaches the client, but the call is never presented in time to be answered.

`TODO: confirm in the Flexisip logs whether the ContactRouteInserter restore and Router lookup succeed, whether the FCM push is sent, and when the CANCEL from the PBX arrives compared to the app's re-REGISTER.`

### Steps to reproduce

1. Configure Flexisip 2.6.2 as a push gateway in front of the PBX (config above).
2. Register the Android client through Flexisip with push params in the Contact.
3. Close the app or let the device sleep.
4. Call the extension from another phone.

### Logs

```
TODO: REGISTER as received from the client and as forwarded to the PBX (Contact headers)
```
```
TODO: incoming INVITE from the PBX (Request-URI) plus the ContactRouteInserter / Router debug lines
```

---

## Issue 2: no SNI on Flexisip's outgoing TLS connections

### Expected behavior

When Flexisip opens an outgoing TLS connection, for example to the upstream PBX, it sends the SNI (`server_name`) extension with the target hostname. Servers that pick their certificate by SNI, like a multi-tenant PBX, can then present the right certificate.

### Actual behavior

The ClientHello from Flexisip has no SNI. `TODO: describe the effect, e.g. the server presents its default certificate, the handshake fails, or the TLS connection is rejected.`

### How to verify

```bash
# On the Flexisip host, capture the outgoing handshake:
sudo tcpdump -i any -w flexisip-tls.pcap 'tcp port 5061'
# Open it in Wireshark, filter on tls.handshake.type == 1,
# and check for the "server_name" extension in the ClientHello.
```

```
TODO: paste the ClientHello summary or the relevant Flexisip TLS log lines
```

---

## Workarounds tried

- `TODO`

## Contact

Repo maintained by EndlessCalls.
