# noble-services

The generic GATT services and profiles of the noble Bluetooth Low Energy stack.  The package holds the services a
Peripheral exposes (Device Information, Battery, Heart Rate, HID over GATT, Current Time, Link Loss, Tx Power, Bond
Management, Reconnection Configuration, Object Transfer, the cycling, running and fitness services, the health and
body sensors, and the Nordic UART Service) and, for most of them, the Collector or client side that reads the service
from a peer.  The services run over the Host of the `noble` package: each adds its characteristics to a `GattServer`
through the `profile_builder` steps, and each client drives a `GattClient`.

A service takes its values from the program that uses it.  Where a product decides (which Device Information strings to
publish, how many Battery levels to report, how to answer a HID output report), the service has a policy trait that asks it (`DisPolicy`, `BasPolicy`, `HogpPolicy`), and the program passes its own
implementation.

| Module | Holds |
|---|---|
| `device_information`, `dis_policy`, `device_information_client` | the Device Information Service with the product's strings and the rules the stack keeps (UTF-8, the length of the System ID and the PnP ID), and its client |
| `battery`, `bas_policy`, `battery_client` | the Battery Service, with several instances and the policy that names their levels, and its client |
| `hid`, `hogp` | the HID Service and HID over GATT: the Report Map, the reports and the boot reports, the Protocol Mode of each connection, and the host's writes |
| `heart_rate`, `health_thermometer`, `blood_pressure`, `pulse_oximeter`, `glucose`, `cgm`, `racp`, `generic_health_sensor`, `insulin_delivery` and the `insulin_delivery_*` modules | the health services, the Record Access Control Point they share, and the Insulin Delivery Service in its three families |
| `cycling_power`, `cycling_speed_cadence`, `running_speed_cadence`, `fitness_machine`, `physical_activity_monitor`, `weight_scale`, `body_composition`, `location_navigation`, `environmental_sensing`, `user_data` | the sports, fitness and sensing services |
| `current_time`, `link_loss`, `tx_power`, `bond_management`, `reconnection_configuration` | the services of a connection: time, link loss alerts, transmit power, the bonds of a device and the reconnection parameters |
| `object_transfer`, `object_channel`, `object_list`, `object_store`, `object_transfer_client` | the Object Transfer Service, the channel the objects move through, the list of objects, the store a product supplies, and the client |
| `nordic_uart` | the Nordic UART Service |
| `ranging_data`, `ranging_service`, `ranging_client`, `ranging_profile` | the Ranging Service (RAS 1.0, UUID 0x185B) and the Ranging Profile (RAP 1.0): the Ranging Data Body of one Channel Sounding procedure and its segments, the server with Real-time and On-demand Ranging Data, the control point, Ranging Data Ready and Overwritten, the client that discovers, subscribes, reassembles and acknowledges, and the pairing of the peer's data with the Host's own result for the estimator `cs-ranging` |
| `sensor_control`, `sensor_client` | what the sensor services share: the control point and the client session of the sensor clients |
| `<service>_client` | the Collector side of the service of the same name |
| `services_budget` | the RAM the keys of this package's schema add up to |

A service adds its characteristics to a database with the steps of `profile_builder` and answers the requests the Host
hands it.  The Battery Service holds one level, cut to 100:

```novo
use battery

// A level above 100 is held as 100 and a negative level as 0, whatever the program passes.
fn battery_level_is_cut_to_100() -> Bool
    battery.cut_level(130) == 100 and battery.cut_level(-5) == 0 and battery.cut_level(80) == 80
```

The sizes of the tables are keys of the package configuration.  A consumer writes a `[noble-services]` section in its
config file for the Device Information Service (`max_dis_octets`) and the HID Service (`max_hid_reports`,
`max_hid_map_octets`, `max_profiles_btp`).  The number of Battery Service instances, `max_bas`, stays in the `[noble]`
section, because its storage is a product of `max_connections` and a cost formula reads only the keys of its own schema.  The
build writes the keys as `noble_services.CONFIG.MAX_DIS_OCTETS` and `noble.CONFIG.MAX_BAS`, and `novo config explain`
prints the storage they imply.  `configs/` holds the profiles this package's tests run under.

The package depends on `noble`.  No module of `noble` imports a module of this package, and the services
import no module of `noble-audio`.  `docs/services.md` describes the modules and the configuration.

## Install, build and test

```sh
novo pkg add noble-services
```

adds the package to the `[dependencies]` table of the consumer's `novo.toml`.  In a checkout of the package:

```sh
novo build
novo test
```

`novo build` type-checks and effect-checks every module under `src/`; the package is a library and writes no binary.
`novo test` runs the suites under `tests/`, one for each module, `tests/<module>_tests.nv`.  `novo test
tests/<module>_tests.nv` runs one of them.  A suite takes a profile with `--config configs/pc.toml`; `novo config ci` runs
every suite under the smallest and the largest configuration the schema allows and under each file of `configs/`.  The tests
are not part of the published package.
