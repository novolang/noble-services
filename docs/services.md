# Modules, configuration and the policy hooks

`noble-services` is the generic GATT service half of the noble stack: the services a Peripheral exposes and the Collector
side that reads them from a peer, written over the Host of `noble`.  It holds no register, no codec and no Controller code.

## What it reads

| Package | What the services take from it |
|---|---|
| `noble` | the link table (`links`), the GATT server and client (`gatt_server`, `gatt_client`), ATT (`att`), the profile builder, the notification helper, the byte encodings and the shared control steps (`profile_builder`, `notify`, `wire`, `control_common`), and the L2CAP credit based channels (`l2cap_coc`) the Object Transfer Service moves its objects through |

No module of `noble` imports a module of this package, and the services import no module of `noble-audio`;
`tests/orbit/test_ble_core_imports_no_extension.sh` in the repository holds both rules.

## The services

1. **Device Information, Battery.**  `device_information` is the Device Information Service with its characteristics in
   fixed order, `dis_policy` adds the product's strings and the rules the stack keeps (UTF-8, the length of the System ID and
   the PnP ID), `battery` is the Battery Service, and `bas_policy` adds several instances and the policy that names their
   levels.  `device_information_client` and `battery_client` read them.
2. **HID.**  `hid` is the HID Service, and `hogp` is HID over GATT: the Report Map, the reports and the boot reports, the
   Protocol Mode of each connection, and the host's writes that the product's policy answers.
3. **Health.**  `heart_rate`, `health_thermometer`, `blood_pressure`, `pulse_oximeter`, `glucose` and `cgm` are the health
   services, `racp` the Record Access Control Point the glucose, continuous glucose monitoring and insulin delivery services share, `generic_health_sensor` the Generic Health
   Sensor Service, and `insulin_delivery` with `insulin_delivery_history` and `insulin_delivery_command` the Insulin Delivery
   Service in its three families.
4. **Sports, fitness and sensing.**  `cycling_power`, `cycling_speed_cadence`, `running_speed_cadence`, `fitness_machine`,
   `physical_activity_monitor`, `weight_scale`, `body_composition`, `location_navigation`, `environmental_sensing` and
   `user_data`.  `sensor_control` holds what the Cycling Speed and Cadence, Cycling Power and Running Speed and Cadence
   services share, and `sensor_client` the client session the sensor clients use.
5. **Connection services.**  `current_time`, `link_loss`, `tx_power`, `bond_management` and `reconnection_configuration`.
6. **Object Transfer.**  `object_transfer` is the service, `object_channel` moves the objects through an L2CAP LE Credit Based
   channel, `object_list` is the list of objects, `object_store` declares the store a product supplies and holds an in-memory
   store, and `object_transfer_client` is the client.
7. **Nordic UART.**  `nordic_uart` is the Nordic UART Service.

Every `<service>_client` module is the Collector side of the service of the same name.

## Policy hooks

A service that has a decision for the product asks a policy trait: `DisPolicy` for the Device Information strings and
who may read them, `BasPolicy` for the Battery levels, `HogpPolicy` for the host's writes of the Protocol Mode, the control
point and the output reports.  A program passes its own implementation of the trait when it adds the service.

## Configuration

The schema is the `[config.<key>]` tables of `novo.toml`: `max_dis_octets` is the longest string value of the Device
Information Service, `max_hid_reports` and `max_hid_map_octets` size the HID Service, and `max_profiles_btp` is the switch
the BTP session of `orbit/ble` turns on when it holds the three services.  The number of Battery Service instances,
`max_bas`, multiplies `max_connections` and a cost formula reads only the keys of its own schema, so `max_bas` is a key of
the `[noble]` section and the sources read it as `noble_config.MAX_BAS`.  It moves to this schema when a formula can read the
keys of another package.

`services_budget` adds up the storage of the keys of this schema and holds the sum to the total `novo config explain`
prints.  The RAM the board leaves the Host (`host_budget_bytes`) is a key of `orbit/ble`, which adds the storage of this
package to the sums of the packages `noble` and `noble-audio`.  `configs/<profile>.toml` are the profiles the package's tests
run under; the same names exist in `orbit/ble/configs/`, whose `[noble]` and `[noble-services]` sections equal these, and
`tests/orbit/test_ble_config_equals_limits.sh` checks the two.

## The public surface

`orbit/ble` (the BTP faces, the soak and chip images, the PC Hosts) names 32 items of the package across the package boundary;
they are `pub`, each with a doc comment, and every other item is private.  `novo doc` prints them: `hogp` (12), `bas_policy` (8),
`dis_policy` (5), `hid` (4) and `device_information` (3).  The other modules are private to the package until an in-tree consumer names them.

## Tests

The unit tests of a module are the suite `tests/<module>_tests.nv`, which `novo test tests/<module>_tests.nv` runs; `src/`
holds none.  A suite sees the private names of the package, so it names a module's helpers directly.  A fixture that the
suites of two modules share stays in `src/` under a heading that says so, because a suite cannot import another suite
(`object_transfer` holds the rig that `object_channel_tests` and `object_transfer_client_tests` build their servers from).
