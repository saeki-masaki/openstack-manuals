=================
Identity concepts
=================

Authentication
    The process of confirming the identity of a user.
    OpenStack Identity confirms an incoming request
    by validating a set of credentials supplied by the
    user. These credentials are initially a user name and
    password, or a user name and API key. When user
    credentials are validated, OpenStack Identity issues an
    authentication token which the user provides in subsequent
    requests.

Credentials
    Data that confirms the user's identity. For
    example: user name and password, user name and API
    key, or an authentication token provided by the
    Identity service.

Domain
    A domain is a collection of projects and users. It defines administrative
    boundaries for the management of Identity entities. A domain may represent
    an individual, company, or operator-owned space. It is used for exposing
    administrative activities directly to the system users. Users may be given
    a domain's administrator role. A domain administrator may create
    projects, users, and groups within a domain and assign roles to users
    and groups.

Endpoint
    A network-accessible address where you access a service,
    usually a URL address. If you are using an extension for
    templates, an endpoint template can be created, which
    represents the templates of all the consumable services
    that are available across the regions.

OpenStackClient
    A command-line interface for several OpenStack services
    including the Identity API. For example, users can run the
    :command:`openstack service create` and
    :command:`openstack endpoint create` commands
    to register services in their OpenStack
    installations.

Project
    A container used to group or isolate resources.
    Projects also group or isolate identity objects.
    Depending on the service operator, a project may map
    to a customer, account, organization, or tenant.

Role
    A personality with a defined set of user rights and
    privileges to perform a specific set of operations.
    In the Identity service, a token that is issued
    to a user includes the list of roles. Services that are
    being called by that user determine how they interpret the
    set of roles a user has and to which operations or
    resources each role grants access.

Service
    An OpenStack service, such as Compute (nova),
    Object Storage (swift), or Image service (glance).
    It provides one or more endpoints in which
    users can access resources and perform operations.

Token
    An alpha-numeric string of text used to access
    OpenStack APIs and resources. A token may be
    revoked at any time and is valid for a
    finite duration. While OpenStack Identity supports token-based
    authentication in this release, the intention is
    to support additional protocols in the future.
    Its main purpose is to be an integration service,
    and not aspire to be a full-fledged identity store
    and management solution.

User
    Digital representation of a person, system, or
    service who uses OpenStack cloud services. The
    Identity service validates that incoming requests
    are made by the user who claims to be making the
    call. Users have a login and may be assigned
    tokens to access resources. Users can be directly
    assigned to a particular project and behave as if
    they are contained in that project.

User management
~~~~~~~~~~~~~~~

Identity user management examples:

* Create a user named ``alice``:

  .. code-block:: console

     $ openstack user create --password-prompt --email alice@example.com alice

* Create a project named ``acme``:

  .. code-block:: console

     $ openstack project create acme

* Create a domain named ``emea``:

  .. code-block:: console

     $ openstack --os-identity-api-version=3 domain create emea

* Create a role named ``compute-user``:

  .. code-block:: console

     $ openstack role create compute-user

  .. note::

     Individual services, such as Compute and the Image service,
     assign meaning to roles. In the Identity service, a role is
     simply a name.

The Identity service assigns a tenant and a role to a user. You might
assign the ``compute-user`` role to the ``alice`` user in the ``acme``
tenant:

.. code::

    $ openstack user list
    +--------+-------+
    | ID     | Name  |
    +--------+-------+
    | 892585 | alice |
    +--------+-------+

.. code::

    $ openstack role list
    +--------+---------------+
    | ID     | Name          |
    +--------+---------------+
    | 9a764e | compute-user  |
    +--------+---------------+

.. code::

    $ openstack project list
    +--------+--------------------+
    | ID     | Name               |
    +--------+--------------------+
    | 6b8fd2 | acme               |
    +--------+--------------------+

.. code::

    $ openstack role add --project 6b8fd2 --user 892585 9a764e

A user can have different roles in different tenants. For example, Alice
might also have the ``admin`` role in the ``Cyberdyne`` tenant. A user
can also have multiple roles in the same tenant.

The :file:`/etc/[SERVICE_CODENAME]/policy.json` file controls the tasks that
users can perform for a given service. For example,
:file:`/etc/nova/policy.json` file specifies the access policy for the Compute
service, :file:`/etc/glance/policy.json` file specifies the access policy for
the Image service, and :file:`/etc/keystone/policy.json` file specifies the
access policy for the Identity service.

