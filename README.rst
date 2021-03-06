openssh
=======
Install and configure an openssh server.

.. note::

    See the full `Salt Formulas installation and usage instructions
    <http://docs.saltstack.com/en/latest/topics/development/conventions/formulas.html>`_.

Available states
================

.. contents::
    :local:

``openssh``
-----------

Installs the ``openssh`` server package and service.

``openssh.auth``
-----------

Manages SSH certificates for users.

``openssh.auth_map``
-----------

Same functionality as openssh.auth but with a simplified Pillar syntax.
Plays nicely with `Pillarstack
<https://docs.saltstack.com/en/latest/ref/pillar/all/salt.pillar.stack.html>`_.

``openssh.banner``
------------------

Installs a banner that users see when SSH-ing in.

``openssh.client``
------------------

Installs the openssh client package.

``openssh.config``
------------------

Installs the ssh daemon configuration file included in this formula
(under "openssh/files"). This configuration file is populated
by values from pillar. ``pillar.example`` results in the generation
of the default ``sshd_config`` file on Debian Wheezy.

It is highly recommended ``PermitRootLogin`` is added to pillar
so root login will be disabled.

``openssh.config_ini``
----------------------

Version of managing ``sshd_config`` that uses the 
`ini_managed.option_present <https://docs.saltstack.com/en/latest/ref/states/all/salt.states.ini_manage.html>`_
state module, so it enables to override only one or 
multiple values and keeping the defaults shipped by your 
distribution. 


``openssh.known_hosts``
-----------------------

Manages the side-wide ssh_known_hosts file and fills it with the
public SSH host keys of your minions (collected via the Salt mine)
and of hosts listed in you pillar data. You can restrict the set of minions
whose keys are listed by using the pillar data ``openssh:known_hosts:target``
and ``openssh:known_hosts:tgt_type`` (those fields map directly to the
corresponding attributes of the ``mine.get`` function).

The Salt mine is used to share the public SSH host keys, you must thus
configure it accordingly on all hosts that must export their keys. Two
mine functions are required, one that exports the keys (one key per line,
as they are stored in ``/etc/ssh/ssh_host_*_key.pub``) and one that defines
the public hostname that the keys are associated to. Here's the way to
setup those functions through pillar::

    # Required for openssh.known_hosts
    mine_functions:
      public_ssh_host_keys:
        mine_function: cmd.run
        cmd: cat /etc/ssh/ssh_host_*_key.pub
        python_shell: True
      public_ssh_hostname:
        mine_function: grains.get
        key: id

The above example assumes that the minion identifier is a valid DNS name
that can be used to connect to the host. If that's not the case, you might
want to use the ``fqdn`` grain instead of the ``id`` one. The above example
also uses the default mine function names used by this formula. If you have to
use other names, then you should indicate the names to use in pillar keys
``openssh:known_hosts:mine_keys_function`` and
``openssh:known_hosts:mine_hostname_function``.

You can also integrate alternate DNS names of the various hosts in the
ssh_known_hosts files. You just have to list all the alternate DNS names as a
list in the ``openssh:known_hosts:aliases`` pillar key. Whenever the IPv4 or
IPv6 behind one of those DNS entries matches an IPv4 or IPv6 behind the
official hostname of a minion, the alternate DNS name will be associated to the
minion's public SSH host key.

To add public keys of hosts not among your minions list them under the
pillar key ``openssh:known_hosts:static``::

    openssh:
      known_hosts:
        static:
          github.com: 'ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq[...]'
          gitlab.com: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABA[...]'


``openssh.moduli``
-----------------------

Manages the system wide ``/etc/ssh/moduli`` file.
