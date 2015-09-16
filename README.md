osbs-proxy
==========

This role provides Apache-based authenticating reverse proxy that can be placed
in front of OpenShift master to provide authentication based on Kerberos or
plain htpasswd file.

The role is part of
[ansible-osbs](https://github.com/projectatomic/ansible-osbs/) playbook for
deploying OpenShift build service. Please refer to that github repository for
[documentation](https://github.com/projectatomic/ansible-osbs/blob/master/README.md)
and [issue tracker](https://github.com/projectatomic/ansible-osbs/issues).

Role Variables
--------------

Generate self-signed certificate when the path set in `ssl_cert_file` does not
exist? Useful for development/debugging.

    osbs_proxy_ssl_generate_selfsigned: false

Certificate and the corresponding key the proxy will use to talk with outside
clients.

    osbs_proxy_ssl_cert_file: /etc/pki/tls/certs/fqdn.example.com.crt
    osbs_proxy_ssl_key_file: /etc/pki/tls/private/fqdn.example.com.key

Concatenated certificate and key used to authenticate to the proxied server.  If
unset, SSL client authentication will not be performed.

    osbs_proxy_ssl_client_certkey_file: /etc/httpd/openshift_proxy_certkey.crt

CA against which the certificate of the proxied server will be verified. If
unset, no verification takes place.

    osbs_proxy_ssl_client_ca_file: /etc/httpd/openshift_proxy_ca.crt

Path to the apache configuration snippet.

    osbs_proxy_httpd_conf_path: /etc/httpd/conf.d/openshift_proxy.conf

Authentication type. Supported values:
* htpasswd - uses static `.htpasswd` file
* kerberos - uses kerberos

    osbs_proxy_type: kerberos

Proxy will listen on this port.

    osbs_proxy_port: 9443

URL of the proxied web server.

    osbs_proxy_dest_url: https://127.0.0.1:8443/

Authentication realm used for basic authentication (e.g. when htpasswd auth is
used or `osbs_proxy_kerberos_password_login` set to `true`).

    osbs_proxy_authname: OpenShift Authentication

Permissions for sensitive files that should be accessible by the webserver.

    osbs_proxy_secrets_owner: apache
    osbs_proxy_secrets_group: root
    osbs_proxy_secrets_perms: "0600"

Manually assign openshift user account to an IP or IP range. Example:

    osbs_proxy_ip_whitelist:
      - subnet: 1.2.3.4
        user: kojibuilder

Location of the service keytab file the proxy will use.

    osbs_proxy_kerberos_keytab_file: /etc/httpd/HTTP-FQDN.EXAMPLE.COM.keytab

Kerberos: if no ticket is supplied, fall back to basic authentication by login+password
(through kerberos).

    osbs_proxy_kerberos_password_login: false

Kerberos realm to be used for authentication. If undefined it is taken from
kerberos configuration.

    osbs_proxy_kerberos_realm: EXAMPLE.COM

Location of the htpasswd file for `osbs_proxy_type == htpasswd`.

    osbs_proxy_htpasswd_file: /etc/httpd/openshift_proxy.htpasswd

List of user/password pairs to add to `osbs_proxy_htpasswd_file`. Useful for
development/debugging. Example:

    osbs_proxy_htpasswd_users:
      - user: test
        password: test

Extra verbose httpd logs?

    osbs_proxy_debug: false

Set to false if you don't use firewalld or do not want the playbook to modify it.

    osbs_manage_firewalld: true

Dependencies
------------

The role currently only works on the same machine as the OpenShift master. If
you are using the kerberos authentication type the machine also has to be
configured as a kerberos client.

Example Playbook
----------------

Simple htpasswd authentication for development:

    - hosts: builders
      roles:
        - role: osbs-proxy
          osbs_proxy_type: htpasswd
          osbs_proxy_ssl_generate_selfsigned: true
          osbs_proxy_htpasswd_users:
            - user: test
              password: test
          osbs_proxy_debug: true

Example kerberos setup:

    - hosts: builders:
      roles:
        - role: osbs-proxy
          osbs_proxy_type: kerberos
          osbs_proxy_ssl_cert_file: /etc/fqdn.example.com.crt
          osbs_proxy_ssl_key_file: /etc/fqdn.example.com.key
          osbs_proxy_kerberos_keytab_file: /etc/HTTP-FQDN.EXAMPLE.COM.keytab
          osbs_proxy_kerberos_realm: EXAMPLE.COM
          osbs_proxy_ip_whitelist:
            - subnet: 192.168.66.0/24
              user: kojibuilder

License
-------

BSD

Author Information
------------------

Martin Milata &lt;mmilata@redhat.com&gt;
