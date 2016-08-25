# VSFTPD
sample vsftpd config I used to configure a secure ftp service
umask is set to 002 so groups can read/write

````
anon_upload_enable=NO
anonymous_enable=NO
banner_file=/etc/issue
chroot_list_enable=NO
chroot_local_user=YES
connect_from_port_20=NO
dirlist_enable=NO
dirmessage_enable=NO
download_enable=YES
force_local_data_ssl=YES
force_local_logins_ssl=YES
idle_session_timeout=600
listen=YES
listen_port=8022
local_enable=YES
local_umask=002
max_clients=5
max_per_ip=1
pam_service_name=vsftpd
pasv_enable=YES
pasv_max_port=10090
pasv_min_port=10090
require_ssl_reuse=NO
rsa_cert_file=/etc/vsftpd/<your_hostname>.pem
setproctitle_enable=NO
ssl_enable=YES
ssl_sslv2=NO
ssl_sslv3=NO
ssl_tlsv1=YES
tcp_wrappers=YES
text_userdb_names=NO
userlist_deny=NO
userlist_enable=YES
userlist_file=/etc/vsftpd.user_list
write_enable=YES
xferlog_enable=YES
xferlog_file=/var/log/vsftpd.log
xferlog_std_format=YES
````
