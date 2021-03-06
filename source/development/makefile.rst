.. Copyright (C) 2019 Wazuh, Inc.

.. _wazuh_makefile:

Makefile options
================

This section contains instructions to customize the installation from Wazuh by compiling the source code before executing the installation script.

You can also find here the different settings available for the ``Makefile``. Each setting is described and includes the default and allowed values that you can use.

Compiling the source code
-------------------------

When following the official documentation to install the Wazuh manager :ref:`from sources <sources_installation>` (or the :ref:`Wazuh agent <agent-sources>`), the user runs the ``install.sh`` script. This will automatically compile the source code before installing it, but some customizations can be made prior to the script execution.

To compile the code with ``make``, the working directory must be where the ``MAKEFILE`` resides, in this case, the ``/src`` directory of the installation folder:

.. code-block:: console

  # cd wazuh/src
  # make deps
  # make <OPTIONS>

After compiling the source code, now you can execute the installation script:

.. code-block:: console

  # cd ../
  # ./install.sh

.. warning::
  Some dependencies must be downloaded before compiling. If ``make deps`` is not executed before that, an error message will appear asking the user to do it.

Makefile reference
------------------

Available targets
^^^^^^^^^^^^^^^^^

+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **deps**              | Download external dependencies, required for compiling the code. Requires Internet connectivity.                       |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **external**          | Compile the external dependencies. Will be done automatically when using ``build``.                                    |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **build**             | Compile the source code. Requires external dependencies and a ``TARGET`` flag.                                         |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **utils**             | Compile the complementary tools used by Wazuh binaries.                                                                |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **test-rules**        | Run test suite for rules and decoders.                                                                                 |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean**             | Removes all contents, including compiled files, including external dependencies, tests, and configuration.             |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean-deps**        | Removes all external dependencies, including downloaded files.                                                         |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean-external**    | Removes compiled external dependencies, but won't remove downloaded files.                                             |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean-internals**   | Removes all compiled internal dependencies.                                                                            |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean-framework**   | Removes all compiled files used to build the Wazuh framework.                                                          |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean-windows**     | Removes all compiled files used to build the Windows agent.                                                            |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean-config**      | Removes all compiled configuration files.                                                                              |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **clean-test**        | Removes all compiled files used for testing.                                                                           |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+

There are other targets used to get information about the Makefile, but they won't build, download or compile anything:

+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **help**              | Show information about the Makefile.                                                                                   |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **settings**          | Show default values of compilation flags.                                                                              |
+-----------------------+------------------------------------------------------------------------------------------------------------------------+

Available flags
^^^^^^^^^^^^^^^

+-----------------------+------------------------------------------------------------------------------------------------------------------------+
| **TARGET**            | Defines the type of installation to build.                                                                             |
|                       |                                                                                                                        |
|                       | The most common are ``server`` to compile a manager, and ``agent/winagent``                                            |
|                       | to compile agents.                                                                                                     |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | n/a                                                                                                 |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | server, local, hybrid, agent, winagent                                                              |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **V**                 | Display full compiler messages.                                                                                        |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | 0                                                                                                   |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | 0, 1                                                                                                |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **DEBUG**             | Build with symbols and without optimization.                                                                           |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | 0                                                                                                   |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | 0, 1                                                                                                |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **DEBUGAD**           | Enables extra debugging logging in ``ossec-analysisd``.                                                                |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | 0                                                                                                   |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | 0, 1                                                                                                |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **PREFIX**            | Install Wazuh to the specified absolute path.                                                                          |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | /var/ossec                                                                                          |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | Any valid absolute path.                                                                            |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **MAXAGENTS**         | Set the number of maximum agents.                                                                                      |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | 14000                                                                                               |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | Any number.                                                                                         |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **ONEWAY**            | Disables manager's ACK towards the agent. It allows connecting agents without a backward connection from the manager.  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **CLEANFULL**         | Makes the alert mailing subject clear in the format: ``<location> - <level> - <description>``                          |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **RESOURCES_URL**     | Set the Wazuh resources URL.                                                                                           |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | ``https://packages.wazuh.com/deps/$(VERSION)``                                                      |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | Any valid URL string.                                                                               |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_ZEROMQ**        | Build with ZeroMQ support.                                                                                             |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_PRELUDE**       | Build with Prelude support.                                                                                            |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_INOTIFY**       | Build with Inotify support.                                                                                            |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_MSGPACK_OPT**   | Build with Msgpack full optimization.                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | yes                                                                                                 |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **BIG_ENDIAN**        | Build with big endian support.                                                                                         |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_SELINUX**       | Build with SELinux policies.                                                                                           |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_AUDIT**         | Build with audit service support.                                                                                      |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_FRAMEWORK_LIB** | Use external SQLite library for the framework.                                                                         |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **USE_GEOIP**         | Build with GeoIP support.                                                                                              |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | no                                                                                                  |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, no                                                                                             |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **DATABASE**          | Build with database support. Allows support for MySQL or PostgreSQL.                                                   |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | n/a                                                                                                 |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | mysql, pgsql                                                                                        |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **OSSEC_GROUP**       | Defines the OSSEC group.                                                                                               |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | ossec                                                                                               |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | Any string.                                                                                         |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **OSSEC_USER**        | Defines the OSSEC user.                                                                                                |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | ossec                                                                                               |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | Any string.                                                                                         |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **OSSEC_USER_MAIL**   | Defines the OSSEC user mail.                                                                                           |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | ossecm                                                                                              |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | Any string.                                                                                         |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **OSSEC_USER_REM**    | Defines the OSSEC user rem.                                                                                            |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | ossecr                                                                                              |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | Any string.                                                                                         |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **DISABLE_SHARED**    | Disable the compilation of Wazuh shared libraries and use static libraries.                                            |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | n/a                                                                                                 |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, true                                                                                           |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **DISABLE_SYSC**      | Disable the compilation of the Syscollector module.                                                                    |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | n/a                                                                                                 |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, true                                                                                           |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
| **DISABLE_CISCAT**    | Disable the compilation of the CIS-CAT module.                                                                         |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Default value    | n/a                                                                                                 |
|                       +------------------+-----------------------------------------------------------------------------------------------------+
|                       | Allowed values   | yes, true                                                                                           |
+-----------------------+------------------+-----------------------------------------------------------------------------------------------------+
