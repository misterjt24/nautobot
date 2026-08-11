# NAUTOBOT
## Zielstellung: 
Es soll die Möglichkeit gegeben sein, mit der Devices, welche in Nautobot verwaltet werden, einfacher und revisionssicher gemanacht/konfiguriert werden können.
Hierfür kommen auch config_contexts in zum Einsatz

## pb.nautobot_build_full_config.yaml
Mithilfe diese anible-Playbooks können Konfigs von Devices aus NB in ein config-File exportiert werden.

## Replacing the static inventory files with the Nautobot Inventory module
group_vars/all/all.yaml ->  which holds the device username and password as well as the nautobot URL and token.

test this new inventory file with 
```
ansible-inventory -v --list -i inventory/nautobot_inventory.yaml
```

## use nautobot config_contexts
###connect to Git-Repository Config-Context
Place where all config_contexts stored 


## zum Beispiel: 

Device Tag: offen_access

### Config_content: VLAN 100 (JSON-FORMAT)
``` 
{
    "vlans": [
        {
            "name": "offen",
            "vid": 100
        }
    ]
}
```

Tags: offen_access

### NTP
JSON-FORMAT
```
{
    "ntp": "10.0.0.1"
}
```

YAML-FORMAT
```
ntp: 10.0.0.1

```

###
    

