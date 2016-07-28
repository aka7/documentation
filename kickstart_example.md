## Kickstart config template used in katello
````
<%#
kind: provision
name: Kickstart - BareMetal
oses:
- CentOS 5
- CentOS 6
- CentOS 7
- RedHat 5
- RedHat 6
- RedHat 7
- Fedora 19
- Fedora 20
%>
<%
  rhel_compatible = @host.operatingsystem.family == 'Redhat' && @host.operatingsystem.name != 'Fedora'
  os_major = @host.operatingsystem.major.to_i
  # safemode renderer does not support unary negation
  pm_set = @host.puppetmaster.empty? ? false : true
  puppet_enabled = pm_set || @host.params['force-puppet']
  salt_enabled = @host.params['salt_master'] ? true : false
  section_end = (rhel_compatible && os_major <= 5) ? '' : '%end'
%>
install
<%= @mediapath %>
# System language
lang en_GB.UTF-8
# Use text mode install
text
# Installation logging level
logging --level=info
# SE Linux Enable 
selinux --enforcing
# System keyboard
keyboard uk
# Do not configure the X Window System
skipx
# Network information
#network --bootproto <%= @static ? "static --ip=#{@host.ip} --netmask=#{@host.subnet.mask} --gateway=#{@host.subnet.gateway} --nameserver=#{[@host.subnet.dns_primary,@host.subnet.dns_secondary].reject{|n| n.blank?}.join(',')}" : 'dhcp' %> --hostname <%= @host %>
network --device=<%= @host.provision_interface.identifier %> --bootproto=<%= "static --ip=#{@host.provision_interface.ip} --netmask=#{@host.provision_interface.subnet.mask} --gateway=#{@host.provision_interface.subnet.gateway} --nameserver=#{[@host.subnet.dns_primary,@host.subnet.dns_secondary].reject{|n| n.blank?}.join(',')}" %> --hostname <%= @host %>
rootpw --iscrypted <%= root_pass %>
# Firewall configuration
firewall --disabled
# Services to enable/disable
services --disabled=iscsi,iscsid,iptables,ip6tables,sendmail --enabled=sssd,ntpd,rhnsd,snmpd,mcstrans,multipathd,messagebus
# System authorization information
auth  --useshadow  --passalgo=sha512 --enableldap --enableldapauth --ldapserver=ldap://<ldap_server_address> --ldapbasedn=uid=nssuser,ou=LdapAdmins,dc=test,dc=example,dc=com --enableldaptls --enablesssd --enablesssdauth --enablecache
# System timezone
timezone  Europe/London

<% if os_major == 5 -%>
key --skip
<% end -%>

<% if @dynamic -%>
%include /tmp/diskpart.cfg
<% else -%>
<%= @host.diskLayout %>
<% end -%>

# Reboot after installation
reboot

# Package selection
%packages
@large-systems
@british-support
sssd
bridge-utils
lldpad
libsss_sudo
nfs-utils
autofs
net-snmp
net-snmp-utils
device-mapper-multipath
setroubleshoot
mcstrans
sendmail
telnet
dos2unix
wget

-hypervkvpd
-acpid
-smartmontools
<%= section_end -%>

<% if @dynamic -%>
%pre
<%= @host.diskLayout %>
<%= section_end -%>
<% end -%>

%post --nochroot
exec < /dev/tty3 > /dev/tty3
#changing to VT 3 so that we can see whats going on....
/usr/bin/chvt 3
(
cp -va /etc/resolv.conf /mnt/sysimage/etc/resolv.conf
/usr/bin/chvt 1
) 2>&1 | tee /mnt/sysimage/root/install.postnochroot.log
<%= section_end -%>


%post
logger "Starting anaconda <%= @host %> postinstall"
exec < /dev/tty3 > /dev/tty3
#changing to VT 3 so that we can see whats going on....
/usr/bin/chvt 3
(

#update local time
echo "updating system time"
/usr/sbin/ntpdate -sub <%= @host.params['ntp-server'] || '0.fedora.pool.ntp.org' %>
/usr/sbin/hwclock --systohc

<%= snippet "kickstart_networking_setup" %>

<%= snippet "Subscription_Manager_Registration" %>

<% if @host.respond_to?(:realm) && @host.otp && @host.realm && @host.realm.realm_type == "FreeIPA" -%>
<%= snippet "freeipa_register" %>
<% end -%>

# update all the base packages from the updates repository
yum -t -y -e 0 update

<% if salt_enabled %>
yum -t -y -e 0 install salt-minion
cat > /etc/salt/minion << EOF
<%= snippet 'saltstack_minion' %>
EOF
# Setup salt-minion to run on system reboot
/sbin/chkconfig --level 345 salt-minion on
# Running salt-call to trigger key signing
salt-call --no-color --grains >/dev/null
<% end -%>

<% if puppet_enabled %>
# and add the puppet package
yum -t -y -e 0 install puppet

echo "Configuring puppet"
cat > /etc/puppet/puppet.conf << EOF
<%= snippet 'puppet.conf' %>
EOF

# Setup puppet to run on system reboot
/sbin/chkconfig --level 345 puppet on

/usr/bin/puppet agent --config /etc/puppet/puppet.conf -o --tags no_such_tag <%= @host.puppetmaster.blank? ? '' : "--server #{@host.puppetmaster}" %> --no-daemonize
<% end -%>

echo "Loading snipet Settingsi, if defined"
<% if @host.params['snippetName'] && @host.params['snippetName'] != '' %>
echo "snippetName set, adding in <%= @host.params['snippetName'] %>"
<%= snippet @host.params['snippetName'] %>
<% end %>

puppet agent -t 

sync

<% if @provisioning_type == nil || @provisioning_type == 'host' -%>
# Inform the build system that we are done.
echo "Informing Foreman that we are built"
wget -q -O /dev/null --no-check-certificate <%= foreman_url %>
<% end -%>
) 2>&1 | tee /root/install.post.log
exit 0

<%= section_end -%>
````
