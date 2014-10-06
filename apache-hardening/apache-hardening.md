Apache Hardening
================

Introduction
============

This is a guide to hardening apache / web front end Linux hosts, primarily aimed at RHEL / Centos.


##1. Stopping them getting in##


###Install###

Systems should always have a minimal amount of packages installed - a webserver should not need XWindows, etc. The greater the install footprint, the more likely something is installed which can be exploited.

Standard Linux system hardening should be applied. (eg /tmp should be mounted nosuid,noexec). SELinux (if on RH / Centos) should be set in enforcing mode.

###Network / Firewall###

If possible place your webserver on an internal IP behind a firewall doing Static NAT in a DMZ. The webserver should not have direct access to the internet (if certain external URLs must be accessible this should be limited through firewall rules or ideally the webserver should be configured to use a locked down HTTP proxy on another host).

If your webserver is standalone without a separate firewall, you can configure iptables rules to [limit or block outgoing traffic from the webserver user](http://linuxpoison.blogspot.co.uk/2010/11/how-to-limit-network-access-by-user.html) - eg <code>iptables -A OUTPUT -o eth0 -m owner --uid-owner http -j ACCEPT</code>. This is in addition to permitting the minimum possible.

####Other vectors for attack####

Ssh can be hardened by: 

* Restricting source IP in iptables
* Moving it to a non-standard port (this will reduce automated attacks, but should not be relied upon and be used with other techniques).
* Using a tool such as denyhosts to block dictionary attacks (I recommend whitelisting trusted source address in /etc/hosts.deny here)
* Using certificate or 2-factor authentication only
* Limiting ssh users with Match, AllowGroups, AllowUsers configuration. sshd is [highly configurable](http://www.manpagez.com/man/5/sshd_config/).

###Patching###

Ensure your system is up to date with patches. Various update methods listed below:

* (Centos) Enable updates with [yum-cron](http://docs.fedoraproject.org/en-US/Fedora/14/html /Software_Management_Guide/ch07s07.html), [spacewalk](http://spacewalk.redhat.com/)
* (Debian / Ubuntu) Enable updates with unattended-upgrades [ubuntu](https://help.ubuntu.com/community/AutomaticSecurityUpdates) / [debian](https://wiki.debian.org/UnattendedUpgrades)
* Enable updates through [Puppet](http://puppetlabs.com/) / [Ansible](http://www.ansible.com/home) / similar system management tool

Updates should be fetched from an internal repo if possible - not directly from the internet (see setup

Additionally once servers are patched ensure the newer version of packages are running, this includes kernel packages. For normal daemons you can just restart, for the kernel unless you are using [ksplice](https://www.ksplice.com/) or similar you'll need to reboot.

One check you could do to see if old library versions are in use is "lsof -n | grep DEL | grep lib", however if in doubt a reboot is safest.

You can use [check_updates](http://exchange.nagios.org/directory/Plugins/Operating-Systems/Linux/check_updates/details) to verify no updates are pending on a Centos system (additionally this will check if you are running the latest kernel, or if you need to reboot).

