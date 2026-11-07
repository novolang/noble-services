# Changelog

Every published version, newest first. This file is on the publish
allow-list, so it travels with the package: it is the only thing a
consumer deciding whether to upgrade can read.

## Unreleased

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