The default :file:`policy.json` files in the Compute, Identity, and Image
service recognize only the ``admin`` role; all operations that do not
require the ``admin`` role are accessible by any user that has any role
in a tenant.

To restrict users from performing operations in, for example, the
Compute service, you need to create a role in the Identity service and
then modify the :file:`/etc/nova/policy.json` file so that this role
is required for Compute operations.

For example, the following line in the :file:`/etc/nova/policy.json` file
specifies that there are no restrictions on which users can create volumes:

.. code:: json

    "volume:create": "",

If the user has any role in a tenant, they can create volumes in that tenant.

To restrict the creation of volumes to users who had the ``compute-user``
role in a particular tenant, you would add ``"role:compute-user"``:

.. code:: json

    "volume:create": "role:compute-user",

To restrict all Compute service requests to require this role, the
resulting file would look like:

.. code-block:: json
   :linenos:

   {
      "admin_or_owner": "role:admin or project_id:%(project_id)s",
      "default": "rule:admin_or_owner",
      "compute:create": "role:compute-user",
      "compute:create:attach_network": "role:compute-user",
      "compute:create:attach_volume": "role:compute-user",
      "compute:get_all": "role:compute-user",
      "compute:unlock_override": "rule:admin_api",
      "admin_api": "role:admin",
      "compute_extension:accounts": "rule:admin_api",
      "compute_extension:admin_actions": "rule:admin_api",
      "compute_extension:admin_actions:pause": "rule:admin_or_owner",
      "compute_extension:admin_actions:unpause": "rule:admin_or_owner",
      "compute_extension:admin_actions:suspend": "rule:admin_or_owner",
      "compute_extension:admin_actions:resume": "rule:admin_or_owner",
      "compute_extension:admin_actions:lock": "rule:admin_or_owner",
      "compute_extension:admin_actions:unlock": "rule:admin_or_owner",
      "compute_extension:admin_actions:resetNetwork": "rule:admin_api",
      "compute_extension:admin_actions:injectNetworkInfo": "rule:admin_api",
      "compute_extension:admin_actions:createBackup": "rule:admin_or_owner",
      "compute_extension:admin_actions:migrateLive": "rule:admin_api",
      "compute_extension:admin_actions:migrate": "rule:admin_api",
      "compute_extension:aggregates": "rule:admin_api",
      "compute_extension:certificates": "role:compute-user",
      "compute_extension:cloudpipe": "rule:admin_api",
      "compute_extension:console_output": "role:compute-user",
      "compute_extension:consoles": "role:compute-user",
      "compute_extension:createserverext": "role:compute-user",
      "compute_extension:deferred_delete": "role:compute-user",
      "compute_extension:disk_config": "role:compute-user",
      "compute_extension:evacuate": "rule:admin_api",
      "compute_extension:extended_server_attributes": "rule:admin_api",
      "compute_extension:extended_status": "role:compute-user",
      "compute_extension:flavorextradata": "role:compute-user",
      "compute_extension:flavorextraspecs": "role:compute-user",
      "compute_extension:flavormanage": "rule:admin_api",
      "compute_extension:floating_ip_dns": "role:compute-user",
      "compute_extension:floating_ip_pools": "role:compute-user",
      "compute_extension:floating_ips": "role:compute-user",
      "compute_extension:hosts": "rule:admin_api",
      "compute_extension:keypairs": "role:compute-user",
      "compute_extension:multinic": "role:compute-user",
      "compute_extension:networks": "rule:admin_api",
      "compute_extension:quotas": "role:compute-user",
      "compute_extension:rescue": "role:compute-user",
      "compute_extension:security_groups": "role:compute-user",
      "compute_extension:server_action_list": "rule:admin_api",
      "compute_extension:server_diagnostics": "rule:admin_api",
      "compute_extension:simple_tenant_usage:show": "rule:admin_or_owner",
      "compute_extension:simple_tenant_usage:list": "rule:admin_api",
      "compute_extension:users": "rule:admin_api",
      "compute_extension:virtual_interfaces": "role:compute-user",
      "compute_extension:virtual_storage_arrays": "role:compute-user",
      "compute_extension:volumes": "role:compute-user",
      "compute_extension:volume_attachments:index": "role:compute-user",
      "compute_extension:volume_attachments:show": "role:compute-user",
      "compute_extension:volume_attachments:create": "role:compute-user",
      "compute_extension:volume_attachments:delete": "role:compute-user",
      "compute_extension:volumetypes": "role:compute-user",
      "volume:create": "role:compute-user",
      "volume:get_all": "role:compute-user",
      "volume:get_volume_metadata": "role:compute-user",
      "volume:get_snapshot": "role:compute-user",
      "volume:get_all_snapshots": "role:compute-user",
      "network:get_all_networks": "role:compute-user",
      "network:get_network": "role:compute-user",
      "network:delete_network": "role:compute-user",
      "network:disassociate_network": "role:compute-user",
      "network:get_vifs_by_instance": "role:compute-user",
      "network:allocate_for_instance": "role:compute-user",
      "network:deallocate_for_instance": "role:compute-user",
      "network:validate_networks": "role:compute-user",
      "network:get_instance_uuids_by_ip_filter": "role:compute-user",
      "network:get_floating_ip": "role:compute-user",
      "network:get_floating_ip_pools": "role:compute-user",
      "network:get_floating_ip_by_address": "role:compute-user",
      "network:get_floating_ips_by_project": "role:compute-user",
      "network:get_floating_ips_by_fixed_address": "role:compute-user",
      "network:allocate_floating_ip": "role:compute-user",
      "network:deallocate_floating_ip": "role:compute-user",
      "network:associate_floating_ip": "role:compute-user",
      "network:disassociate_floating_ip": "role:compute-user",
      "network:get_fixed_ip": "role:compute-user",
      "network:add_fixed_ip_to_instance": "role:compute-user",
      "network:remove_fixed_ip_from_instance": "role:compute-user",
      "network:add_network_to_project": "role:compute-user",
      "network:get_instance_nw_info": "role:compute-user",
      "network:get_dns_domains": "role:compute-user",
      "network:add_dns_entry": "role:compute-user",
      "network:modify_dns_entry": "role:compute-user",
      "network:delete_dns_entry": "role:compute-user",
      "network:get_dns_entries_by_address": "role:compute-user",
      "network:get_dns_entries_by_name": "role:compute-user",
      "network:create_private_dns_domain": "role:compute-user",
      "network:create_public_dns_domain": "role:compute-user",
      "network:delete_dns_domain": "role:compute-user"
   }

