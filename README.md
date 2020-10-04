# ansible-role-minecraft

This role installs Minecraft or Spigot and configures it to run under systemd or Supervisor.

# Install

```
ansible-galaxy install totaldebug.minecraft
```

# Features

* supports vanilla Minecraft and Spigot
* supports Debian 10, Ubuntu 18.04, RHEL/CentOS 7 and RHEL/CentOS 8
* safely stops the server using stop when running under systemd
* Uses Github Actions & molecule to run integration tests
* manages user ACLs
* manages `server.properties`
* hooks: include arbitrary tasks at specific stages during execution

# Role Variables

Role variables
--------------

The following variable defaults are defined in ``defaults/main.yml``.

``minecraft_server_type``
  choose between ``minecraft`` or ``spigot`` (default: ``minecraft``)

``minecraft_version``
   Minecraft version to install (default: ``latest``)

   Examples:

   .. code:: yaml

       minecraft_version: latest
       minecraft_version: 1.10
       minecraft_version: 1.9.1
       minecraft_version: 16w21a

``minecraft_server_download_url``
   Minecraft download URL (default:
   ``https://launcher.mojang.com/v1/objects``)

``minecraft_server_download_checksum``
   Minecraft download URL Checksum

``minecraft_user``
   system user Minecraft runs as (default: ``minecraft``)

``minecraft_group``
   system group Minecraft runs as (default: ``minecraft``)

``minecraft_home``
   directory to install Minecraft to (default: ``/srv/minecraft``)

``minecraft_max_memory``
   Java max memory (``-Xmx``) to allocate (default: ``1024M``)

``minecraft_initial_memory``
   Java initial memory (``-Xms``) to allocate (default: ``1024M``)

``minecraft_service_name``
   systemd service name or Supervisor program name (default: ``minecraft``)

``minecraft_supervisor_name``
   **DEPRECATED:** Supervisor program name (default: ``{{ minecraft_service_name }}``)

``minecraft_process_control``
   Choose between ``systemd`` and ``supervisor`` (default: ``systemd``).

``minecraft_whitelist``
   list of Minecraft usernames to whitelist (default: ``[]``)

``minecraft_ops``
   list of Minecraft usernames to make server ops (default: ``[]``)

``minecraft_banned_players``
   list of Minecraft usernames to ban (default: ``[]``)

``minecraft_banned_ips``
   list of IP addresses to ban (default: ``[]``)

``minecraft_server_properties``
   dictionary of server.properties entries (e.g. ``server-port: 25565``) to set (default: ``{}``)

Hooks and run stages
--------------------

**ansible-minecraft** organizes execution into a number of run stages:

``setup``
   -  install prerequisites (e.g., Java)
   -  create Minecraft user and group

``download``
   -  fetch the latest version of from the launcher API
   -  download Minecraft

``install``
   -  symlink version to ``minecraft_server.jar``
   -  agree to EULA

``acl``
   -  configure server ACLs (whitelist, banned players, etc.)

``configure``
   -  set ``server.properties``

``start``
   -  (re)start server

You can execute custom tasks before or after specific stages. Simply specify a `task include file <https://docs.ansible.com/ansible/playbooks_roles.html#task-include-files-and-encouraging-reuse>`__ using the relevant role variable:

.. code:: yaml

    - hosts: minecraft
      roles:
        - role: devops-coop.minecraft
          minecraft_hook_before_start: "{{ playbook_dir }}/download-world-from-s3.yml"

The available hooks are:

``minecraft_hook_before_setup``
   run before ``setup`` tasks

``minecraft_hook_after_setup``
   run after ``setup`` tasks

``minecraft_hook_before_download``
   run before ``download`` tasks

``minecraft_hook_after_download``
   run after ``download`` tasks

``minecraft_hook_before_install``
   run before ``install`` tasks

``minecraft_hook_after_install``
   run after ``install`` tasks

``minecraft_hook_before_start``
   run before ``start`` tasks

``minecraft_hook_after_start``
   run after ``start`` tasks

Example
-------

.. code:: yaml

    - hosts: minecraft
      roles:
         - { role: devops-coop.minecraft, minecraft_whitelist: ["jeb_", "dinnerbone"]}

# Contributing

Pull requests are welcome. Among other features, this role lacks support for custom Minecraft servers.

## Versioning

This project follows semantic versioning.

In the context of semantic versioning, consider the role contract to be defined by the role variables.

* Breaking Changes or changes that require user intervention will increase the major version. This includes changing the default value of a role variable.
* Changes that do not require user intervention, but add new features, will increase the minor version.
* Bug fixes will increase the patch version.

