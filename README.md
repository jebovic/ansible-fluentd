FluentD
=======

[![Build Status](https://travis-ci.org/jebovic/ansible-fluentd.svg?branch=master)](https://travis-ci.org/jebovic/ansible-fluentd) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-jebovic.fluentd-blue.svg?style=flat)](https://galaxy.ansible.com/jebovic/fluentd)

Install and configure fluentd

This role is a part of my [OPS project](https://github.com/jebovic/ops), follow this link to see it in action. OPS provides a lot of stuff, like a vagrant file for development VMs, playbooks for roles orchestration, inventory files, examples for roles configuration, ansible configuration file, and many more.

Compatibility
-------------

Tested and approved on :

* Debian jessie (8+)
* Ubuntu Trusty (14.04 LTS)
* Ubuntu Xenial (16.04 LTS)

Role Variables
--------------

```yaml
# fluentd install configuration
fluentd_apt_key_url: https://packages.treasuredata.com/GPG-KEY-td-agent
fluentd_apt_repositories:
  - "deb http://packages.treasuredata.com/2/{{ ansible_distribution | lower }}/{{ ansible_distribution_release | lower }}/ {{ ansible_distribution_release | lower }} contrib"
fluentd_packages:
  - td-agent
fluentd_daemon: td-agent
fluentd_config_path: /etc/td-agent
fluentd_log_path: /var/log/td-agent
fluentd_plugins: []
fluentd_user: td-agent
fluentd_usergroup: td-agent

# fluentd basic configuration
fluentd_system_config:
  process_name: ansible-fluentd

fluentd_source_config:
  - "@type": forward
    port: 24224
  - "@type": http
    port: 9880

fluentd_match_config:
  - pattern: "myapp.access"
    "@type": file
    path: /var/log/td-agent/myapp.access
    time_slice_format: "%Y%m%d"
    time_slice_wait: 10m
    time_format: "%Y%m%dT%H%M%S%z"

fluentd_filter_config:
  - name: myapp.access
    "@type": record_transformer
    record:
      host_param: '"#{Socket.gethostname}"'
```

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
     - { role: jebovic.fluentd }
```

Example : config
----------------

Fluentd configuration for client (any instance which want to send messages to the central fluentd):

```yaml
# Source config, tail nginx access logs
fluentd_source_config:
  - "@type": tail
    path: "/var/log/nginx/example.com.access.log"
    pos_file: "/var/log/td-agent/example.com.access.log.pos"
    tag: nginx.access
    format: nginx

# Match config, send messages from nginx.access tag to another server fluentd instance
fluentd_match_config:
  - pattern: nginx.access
    "@type": forward
    send_timeout: 60s
    recover_wait: 10s
    heartbeat_interval: 1s
    phi_threshold: 16
    hard_timeout: 60s
    server:
      name: monitoring.local
      host: 192.168.1.50
      port: 24224
      weight: 60

# Add field from_host with current hostname
fluentd_filter_config:
  - name: nginx.access
    "@type": record_transformer
    record:
      from_host: '"#{Socket.gethostname}"'
```

Fluentd configuration for server (central fluent, aside elasticsearch instance:

```yaml
# Additional packages (geoip)
fluentd_packages:
  - td-agent
  - build-essential
  - libgeoip-dev

# fluentd additional plugins
fluentd_plugins:
  - fluent-plugin-elasticsearch
  - fluent-plugin-geoip

fluentd_system_config:
  process_name: fluentd-central

# Source config, listen on port 24224 for messages reception
fluentd_source_config:
  - "@type": forward
    port: 24224

# Send nginx.access tagged messages to elasticsearch
fluentd_match_config:
  - pattern: "*nginx.access"
    "@type": elasticsearch
    host: localhost
    port: 9201
    type_name: nginx_access
    logstash_format: "true"
    logstash_prefix: nginx_access

# Add location field qith geoip plugin
fluentd_filter_config:
  - name: nginx.access
    "@type": geoip
    geoip_lookup_key: remote
    record:
      location: "'[${longitude[\"remote\"]},${latitude[\"remote\"]}]'"
      latitude: '${latitude["remote"]}'
      longitude: '${longitude["remote"]}'
      country_code3: '${country_code3["remote"]}'
      country: '${country_code["remote"]}'
      country_name: '${country_name["remote"]}'
      dma: '${dma_code["remote"]}'
      area: '${area_code["remote"]}'
      region: '${region["remote"]}'
      city: '${city["remote"]}'
    tag: "geoip.${tag}"
    skip_adding_null_record: "true"
```

Tags
----

* fluentd_config : only update config and restart service
* fluentd_plugins : only install plugins and restart service

License
-------

MIT

Author Information
------------------

Jérémy Baumgarth https://github.com/jebovic
