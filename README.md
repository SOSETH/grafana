# Grafana

Tested on: Debian 10, 11

## Configuration
| Var                                 | Default value                                     | Description                                                                                        |
|-------------------------------------|---------------------------------------------------|----------------------------------------------------------------------------------------------------|
| grafana_enable_repo                 | `False`                                           | Whether to enable the Grafana upstream apt repo                                                    |
| grafana_bind_address                | `127.0.0.1`                                       | Address to bind grafana on                                                                         |
| grafana_db_host                     | `grafana_db_host`                                 | Host where MySQL/MariaDB are running                                                               |
| grafana_db_password                 | empty                                             | Password to use to connect to the database                                                         |
| grafana_enable_ansible_db           | `False`                                           | Whether to use Ansible to create the database (useful for local db)                                |
| grafana_analytics                   | `True`                                            | Whether to enable analytics                                                                        |
| grafana_allow_signup                | `False`                                           | Whether to allow new user signup                                                                   |
| grafana_min_refresh_interval        | `30s`                                             | Whether to allow new user signup                                                                   |
| grafana_default_home_dashboard_path | ``                                                | Minimum dashboard refresh interval                                                                 |
| grafana_allow_org_create            | `True`                                            | Whether to allow non admin users to create organizations                                           |
| grafana_welcome_email_on_sign_up    | `True`                                            | Whether to send welcome email on signup                                                            |
| grafana_date_formats                | `dict` (see `defaults/main.yml`)                  | Settings to adjust date formats to the locale                                                      |
| grafana_ldap_hosts                  | `[]`                                              | List of hosts to connect for LDAP                                                                  |
| grafana_ldap_bind_user              | empty                                             | DN for the LDAP bind user                                                                          |
| grafana_ldap_bind_password          | empty                                             | Password for the LDAP bind user                                                                    |
| grafana_ldap_group_basedn           | empty                                             | BaseDN for LDAP group objects                                                                      |
| grafana_ldap_group_filter           | empty                                             | Filter expression for ldap groups                                                                  |
| grafana_ldap_user_basedn            | empty                                             | BaseDN for LDAP user objects                                                                       |
| grafana_ldap_user_filter            | `(uid=%s)`                                        | Filter expression for LDAP users                                                                   |
| grafana_ldap_is_ad                  | `False`                                           | Whether the LDAP in question is Microsoft Active Directory (enables ready-made filter expressions) |
| grafana_ldap_username               | `uid`                                             | LDAP user's username attribute                                                                     |
| grafana_ldap_memberof               | `cn`                                              | LDAP group's name attribute                                                                        |
| grafana_group_mappings              | `[]`                                              | Map of LDAP groups to Grafana groups                                                               |
| grafana_enable_keycloak             | `False`                                           | Whether to enable Keycloak OIDC support                                                            |
| grafana_keycloak_base               | empty                                             | Keycloak base URL                                                                                  |
| grafana_keycloak_clientid           | empty                                             | OIDC client ID                                                                                     |
| grafana_keycloak_clientsecret       | empty                                             | OIDC client secret                                                                                 |
| grafana_keycloak_domains            | empty                                             | Keycloak domains to allow                                                                          |
| grafana_session_provider            | `mysql`                                           | Session provider to use                                                                            |
| grafana_session_provider_options    | `user:password@tcp(127.0.0.1:3306)/database_name` | Database URL for the session provider                                                              |
| grafana_enable_new_alerts           | `False`                                           | Whether to enable the new Grafana 8.x alert system                                                 |

## Signon methods
All signon methods need some way to map groups from your user database to Grafana roles (since automated mapping to
Grafana groups is an enterprise feature). Example:
```
grafana_group_mappings:
  - name: monadmins
    role: Admin
```

### LDAP
When enabling LDAP, at this point, signup for new LDAP users is automatically enabled. You'll most likely need to
fine-tune the exact filter expressions you use, since they depend on the object structure.

#### Example provider: FreeIPA
```
grafana_ldap_hosts: ["dc1.<your domain>", "dc2.<your domain>"]
grafana_ldap_bind_user: "uid=sys_icinga_bind,cn=users,cn=accounts,dc=example,dc=org"
grafana_ldap_bind_password: "<password>"
grafana_ldap_group_basedn: "cn=groups,cn=accounts,dc=example,dc=org"
grafana_ldap_group_filter: "(&(objectClass=posixGroup)(member=uid=%s,cn=users,cn=accounts,dc=example,dc=org))"
grafana_ldap_user_basedn: "cn=users,cn=accounts,dc=example,dc=org"
```

#### Example provider: 389ds default
```
grafana_ldap_hosts: ["dc1.<your domain>", "dc2.<your domain>"]
grafana_ldap_bind_user: "cn=monbind,ou=TechnicalUsers,dc=example,dc=org"
grafana_ldap_bind_password: "<password>"
grafana_ldap_group_basedn: "ou=Groups,dc=example,dc=org"
grafana_ldap_group_filter: "(&(objectClass=groupOfNames)(member=uid=%s,ou=Users,dc=example,dc=org))"
grafana_ldap_user_basedn: "ou=Users,dc=example,dc=org"
```

### Keycloak
For OIDC, the exact setup is very IdP-specific - at this point, we explicitly support only Keycloak (though patches are
welcome). You'll need to create a new OIDC app, provide client ID and client secret as well as domains and the base
URL.
Just like for LDAP, automatic signup is enabled for Keycloak users by default.


## Alerting
Currently, Grafana 8.x style alerts can be enabled. The role does not yet support automatically clustering multiple
Grafana instances together for alerting, though we expect this to be added in the future
