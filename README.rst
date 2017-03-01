How to deploy OpenStack Glare with ansible
################################################

Assume we have two nodes: command center (e.g. your laptop) and target. Target
ip address is *192.168.10.10*.

SSH
===

Ansible using ssh, so target node should be accessible via ssh without password. Generate ssh keypair on command node if necessary::

    $ ssh-keygen

*Good news: ssh-agent is supported by ansible*

Put your public key to the target if necessary::

    $ ssh-copy-id user@192.168.10.10

*use proper address of target*

The target
==========

It is important to be able to ssh and sudo on target node without significant
delays. Check your DNS settings and /etc/hosts if necessary. Use 'manage_etc_hosts'
cloud-init setting if target is VM in OpenStack cloud, or check /etc/hosts manually.
This file should contain own hostname::

    127.0.0.1 localhost
    127.0.1.1 target-hostname.example.net

*second line is important*

Also python should be installed on target::

    $ sudo apt-get install python

The command center
==================

Install ansible::

    $ sudo apt-get install ansible

*Replace apt-get with yum/dnf/emerge etc if necessary*

Edit /etc/ansible/hosts::

    [target]
    192.168.10.10 # here shold be address of target node

Ensure target is visible for ansible::

    $ ansible -m ping target -u ubuntu

Output should look like this::

    192.168.10.10 | SUCCESS => {
        "changed": false, 
        "ping": "pong"
    }

Assume we want to keep all ansible files in /store/ansible.

Clone this repository::

    $ mkdir -p /store/ansible/
    $ cd /store/ansible/
    $ git clone https://github.com/Fedosin/ansible-glare.git

Next we should tell ansible roles search path. Edit /etc/ansible/ansible.cfg and change one line in [defaults] section::

    [defaults]
    roles_path = /etc/ansible/roles:/store/ansible/ansible-glare/roles

Now copy/edit examples/full.yml::

    ---
    - hosts: target
      remote_user: ubuntu
      become: yes
      vars:
        glare_pip: git+git://git.openstack.org/openstack/glare.git#egg=glare
        glare_nginx_port: 9494
        glare_user: www-data
        glare_deploy_with: uwsgi
        glare_uwsgi_socket: /tmp/glare.sock
        glare_db_location: /tmp/glare.sqlite
        glare_enabled_artifact_types:
        glare_custom_artifact_types_modules:
        glare_protocol: http
        glare_domain: 192.168.10.10:9494
        glare_flavor:
        glare_keycloak_auth_url:
        init: systemd
      roles:
        - nginx
        - glare

NOTE: use "init: upstart" for upstart based systems

And finally deploy everything::

    $ ansible-playbook examples/full.yml

After that OpenStack Glare will be available by url: http://192.168.10.10
