.. Copyright (C) 2019 Wazuh, Inc.

.. _use-registration-service:

Using the registration service
==============================

The ``ossec-authd`` daemon allows to register agents automatically.

- The manager uses :ref:`ossec-authd` to launch the registration service.
- On the agent, :ref:`agent-auth` is used to connect to the registration service.

Launching the daemon on the manager with default options would allow any agent to register itself, and then connect to it. The secure methods provide some mechanisms to authorize the connections.

+------------+--------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
| Type       | Method                                                                                     | Description                                                                                                                 |
+============+============================================================================================+=============================================================================================================================+
| Not secure | `Simple method`_                                                                           | The easiest method. There is no authentication or host verification.                                                        |
+------------+--------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
| Secure     | `Password authorization`_                                                                  | Allows agents to authenticate via a shared password. This method is easy but does not perform host validation.              |
|            +--------------------------------+-----------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
|            | `Host verification using SSL`_ | `Manager verification using SSL`_                         | The manager's certificate is signed by a CA that agents use to validate the server. This may include host checking.         |
|            |                                +---------------------------------+-------------------------+-----------------------------------------------------------------------------------------------------------------------------+
|            |                                | `Agent verification using SSL`_ | With host validation    | The same as above, but the manager verifies the agent's certificate and address. There should be one certificate per agent. |
|            |                                |                                 +-------------------------+-----------------------------------------------------------------------------------------------------------------------------+
|            |                                |                                 | Without host validation | The manager validates the agent by CA but not the host address. This method allows the use of a shared agent certificate.   |
+------------+--------------------------------+---------------------------------+-------------------------+-----------------------------------------------------------------------------------------------------------------------------+

.. note::
  The secure methods can be combined for a stronger security during the registration process.

Prerequisites
-------------

The registration service requires an SSL certificate on the manager in order to work. If the system already has the ``openssl`` package, a new one will be generated automatically during the installation process. The certificate (and its key) will be available at ``/var/ossec/etc/``.

It's possible to use a valid certificate with its key, just by copying them into the same path:

.. code-block:: console

  # cp <ssl_cert> /var/ossec/etc/sslmanager.cert
  # cp <ssl_key> /var/ossec/etc/sslmanager.key

Otherwise, you can create a self-signed certificate using the following command:

.. code-block:: console

  # openssl req -x509 -batch -nodes -days 365 -newkey rsa:2048 -out /var/ossec/etc/sslmanager.cert -keyout /var/ossec/etc/sslmanager.key

.. note::

  From Fedora v22 to v25, it's required to install ``openssl`` package (``yum install openssl``).

Simple method
-------------

This is the easiest method to register agents. It doesn't require any kind of authorization or host verification. To do so, follow these steps:

1. On the agents, run the ``agent-auth`` program, pointing to the Wazuh manager address.

  .. code-block:: console

    # /var/ossec/bin/agent-auth -m <MANAGER_IP>

2. Edit the Wazuh agent configuration to add the Wazuh manager address.

  - In the file ``/var/ossec/etc/ossec.conf``, replace *MANAGER_IP* with the Wazuh manager address:

    .. code-block:: xml

      <client>
        <server>
          <address>MANAGER_IP</address>
          ...
        </server>
      </client>

  - Or using ``sed`` to replace it with the Wazuh manager address, using ``10.0.0.4`` as an example IP:

    .. code-block:: console

      # sed -i 's:MANAGER_IP:10.0.0.4:g' /var/ossec/etc/ossec.conf

3. Restart the agent.

  a. For Systemd:

    .. code-block:: console

      # systemctl restart wazuh-agent

  b. For SysV Init:

    .. code-block:: console

      # service wazuh-agent restart

Password authorization
----------------------
You can protect the manager from unauthorized registrations by using a password. Choose one by yourself, or let the registration service generate a random password.

To allow this option, change the value to *yes* in the ``/var/ossec/etc/ossec.conf`` file:

.. code-block:: xml

  <auth>
    ...
    <use_password>yes</use_password>
    ...
  </auth>

To apply the changes, restart the manager:

  a. For Systemd:

    .. code-block:: console

      # systemctl restart wazuh-manager

  b. For SysV Init:

    .. code-block:: console

      # service wazuh-manager restart

