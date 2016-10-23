FluentD
=======

[![Build Status](https://travis-ci.org/jebovic/ansible-fluentd.svg?branch=master)](https://travis-ci.org/jebovic/ansible-fluentd) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-jebovic.fluentd-blue.svg?style=flat)](https://galaxy.ansible.com/jebovic/fluentd)

Install and configure fluentd

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
