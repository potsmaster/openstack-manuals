===================
Manage IP addresses
===================

Each instance has a private, fixed IP address (assigned when launched)
and can also have a public, or floating, address. Private IP addresses
are used for communication between instances, and public addresses are
used for communication with networks outside the cloud, including the
Internet.

- By default, both administrative and end users can associate floating IP
  addresses with projects and instances. You can change user permissions for
  managing IP addresses by updating the ``/etc/nova/policy.json``
  file. For basic floating-IP procedures, refer to the *Manage IP
  Addresses* section in the `OpenStack End User Guide <http://docs.openstack.org/user-guide/>`_.

- For details on creating public networks using OpenStack Networking
  (``neutron``), refer to the `OpenStack Cloud Administrator Guide
  <http://docs.openstack.org/admin-guide-cloud/content/>`_. No
  floating IP addresses are created by default in OpenStack Networking.

As an administrator using legacy networking (``nova-network``), you
can use the following bulk commands to list, create, and delete ranges
of floating IP addresses. These addresses can then be associated with
instances by end users.

List addresses for all projects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To list all floating IP addresses for all projects, run::

  $ nova floating-ip-bulk-list
  +------------+---------------+---------------+--------+-----------+
  | project_id | address       | instance_uuid | pool   | interface |
  +------------+---------------+---------------+--------+-----------+
  | None       | 172.24.4.225  | None          | public | eth0      |
  | None       | 172.24.4.226  | None          | public | eth0      |
  | None       | 172.24.4.227  | None          | public | eth0      |
  | None       | 172.24.4.228  | None          | public | eth0      |
  | None       | 172.24.4.229  | None          | public | eth0      |
  | None       | 172.24.4.230  | None          | public | eth0      |
  | None       | 172.24.4.231  | None          | public | eth0      |
  | None       | 172.24.4.232  | None          | public | eth0      |
  | None       | 172.24.4.233  | None          | public | eth0      |
  | None       | 172.24.4.234  | None          | public | eth0      |
  | None       | 172.24.4.235  | None          | public | eth0      |
  | None       | 172.24.4.236  | None          | public | eth0      |
  | None       | 172.24.4.237  | None          | public | eth0      |
  | None       | 172.24.4.238  | None          | public | eth0      |
  | None       | 192.168.253.1 | None          | test   | eth0      |
  | None       | 192.168.253.2 | None          | test   | eth0      |
  | None       | 192.168.253.3 | None          | test   | eth0      |
  | None       | 192.168.253.4 | None          | test   | eth0      |
  | None       | 192.168.253.5 | None          | test   | eth0      |
  | None       | 192.168.253.6 | None          | test   | eth0      |
  +------------+---------------+---------------+--------+-----------+

Bulk create floating IP addresses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To create a range of floating IP addresses, run::

$ nova floating-ip-bulk-create [--pool POOL_NAME] [--interface INTERFACE] RANGE_TO_CREATE

For example::

  $ nova floating-ip-bulk-create --pool test 192.168.1.56/29

By default, **floating-ip-bulk-create** uses the
``public`` pool and ``eth0`` interface values.

.. note:: You should use a range of free IP addresses that is correct for your
  network. If you are not sure, at least try to avoid the DHCP address
  range:

  - Pick a small range (/29 gives an 8 address range, 6 of
    which will be usable).

  - Use **nmap** to check a range's availability. For example,
    192.168.1.56/29 represents a small range of addresses
    (192.168.1.56-63, with 57-62 usable), and you could run the
    command **nmap -sn 192.168.1.56/29** to check whether the entire
    range is currently unused.

Bulk delete floating IP addresses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To delete a range of floating IP addresses, run::

  $ nova floating-ip-bulk-delete RANGE_TO_DELETE

For example::

  $ nova floating-ip-bulk-delete 192.168.1.56/29
