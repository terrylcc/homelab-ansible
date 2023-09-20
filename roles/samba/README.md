# Samba module

After creating samba user, update the samba password

```shell
$ smbpasswd -a <samba_user>
```

For each user home directory, change the ownership respectively

```shell
$ chown -R <samba_user>:<samba_user> /srv/samba/home/<samba_user>
```

