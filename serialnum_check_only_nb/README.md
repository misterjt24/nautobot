Nautobot Cisco Serial Check
===========================

The playbook `pb_serialnum_check_only_nb.yml` queries Nautobot GraphQL for
devices whose role is `network`, filters to Cisco device types, connects to
each Cisco IOS device, and checks its chassis and stack-member serial numbers
against Nautobot. Missing serials are appended to
`files/log/serials_not_found.log`.

Requires the `cisco.ios` collection, Nautobot API access, and Cisco device
connection credentials supplied by the inventory or Ansible Vault. Each
matching Nautobot device must have a primary IPv4 address.

`nautobot_url` and `nautobot_token` must be defined in inventory or group vars.
`serialnum_check_only_nb_device_role` defaults to `network`;
`serialnum_check_only_nb_log_file` sets the missing-serial log path; and
`serialnum_check_only_nb_validate_certs` defaults to `true`.

Run from the `nautobot/` directory with an inventory that provides Cisco login
credentials for the devices returned by Nautobot:

  ansible-playbook -i ./hostfiles/inventory pb_serialnum_check_only_nb.yml

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
