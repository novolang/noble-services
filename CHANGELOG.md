# Changelog

Every published version, newest first. This file is on the publish
allow-list, so it travels with the package: it is the only thing a
consumer deciding whether to upgrade can read.

## 0.1.2 — 2026-11-17

2026-11-14: `smp_gatt` answers a request that completes while the last response is still being sent, in place of dropping it without a word.  A client that keeps several requests in flight (the window of the upload in nRF Connect Device Manager's Image Upload card) got no answer to the ones that came early and reported "Connection timed out".  `SmpEngine` gains `out_len`, `out_pop` and `out_clear`: the engine keeps the responses that wait in a queue, `answer` puts the response at its end, `tx_byte` reads the oldest, and the service sends them in the order of the requests and removes each once its last piece is queued.  What an engine does when its queue has no room is its own (`mcumgr-nv` 0.1.2 refuses a request that is not an upload with EBUSY and answers an upload request with the offset to send again).  `tests/smp_gatt_tests.nv` has the case, 16 tests.

2026-11-14: `smp_gatt`, the SMP service of MCUmgr (service 8D53DC1D-1DB7-4CD3-868B-8A527460AA84, one characteristic DA2E7828-FBCE-4E01-AE9E-261174997C48 that is written without response and notifies), the firmware update service of nRF Connect Device Manager, `mcumgr` and `smpmgr`.  It implements `noble.Service`: `attach` adds the service and binds the characteristic to a function, `on_write` takes the bytes of a write command and reassembles the request by the Data Length of its header, `tick` sends the response in notifications of ATT_MTU - 3 bytes, and `set_mtu` and `peer` let the product tell the service the link's ATT_MTU.  The server behind it is the product's `SmpEngine` (for `mcumgr-nv`, `smpfrag.rx_byte` on its session's request buffer and `smpsess.handle`), so the package depends on no satellite.  `attach` refuses a configuration whose `att_value_bytes` is less than `max_att_mtu` - 3 (`value_fits`).  The characteristic keeps its longest write, `att_value_bytes`, after the first notification (it needs `noble` with `host_api.store_sent_value`).  `smp_gatt_client` is the Central's side for a PC: `open`, `send` and `receive` over the Adapter.  The tests are `tests/smp_gatt_tests.nv` (14, run with `--config configs/smp.toml`) and `tests/smp_gatt_client_tests.nv` (3).

## 0.1.1 — 2026-11-10

This release needs `noble` 0.2.0 (the `Service` trait) and a toolchain 0.24.1 or later.

2026-11-09: the package reads its configuration as `noble_services.CONFIG` (`use noble_services.CONFIG`, `noble_services.CONFIG.MAX_DIS_OCTETS`) in place of the module `noble_services_config`.  The keys, their values and the constants the build writes are unchanged; only the name in the source moved.  The keys of the core stay `noble.CONFIG.MAX_BAS`.  A program that wrote `noble_services_config` builds for one toolchain release with the warning E2947 at each `use` line; write `use noble_services.CONFIG`.  The source needs a toolchain 0.24.1 or later, the first release with the spelling.

`battery`, `device_information`, `heart_rate`: each module gains a value that implements `noble.Service`, for a product that hands the Adapter of the package noble a list of services: `battery_service(level)` (`set_level` on the value the product keeps; a read of the level is answered from it and a change is notified to every subscriber once), `device_information_service(info)` (the values of a `DeviceInformation` are stored in the Host's database at `attach`; a value longer than `att_value_bytes` stops the attach with `E_NO_ROOM`) and `heart_rate_service(config)` (`report(m)` queues a measurement for the next turn, `energy_reset_requested()` reads the control point's reset once; the notification has room for the 20 octets of the default ATT MTU).  The functions of each module for the BTP session of orbit/ble are unchanged.  The other services of the package are NOT YET ported to `Service`.

The Ranging Service and the Ranging Profile: `ranging_data` (the Ranging Data Body of a Channel Sounding procedure, its segmentation and its reassembly), `ranging_service` (the server: RAS Features, Real-time and On-demand Ranging Data, the control point with Get_Ranging_Data, ACK_Ranging_Data, Retrieve_Lost_Ranging_Data_Segments and Abort_Operation, Ranging Data Ready and Ranging Data Overwritten), `ranging_client` (discover, read the features, subscribe, ask for the data, reassemble, ask for lost segments, acknowledge, with the timeouts of RAP) and `ranging_profile` (the Ranging Responder and Requester procedures, and the pairing of the peer's data with the Host's own result of the same procedure).  Set_Filter is not supported and RAS Features bit 3 is clear.

## 0.1.0

First release.  The generic GATT services and profiles of the noble Bluetooth
stack, extracted from `orbit/ble` (`src/host/profiles/`): Device Information,
Battery, Heart Rate, HID over GATT, Current Time, Link Loss, Tx Power, Bond
Management, Reconnection Configuration, Object Transfer, the cycling, running
and fitness services, the health and body sensors, the Insulin Delivery
Service and the Nordic UART Service, each with the policy a product plugs its
decisions into and, for most, the Collector or client side.  The services run
over the Host of `noble`, the only package they depend on.

Four keys of the `orbit/ble` configuration are the schema of this package,
under the section `[noble-services]`: `max_dis_octets`, `max_hid_reports`,
`max_hid_map_octets` and `max_profiles_btp`; the constants are read as
`noble_services_config.MAX_*`.  The number of Battery Service instances,
`max_bas`, stays in the schema of `noble` (`[noble]`), because its cost formula
multiplies `max_connections` and a formula reads only the keys of its own
schema.

The public surface is what `orbit/ble` (the BTP faces, the soak images and the
chip images) names across the package boundary.  The unit tests are in `tests/`,
one suite for each module, and do not ship; they run under the smallest and the
largest configuration and under each file of `configs/`.
