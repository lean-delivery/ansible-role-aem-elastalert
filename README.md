AEM Elastalert role
=========
[![License](https://img.shields.io/badge/license-Apache-green.svg?style=flat)](https://raw.githubusercontent.com/lean-delivery/ansible-role-aem-elastalert/master/LICENSE)
[![Build Status](https://travis-ci.org/lean-delivery/ansible-role-aem-elastalert.svg?branch=master)](https://travis-ci.org/lean-delivery/ansible-role-aem-elastalert)
[![Build Status](https://gitlab.com/lean-delivery/ansible-role-aem-elastalert/badges/master/build.svg)](https://gitlab.com/lean-delivery/ansible-role-aem-elastalert/pipelines)
![Ansible](https://img.shields.io/ansible/role/d/38385.svg)
![Ansible](https://img.shields.io/badge/dynamic/json.svg?label=min_ansible_version&url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv1%2Froles%2F38385%2F&query=$.min_ansible_version)


## Summary


This role:
  - Free alerting for Elasticsearch, based on metrics
  - install elastalert on Ubuntu, CentOS
  - copies prepared configuration file (log path, connect to elasticsearch etc.)




Role tasks
------------


- Prepare server (add elastic repo)
- [Optional] Create folder(s) for custom paths
- Install elastalert
- Copy configuration file


Requirements
------------


- Minimal Version of the ansible for installation: 2.9
- AEM node version 7.5 with jolokia agent https://github.com/lean-delivery/ansible-role-aem-node
- Minimal Version of the Elastic stack: 7.5
- SNS alerting can work only with configured aws profile on instance where elastalert works
- Metricbeat https://github.com/lean-delivery/ansible-role-metricbeat
 - **Supported OS**:
   - CentOS
     - 7,8
   - Ubuntu
     - 16.04, 18.04
   - Debian
     - 8, 9


## Role Variables
--------------


You can override any variable below by setting "variable: value" in playbook.


- `es_host`
Elasticsearch host address. By default localhost
- `es_port`
Elasticsearch port. By default 9200
- `es_username`
Optional; basic-auth username for connecting to es_host.
- `es_password`
Optional; basic-auth password for connecting to es_host.
- `buffer_time`
size of the query window, stretching backwards from the time each query is run. By default 5 minutes
- `writeback_index`
Index where all rules will be stored
- `rules_config`
Custom rule that can will be placed into rules folder. Below you can see examples



## Advanced config parameters:


- `main_folder`
Default folder where config will be stored. By default /opt/elastalert
- `rules_folder`
Default folder where rules will be stored. Default is /opt/elastalert/rules
- `temp_folder`
Default folder where repo will be placed. By default it is /tmp/elastalert


## Dependencies
------------


Nothing


Example Playbook
----------------


### Installing latest elastalert:


```yaml
- name: Install AEM monitoring
  hosts: all
  roles:
    - role: ansible-role-aem-monitoring
  vars:
    es_host: localhost
    es_port: 9200
    run_every:
      minutes: 10
    buffer_time:
      hours: 1
```

### Installing latest elastalert with alert which has type change and sns notification:


```yaml
- name: Install AEM monitoring
  hosts: all
  roles:
    - role: ansible-role-aem-monitoring
  vars:
    es_host: localhost
    es_port: 9200
    run_every:
      minutes: 10
    buffer_time:
      hours: 1
    rules_config:
      rule1:
        name: Example rule
        type: change
        index: metricbeat-*
        compare_key: jolokia.jolokia_metrics.query.healthcheck.status
        ignore_null: true
        query_key: cloud.instance.id
        timeframe:
          minutes: 10
        filter:
        - query:
          - query_string:
              query: "jolokia.jolokia_metrics.memory.heap_usage.committed>1"
        alert: sns
        alert_subject: "Queue was broken"
        alert_text: "AEM queue was broken"
        sns_topic_arn: Your arn of your topic
        aws_access_key: access key
        aws_secret_key: secret key
        aws_region: your region
```

### Installing latest elastalert with alert which has type frequeny and command execution:


```yaml
- name: Install AEM monitoring
  hosts: all
  roles:
    - role: ansible-role-aem-monitoring
  vars:
    es_host: localhost
    es_port: 9200
    run_every:
      minutes: 10
    buffer_time:
      hours: 1
    rules_config:
      rule1:
        name: example_rule_1
        type: frequency
        num_events: 2
        index: metricbeat-*
        timeframe:
          minutes: 30
        filter:
        - query:
          - query_string:
              query: "system.cpu.idle.pct: <0.3"
        alert: command
        command: ["/bin/send_alert", "--username", "{match[username]}"]
      rule2:
        name: example_rule_2
        type: frequency
        num_events: 2
        index: metricbeat-*
        timeframe:
          minutes: 30
        filter:
        - query:
          - query_string:
              query: "system.cpu.idle.pct: <0.5"
        alert: command
        command: ["/bin/send_alert", "--username", "{match[username]}"]
```

License
-------
Apache


Author Information
------------------


authors:
  - Lean Delivery Team <team@lean-delivery.com>