To use a custom password, edit the ``/var/ossec/etc/authd.pass`` file and write it. For example, if we want to use *TopSecret* as a password:

.. code-block:: console

  # echo "TopSecret" > /var/ossec/etc/authd.pass

Then, restart the manager.

.. _verify-hosts:

Host verification using SSL
---------------------------

.. note::
  Using verification with an SSL key certificate is really useful to check if connections between agents and managers are correct.

  This way, the user avoids the mistake of connecting to a different manager or agent.


Creating a Certificate of Authority (CA)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To use the registration service with SSL certification, you must create a Certificate of Authority that will be used to sign certificates for the manager and the agents. The hosts will receive a copy of this CA in order to verify the remote certificate:

.. code-block:: console

  # openssl req -x509 -new -nodes -newkey rsa:2048 -keyout rootCA.key -out rootCA.pem -batch -subj "/C=US/ST=CA/O=Manager"

.. warning::

  The file ``rootCA.key`` that we have just created is the **private key** of the CA. It is needed to sign other certificates and it is critical to keep it secure. Note that we will never copy this file to other hosts.

Manager verification using SSL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  .. image:: ../../../images/manual/managing-agents/SSLregister1.png
    :align: center
    :width: 100%

1. Issue and sign a certificate for the manager, entering the hostname or the IP address that agents will use to connect to the server. For example, if the manager's IP is **192.168.1.2**:

  .. code-block:: console

    # openssl req -new -nodes -newkey rsa:2048 -keyout sslmanager.key -out sslmanager.csr -subj '/C=US/CN=192.168.1.2'
    # openssl x509 -req -days 365 -in sslmanager.csr -CA rootCA.pem -CAkey rootCA.key -out sslmanager.cert -CAcreateserial

2. Copy the newly created certificate (and its key) to the ``/var/ossec/etc`` folder **on the manager**:

  .. code-block:: console

    # cp sslmanager.cert sslmanager.key /var/ossec/etc


3. Modify the ``ssl_manager_cert`` and ``ssl_manager_key`` parameters in ``ossec.conf`` **on the manager** to include these files:

  .. code-block:: xml

    <ssl_manager_cert>/var/ossec/etc/sslmanager.cert</ssl_manager_cert>
    <ssl_manager_key>/var/ossec/etc/sslmanager.key</ssl_manager_key>

4. Restart the manager:

  .. code-block:: console

    # systemctl restart wazuh-manager

5. Copy the CA (**but not the key**) to the ``/var/ossec/etc`` folder **on the agent**, and run the ``agent-auth`` program:

  a. For Linux systems:

    .. code-block:: console

      # cp rootCA.pem /var/ossec/etc
      # /var/ossec/bin/agent-auth -m 192.168.1.2 -v /var/ossec/etc/rootCA.pem

  b. For Windows systems, the CA must be copied to ``C:\Program Files (x86)\ossec-agent``:

    .. code-block:: console

      # cp rootCA.pem C:\Program Files (x86)\ossec-agent
      # C:\Program Files (x86)\ossec-agent\agent-auth.exe -m 192.168.1.2 -v C:\Program Files (x86)\ossec-agent\rootCA.pem

.. warning::
  The manager verification is only a check. Although the verification fails, the connection will be realized successfully and returning just a warning.

Agent verification using SSL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. image:: ../../../images/manual/managing-agents/SSLregister2.png
    :align: center
    :width: 100%

**Agent verification (without host validation)**

In this example, we are going to create a certificate for agents without specifying their hostname, so that the same certificate can be used by many of them. This verifies that agents have a certificate signed by our CA, no matter where they're connecting from.

1. Issue and sign a certificate for the agent. Note that we will not enter the *common name* field:

  .. code-block:: console

    # openssl req -new -nodes -newkey rsa:2048 -keyout sslagent.key -out sslagent.csr -batch
    # openssl x509 -req -days 365 -in sslagent.csr -CA rootCA.pem -CAkey rootCA.key -out sslagent.cert -CAcreateserial

