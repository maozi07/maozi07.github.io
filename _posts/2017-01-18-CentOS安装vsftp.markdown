---
layout: post
---

### 本地用户/系统用户

```
useradd -g ftp -d /vsftp/test1 -s /sbin/nologin -M test1
useradd -g ftp -d /vsftp/test2 -s /sbin/nologin -M test2
useradd -g ftp -d /vsftp/test3 -s /sbin/nologin -M test3

mkdir -p /vsftp/test1 /vsftp/test2 /vsftp/test3;chown -R ftp:ftp /vsftp
```
- 配置文件样例

```
#user test1 config file sample in /etc/vsftp/vconf

local_root=/vsftp/test1
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES

vsftp.conf  sample
anonymous_enable=NO
local_enable=YES
chroot_local_user=YES
user_config_dir=/ect/vsftpd/vconf
```

- 上传文件的权限

用户以及组与ftp用户相同

### 虚拟用户



```
yum install db4-utils
db_load -T -t hash -f /etc/vsftpd/user.txt /etc/vsftpd/vftpuser.db
change /etc/pam.d/vsftp like this:
auth required /lib64/security/pam_userdb.so db=/etc/vsftpd/vftpuser
account required /lib64/security/pam_userdb.so db=/etc/vsftpd/vftpuser
```

- vsftp.conf sample

```
anonymous_enable=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
guest_enable=YES
guest_username=ftp
user_sub_token=$USER
local_root=/vsftp/$USER
hide_ids=YES
virtual_use_local_privs=YES
local_enable=YES
chroot_local_user=YES
write_enable=YES
local_umask=022
pam_service_name=vsftpd  #pam file path /etc/pam.d/vsftpd
```

- user config file sample not correct in centos5

```
#user test1 config file sample in /etc/vsftp/vconf

local_root=/vsftp/vs1
write_enable=YES
download_enable=YES
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
local_umask=022
```

- uploaded file mode

file user,group are the guset user's
