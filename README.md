[![Build Status](https://travis-ci.org/030/ansible-certbot.svg?branch=master)](https://travis-ci.org/030/ansible-certbot)

# ansible-certbot

Ensure that port 80 is available otherwise certbot will fail.

Installs certbot. Certbot is able to create certificates so that websites could be accessed securely.

A cert will be created based on the hostname of the system. Be sure that this is registered in an external DNS server as well.

## Role variables

By default, neither certs will be created, nor renewed.

    certbot_create_certs: false
    certbot_mail_address: mail@example.com
    certbot_renew_certs: false
    certbot_package: "certbot" # can be certbot, python-certbot-nginx, python-certbot-apache
    certbot_plugin: "standalone" # can be apache, webroot, nginx or standalone
    certbot_install_cert: false # false will use certonly, true will use the plugin configured to add the ssl configs to the sites config.
    certbot_site_names: ["example.com","www.example.com"] # List of domain names to get certificates for.
    certbot_cert_name: {{ certbot_site_names[0] }}
    certbot_renew_prehook: "service {{ certbot_webserver_installed }} stop"
    certbot_renew_posthook: "service {{ certbot_webserver_installed }} start"
    certbot_renew_certs_systemd_oncalendar: # After how long to run the systemd timer for auto-renewing certificates. Check [systemd time](https://www.freedesktop.org/software/systemd/man/systemd.time.html#) for acceptable formats.
    certbot_renew_certs_systemd_rand_sec: # Maximum random number of seconds to wait before actually running the renew command. Useful if you don't want to always hit the Let's Encrypt servers at a specific time
    certbot_renew_method: # Could either be cron or systemd

The email address that receives notifications when certs are going to be expired is empty by default and needs to be set otherwise the run will fail.

## Dependencies

None.

# Example usage

```yml
---
- hosts: website
  become: true
  vars:
    test_domain_name: "www.example.com"
    test_email_address: "info@example.com"
  pre_tasks:
    - name: Update apt cache
      sudo: yes
      apt: update_cache=yes

  roles:
    - role: nginx
      nginx_install_method: "package"
      nginx_sites:
        - server:
            name: "{{ test_domain_name }}"
            __listen: "80;"
            server_name: "{{ test_domain_name }}"
            ssl:
              enabled: false
      nginx_enabled_sites:
        - "{{ test_domain_name }}"
    - role: certbot
      vars:
        certbot_create_certs: true
        certbot_package:  "python-certbot-nginx"
        certbot_plugin:  "nginx"
        certbot_install_cert: true
        certbot_site_names:
          - "{{ test_domain_name }}"
          - "www.{{ test_domain_name }}"
        certbot_mail_address: "test_email_address"