2. Copy the CA (**but not the key**) to the ``/var/ossec/etc`` folder **on the manager** (if it's not already there):

  .. code-block:: console

    # cp rootCA.pem /var/ossec/etc

3. Modify the ``ossec.conf`` file **on the manager** to include the CA path and enable ``ssl_verify_host``:

  .. code-block:: xml

    <ssl_verify_host>yes</ssl_verify_host>
    <ssl_agent_ca>/var/ossec/etc/rootCA.pem</ssl_agent_ca>

4. Restart the manager:

  .. code-block:: console

    # systemctl restart wazuh-manager

5. Copy the newly created certificate (and its key) to the ``/var/ossec/etc`` folder **on the agent**, and run the ``agent-auth`` program. For example, if the manager's IP address is 192.168.1.2:

  a. For Linux systems:

    .. code-block:: console

      # cp sslagent.cert sslagent.key /var/ossec/etc
      # /var/ossec/bin/agent-auth -m 192.168.1.2 -x /var/ossec/etc/sslagent.cert -k /var/ossec/etc/sslagent.key

  b. For Windows systems, the CA must be copied to ``C:\Program Files (x86)\ossec-agent``:

    .. code-block:: console

        # cp sslagent.cert sslagent.key C:\Program Files (x86)\ossec-agent
        # C:\Program Files (x86)\ossec-agent\agent-auth.exe -m 192.168.1.2 -x C:\Program Files (x86)\ossec-agent\sslagent.cert -k C:\Program Files (x86)\ossec-agent\sslagent.key

**Agent verification (with host validation)**

This is an alternative method to the previous one. In this case, we will bind the agent's certificate to its IP address as seen by the manager.

1. Issue and sign a certificate for the agent, entering its hostname or IP address into the *common name* field. For example, if the agent's IP is 192.168.1.3:

  .. code-block:: console

    # openssl req -new -nodes -newkey rsa:2048 -keyout sslagent.key -out sslagent.csr -subj '/C=US/CN=192.168.1.3'
    # openssl x509 -req -days 365 -in sslagent.csr -CA rootCA.pem -CAkey rootCA.key -out sslagent.cert -CAcreateserial

2. Copy the CA (**but not the key**) to the ``/var/ossec/etc`` folder **on the manager** (if it's not already there):

  .. code-block:: console

    # cp rootCA.pem /var/ossec/etc

3. Modify the ``ossec.conf`` file **on the manager** to include the CA path and enable ``ssl_verify_host``:

  .. code-block:: xml

    <ssl_verify_host>yes</ssl_verify_host>
    <ssl_agent_ca>/var/ossec/etc/rootCA.pem</ssl_agent_ca>

4. Restart the manager:

  .. code-block:: console

    # systemctl restart wazuh-manager

5. Copy the newly created certificate (and its key) to the ``/var/ossec/etc`` folder **on the agent**, and run the ``agent-auth`` program. For example, if the manager's IP address is 192.168.1.2:

  a. For Linux systems:

    .. code-block:: console

      # cp sslagent.cert sslagent.key /var/ossec/etc
      # /var/ossec/bin/agent-auth -m 192.168.1.2 -x /var/ossec/etc/sslagent.cert -k /var/ossec/etc/sslagent.key

  b. For Windows systems, the CA must be copied to ``C:\Program Files (x86)\ossec-agent``:

    .. code-block:: console

        cp sslagent.cert sslagent.key C:\Program Files (x86)\ossec-agent
        C:\Program Files (x86)\ossec-agent\agent-auth.exe -m 192.168.1.2 -x C:\Program Files (x86)\ossec-agent\sslagent.cert -k C:\Program Files (x86)\ossec-agent\sslagent.key

Additional configurations
-------------------------

By default, the registration service adds the agents with their static IP address. If you want to add them with a dynamic IP (like using ``any`` on the ``manage_agents`` tool), you must change the manager's configuration file (``/var/ossec/etc/ossec.conf``):

.. code-block:: xml

  <auth>
    <use_source_ip>no</use_source_ip>
  </auth>

Duplicate IPs are not allowed, so an agent won't be added if there is already another agent registered with the same IP. By changing the configuration file, ``ossec-authd`` can be told to **force a registration** if it finds an older agent with the same IP address. This will make the older agent's registration be deleted:

.. code-block:: xml

  <auth>
    <force_insert>yes</force_insert>
    <force_time>0</force_time>
  </auth>

The **0** on ``<force-time>`` means the minimum time, in seconds, since the last connection of the old agent (the one to be deleted). In this case, it means to delete the old agent's registration regardless of how recently it has checked in.