Service management
~~~~~~~~~~~~~~~~~~

The Identity service provides identity, token, catalog, and policy
services. It consists of:

* keystone Web Server Gateway Interface (WSGI) service
    Can be run in a WSGI-capable web server such as Apache httpd to provide
    the Identity service. The service and administrative APIs are run as
    separate instances of the WSGI service.

* Identity service functions
    Each has a pluggable back end that allow different ways to use the
    particular service. Most support standard back ends like LDAP or SQL.

* keystone-all
    Starts both the service and administrative APIs in a single process.
    Using federation with keystone-all is not supported. keystone-all is
    deprecated in favor of the WSGI service.

The Identity service also maintains a user that corresponds to each
service, such as, a user named ``nova`` for the Compute service, and a
special service tenant called ``service``.

For information about how to create services and endpoints, see the
`OpenStack Admin User Guide <http://docs.openstack.org/user-guide-admin/index.html>`__.

Groups
~~~~~~

A group is a collection of users in a domain. Administrators can create groups
and add users to them. A role can then be assigned to the group, rather than
individual users. Groups were introduced with the Identity API v3.

Identity API V3 provides the following group-related operations:

* Create a group

* Delete a group

* Update a group (change its name or description)

* Add a user to a group

* Remove a user from a group

* List group members

* List groups for a user

* Assign a role on a tenant to a group

* Assign a role on a domain to a group

* Query role assignments to groups

.. note::

    The Identity service server might not allow all operations. For
    example, if using the Identity server with the LDAP Identity back
    end and group updates are disabled, then a request to create,
    delete, or update a group fails.

Here are a couple of examples:

* Group A is granted Role A on Tenant A. If User A is a member of Group
  A, when User A gets a token scoped to Tenant A, the token also
  includes Role A.

* Group B is granted Role B on Domain B. If User B is a member of
  Domain B, if User B gets a token scoped to Domain B, the token also
  includes Role B.
