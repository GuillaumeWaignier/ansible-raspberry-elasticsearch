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

The first install, you need to create manually the users:
```bash
# remove elasticsearch.keystore in config dir
rm sudo rm /elk/config/elasticsearch.keystore
# Restart elasticsearch
sudo systemctl restart elasticsearch.service
# sudo as elasticsearch user
sudo su elastic
# Export env (shown in elk.env)
export ES_PATH_CONF=/elk/elasticsearch/config/
# Generate password
/elk/elasticsearch/elasticsearch/bin/elasticsearch-setup-passwords interactive
```


# Example 1

- Raspberry Pi 4 (4 cores, 4Gb ram)
- Raspberry Pi OS 32 bits (armv7l)
- Listen on localhost / exposed behind a proxy

```yaml
- hosts: all
  vars_prompt:
    - name: password
      prompt: elasticsearch password
  tasks:
    - set_fact: password={{ password }}
      no_log: true

- name: Install elasticsearch
  hosts: elasticsearch
  roles:
    - role: elasticsearch
      tags: elasticsearch
      vars:
        - network_host: "localhost"
        - heap: "254m"
        - version: "7.10.2"


- name: Install kibana
  hosts: kibana
  roles:
    - role: kibana
      tags: kibana
      vars:
        - expose_on_localhost: "yes"
        - server_basepath: "/kibana"
        - version: "7.10.2"
        - node_version: "10.23.1"
```

> 32 bits ARM works upto kibana version 7.10


# Example 2

- NUC Intel 11 Pro (i5, 32Gb ram)
- OS linux 64 bits (x86_64)
- Exposed on public

```yaml
# Install Kibana and Elasticserach
# Elasticsearch is exposed on port 9200
# Kibana is exposed on port 5601
- hosts: all
  vars_prompt:
    - name: password
      prompt: elasticsearch password
  tasks:
    - set_fact: password={{ password }}
      no_log: true

- name: Create all certificates
  hosts: certificate
  roles:
    - role: certificate
      tags: certificate
  vars:
    - country_name: FR
    - organization_name: Perso
    - organizational_unit_name: IT Department
    - common_name: ca
  vars_prompt:
    - name: ca_passphrase
      prompt: CA password

- name: Install elasticsearch
  hosts: elasticsearch
  roles:
  - role: elasticsearch
    tags: elasticsearch
    vars:
      - network_host: "0.0.0.0"
      - heap: 8g
      - version: 8.8.2

- name: Install kibana
  hosts: kibana
  roles:
  - role: kibana
    tags: kibana
    vars:
      - expose_on_localhost: "no"
      - server_basepath: ""
      - version: 8.8.2

- name: Install metricbeat
  hosts: all
  roles:
    - role: metricbeat
      tags: metricbeat
      vars:
        - version: 8.8.2

- name: Install filebeat
  hosts: all
  roles:
    - role: filebeat
      tags: filebeat
      vars:
        - version: 8.8.2
```