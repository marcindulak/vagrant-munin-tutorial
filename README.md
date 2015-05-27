-----------
Description
-----------

Vagrantfile that installs and configures Munin server and nodes.
The server runs on CentOS6, nodes can run on Debian(Ubuntu) and CentOS(Fedora).

Tested on: Ubuntu 14.04, CentOS 6/7.

------------
Sample Usage
------------

Assuming you have VirtualBox and Vagrant installed
https://www.virtualbox.org/ https://www.vagrantup.com/downloads.html
test the module with::

        $ git clone https://github.com/marcindulak/vagrant-munin-tutorial.git
        $ cd vagrant-munin-tutorial
        $ vagrant up
        $ curl --user munin:munin http://localhost:8080/munin/
        $ firefox http://localhost:8080/munin/  # Note the final slash! user/pw: munin/munin

After about 5 minutes (Munin's time period),
you should see the following Munin setup:

![Munin](https://raw.github.com/marcindulak/vagrant-munin-tutorial/master/screenshots/munin.png)

Be patient, first only the contents of /var/www/html/munin/ is listed.
Watch /var/log/munin/munin-update.log for Munin's activity on the server
and /var/log/munin-node/munin-node.log on the nodes.

Let's configure monitoring of Apache running on the **centos6** machine::

        $ vagrant ssh centos6 -c "sudo su -c 'cd /etc/munin/plugins/; ln -s /usr/share/munin/plugins/apache_accesses'"
        $ vagrant ssh centos6 -c "sudo su -c 'cd /etc/munin/plugins/; ln -s /usr/share/munin/plugins/apache_processes'"
        $ vagrant ssh centos6 -c "sudo su -c 'cd /etc/munin/plugins/; ln -s /usr/share/munin/plugins/apache_volume'"
        $ vagrant ssh centos6 -c "sudo su -c 'echo \"[apache_*]\" > /etc/munin/plugin-conf.d/apache'"
        $ vagrant ssh centos6 -c "sudo su -c 'service munin-node reload'"

After another 5 minutes Apache plots will be available in Munin's web interface.
Note that Apache is not running on the **centos6** machine, as can be seen from::

        $ vagrant ssh centos6 -c "sudo su -c 'curl http://localhost/'"

and yet, Munin does not report this as problem. Let's start Apache::

        $ vagrant ssh centos6 -c "sudo su -c 'service httpd start'"

After another 5 minutes the "Apache processes" should start showing some data.

This brings up the summary point: Munin out-of-the-box is more a performance
monitoring system. It does not tell you if a service is down, only if
the performance metric does not fall outside of the given threshold.

When done, destroy the test machines with::

        $ vagrant destroy -f


------------
Dependencies
------------

None


-------
License
-------

BSD 2-clause


----
Todo
----

