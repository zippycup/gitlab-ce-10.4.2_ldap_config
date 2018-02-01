# gitlab-ce-10.4.2_ldap_config

## Description

Understand to correctly configure gitlab Community Edition for use with OpenLDAP.

## Issue 1 /var/log/gitlab/sidekiq/current
```
---- Begin output of /opt/gitlab/bin/gitlab-rake cache:clear ----
STDOUT:
STDERR: rake aborted!
Devise::OmniAuth::StrategyNotFound: Could not find a strategy with name `Ldapsecondary'. Please ensure it is required or explicitly set it using the :strategy_class option                          .
Tasks: TOP => cache:clear => environment
(See full trace by running task with --trace)
---- End output of /opt/gitlab/bin/gitlab-rake cache:clear ----
```
This is caused by Community Eddition only supporting the main: ldap server.

## Issue 2 /var/log/gitlab/gitlab-rails/production.log
```
Started POST "/users/auth/ldapmain/callback" for 10.27.52.101 at 2018-02-01 10:47:00 -0500

ArgumentError (base MUST be provided):
  lib/gitlab/middleware/multipart.rb:93:in `call'
  lib/gitlab/request_profiler/middleware.rb:14:in `call'
  lib/gitlab/middleware/go.rb:18:in `call'
  lib/gitlab/etag_caching/middleware.rb:11:in `call'
  lib/gitlab/middleware/read_only.rb:31:in `call'
  lib/gitlab/request_context.rb:18:in `call'
  lib/gitlab/metrics/requests_rack_middleware.rb:27:in `call'

Raven 2.5.3 configured not to capture errors: DSN not set
LDAP search error: No Such Object
```
Your web page will display, 500 'Whoops, something went wrong on our end.'
This is caused by not setting the ldap base.

## Working example /etc/gitlab/gitlab.rb
```
gitlab_rails['ldap_enabled'] = true

###! **remember to close this block with 'EOS' below**
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
main: # 'main' is the GitLab 'provider ID' of this LDAP server
  label: 'LDAP'
  host: 'ldap-1.mydomain.net'
  port: 389
  uid: 'uid'
  bind_dn: 'cn=myadminuser,dc=mydomain,dc=net'
  password: 'mypassword'
  base: 'ou=People,dc=mydomain,dc=net'
  encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     active_directory: true
#     allow_username_or_email_login: false
#     block_auto_created_users: false
#     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
#
# gitlab CE ( community edition only supports one LDAP server )
#secondary: # 'secondary' is the GitLab 'provider ID' of second LDAP server
#  label: 'LDAP'
#  host: 'ldap-2.mydomain.net'
#  port: 389
#  uid: 'uid'
#  bind_dn: 'cn=myadminuser,dc=mydomain,dc=net'
#  password: 'mypassword'
#  base: 'ou=People,dc=mydomain,dc=net'
#  encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     active_directory: true
#     allow_username_or_email_login: false
#     block_auto_created_users: false
#     base: ''
#     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
EOS

```
