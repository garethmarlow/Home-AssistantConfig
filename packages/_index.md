# Packages — Index

Home Assistant loads packages through `homeassistant.packages: !include_dir_named packages`. Read only the group and YAML file relevant to the task.

| Concern | Start here |
|---|---|
| Apple Watch helpers | `apple_watch.yaml` |
| EV charging and vehicle climate | `car/` |
| Appliance power signatures, battery strategy, carbon, energy statistics, inverter/BMS and Octopus slots | `energy_management/` |
| Heating controls and HVAC | `heating/` |
| School calendars and waste collection | `household/` |

When adding or moving a package:

1. keep one operational concern per file;
2. update this index if the routing map changes;
3. review entity IDs and cross-file references;
4. run the safe change sequence in [`docs/agent/access-and-operations.md`](../docs/agent/access-and-operations.md).
