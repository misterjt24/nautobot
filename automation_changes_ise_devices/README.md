# Cisco Internet Switch Automation

Diese Playbooks ermitteln Switches ueber die ISE-Network-Device-Group und Nautobot und aktualisieren anschliessend das lokale IOS-Admin-Passwort beziehungsweise den TACACS-Shared-Secret.

## Voraussetzungen

- Ansible sowie HTTPS-Erreichbarkeit von Cisco ISE ERS, Nautobot und den Switches
- Ein ISE-ERS-Konto mit Leserechten fuer Network Devices
- Ein Nautobot-Token mit Leserechten fuer Devices
- Ein TACACS-/SSH-Konto mit ausreichenden IOS-Rechten und Enable-Zugang
- Auf jedem Switch muss der lokale Benutzer `admin` bereits mit einem geeigneten Privilege-Level vorhanden sein oder durch die verwendete IOS-Version korrekt angelegt werden koennen

Installiere die benoetigten Collections und die Python-Abhaengigkeit fuer IP/CIDR-Vergleiche:

```sh
ansible-galaxy collection install -r requirements.yml
python3 -m pip install netaddr
```

## Vault vorbereiten

Pruefe alle Werte in `internet_vault.yaml`, trage fehlende Zugangsdaten ein und verschluessele die Datei vor der Verwendung. Speichere oder committe sie nicht unverschluesselt:

```sh
ansible-vault encrypt internet_vault.yaml
```

Die Switch-SSH-Zugangsdaten (`switch_username`, `switch_password`, `switch_enable_password`) sind die bestehenden TACACS-/Enable-Daten und nicht das neu zu setzende lokale Admin-Passwort. Playbook 2 erkennt den TACACS-Stil anhand der Running Config und aktualisiert benannte Server, globale Keys oder `server-private`-Eintraege innerhalb der AAA-Gruppe.

IOS ist die Standardeinstellung. Fuer NX-OS muss das Inventory beim jeweiligen Host `tacacs_device_os=nxos` enthalten, zum Beispiel `sw-core-01 ansible_host=10.1.1.1 tacacs_device_os=nxos`. Die benoetigte `cisco.nxos`-Collection ist in `requirements.yml` aufgefuehrt. Playbook 1 bleibt IOS-spezifisch.

TLS-Zertifikatspruefung ist standardmaessig aktiviert. Installiere interne CA-Zertifikate im Systemvertrauensspeicher, statt die Pruefung abzuschalten.

## Ausfuehrung

Alle Befehle in diesem Verzeichnis ausfuehren. Zuerst Discovery starten; es schreibt `hosts_Internet.ini` nur, wenn ISE-Netze und Nautobot-Treffer vorhanden sind:

```sh
ansible-playbook -i localhost, playbook_0_discovery.yml --ask-vault-pass
```

Pruefe die generierte Inventardatei und teste zuerst mit einem einzelnen Geraet und Check Mode:

```sh
ansible-playbook -i hosts_Internet.ini playbook_1_password.yml --ask-vault-pass --limit sw-berlin-01.example.local --check
ansible-playbook -i hosts_Internet.ini playbook_2_tacacs.yml --ask-vault-pass --limit sw-berlin-01.example.local --check
```

Nach manueller Freigabe den Check Mode weglassen, zuerst Passwort und danach TACACS-Key ausrollen:

```sh
ansible-playbook -i hosts_Internet.ini playbook_1_password.yml --ask-vault-pass
ansible-playbook -i hosts_Internet.ini playbook_2_tacacs.yml --ask-vault-pass
```

Fuer den TACACS-Wechsel muss der neue Key zuvor in ISE per **Retire** zusaetzlich aktiviert worden sein. Kontrolliere nach dem Rollout die Switch-Erreichbarkeit und TACACS-Authentifizierung, bevor der alte Key in ISE entfernt wird.

## Discovery-Details

Playbook 0 liest die ISE-ERS-Ressourcen paginiert ein, laedt die Detaildaten der Network Devices und beruecksichtigt nur Eintraege aus `network_device_group`. Danach werden Nautobot-Devices anhand ihrer `primary_ip4` gegen die ISE-Adressen beziehungsweise CIDRs verglichen. Der Nautobot-API-Aufruf nutzt `limit=0`, um die paginierte Device-Antwort vollstaendig abzurufen. Die ISE-Feldwerte `NetworkDeviceIPList[].ipaddress` koennen CIDRs oder Hostadressen sein; ein optionales `mask`-Feld wird angehaengt.

Nautobot-Devices werden paginiert mit `nautobot_page_size` geladen. Fuehre die Playbooks nicht mit `--diff` aus. TACACS-Tasks unterdruecken Ausgaben, die Schluessel enthalten koennten; Konfigurations-Backups werden auf dem Ansible-Controller abgelegt und muessen wie Geheimnisse geschuetzt werden.