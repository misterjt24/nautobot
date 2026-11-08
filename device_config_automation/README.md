# Cisco IOS Konfigurationen erstellen

Das Playbook fragt ausschließlich Nautobots Config Contexts ab, niemals
Devices. Aktive Kontexte ohne Tags gelten global; getaggte Kontexte werden
angewendet, wenn mindestens ein Tag mit `device_tags` des Hosts
übereinstimmt. Kontexte werden nach Gewicht zusammengeführt; `ios_config` aus
`inventory/hosts_ios.yaml` hat bei gleichen Schlüsseln Vorrang. Geräteidentität
und Tags stammen vollständig aus dem Inventory. Die Ausgabe liegt in
`build_config_files/` und ist nur für den ausführenden Benutzer lesbar.

## Einrichtung

1. Gerätename, Management-IP, Tags und IOS-Konfiguration in
   `inventory/hosts_ios.yaml` anpassen. Config-Context-Daten aus Nautobot können
   unter anderem `vlans`, `local_users`, `interfaces`, `snmp` und `ntp_servers`
   enthalten. Interfaces können mit `name` einzeln oder mit `range` als IOS-
   `interface range` gesetzt werden. Für Access-Port-Bereiche beispielsweise:

    ```yaml
    - range: "GigabitEthernet1/0/1 - 23"
       mode: access
       access_vlan: 10
       admin_state: up
    ```
2. Die Nautobot-URL in
   `inventory/group_vars/cisco_ios/nautobot.yml` eintragen. Das Nautobot-Token
   wird dort über `vault_nautobot_token` referenziert.
3. Die referenzierten Geheimnisse in
   `inventory/group_vars/cisco_ios/vault.yml` ablegen und mit Ansible Vault
   verschlüsseln. Benötigte Variablen: `vault_nautobot_token`,
   `vault_ansible_user`,
   `vault_ansible_password`, `vault_ios_enable_secret`,
   `vault_ios_local_user_secret` und `vault_snmp_ro_community`.
4. Collection installieren und Konfigurationen erzeugen:

   ```sh
   ansible-galaxy collection install -r requirements.yml
   ansible-playbook -i inventory/hosts_ios.yaml build_config.yml --ask-vault-pass
   ```

SNMPv2c-Communities werden als `snmp-server community` ausgegeben. Für SNMPv3
können `snmp.groups` und `snmp.users` im Config Context oder in den Hostdaten
gesetzt werden. Geheimnisse erscheinen in der generierten IOS-Konfiguration;
die Dateien werden deshalb mit Modus `0600` erstellt.