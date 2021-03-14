# ansible-raspberry-elasticsearch
Ansible playbook used to install Elasticsearch and Kibana on a Raspberry Pi.


To install Elasticsearch and Kibana do:

```bash
ansible-playbook exposed-play.yml --ask-pass
```

To change parameters, such as version, create the file as

```yaml
- name: Install elasticsearch
  hosts: elasticsearch
  roles:
  - role: elasticsearch
    tags: elasticsearch
    vars:
      - version: "7.10.2"
      - network_host: "localhost"

- name: Install kibana
  hosts: kibana
  roles:
  - role: kibana
    tags: kibana
    vars:
      - version: "7.10.2"
      - server_host: "localhost"
      - server_basepath: "/kibana"
```

The host(s) of the PI are in file _inventory_.

