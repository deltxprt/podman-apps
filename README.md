# podman-apps
contains all apps i use for my personal services all in rootless mode

# Rootless Setup

Sources: 
- https://blog.christophersmart.com/2021/02/20/rootless-podman-containers-under-system-accounts-managed-and-enabled-at-boot-with-systemd/
- https://www.redhat.com/sysadmin/quadlet-podman
- https://www.redhat.com/sysadmin/multi-container-application-podman-quadlet

## User Setup



`export SERVICE="myservice"`

`sudo useradd -r -m -d "/var/lib/${SERVICE}" -s /bin/false "${SERVICE}"`

Automatically start on boot:
`sudo loginctl enable-linger "${SERVICE}"`

For volumes:
`sudo -H -u "${SERVICE}" bash -c "mkdir ~/data"`

For selinux system:

```sh
sudo semanage fcontext -a -t user_home_dir_t \
"/var/lib/${SERVICE}(/.+)?"

sudo semanage fcontext -a -t container_file_t \
"/var/lib/${SERVICE}/data(/.+)?"
 
sudo restorecon -Frv /var/lib/"${SERVICE}"
```

Enable rootless containers:

```sh
NEW_SUBUID=$(($(tail -1 /etc/subuid |awk -F ":" '{print $2}')+65536))
NEW_SUBGID=$(($(tail -1 /etc/subgid |awk -F ":" '{print $2}')+65536))
 
sudo usermod \
--add-subuids ${NEW_SUBUID}-$((${NEW_SUBUID}+65535)) \
--add-subgids ${NEW_SUBGID}-$((${NEW_SUBGID}+65535)) \
  "${SERVICE}"
```

Become the system user:

`sudo -H -u "${SERVICE}" bash -c 'cd; bash'`

Testing:

```sh
podman run -u 0:0 -dit --name busybox -v data:/data:z busybox
podman exec busybox sh -c 'echo -n "In this container, I am ";id -un'
```

** Important Note: everytime you impersonate the system user the user session is not created and therefore you can't interact with systemd properly **
  
To mitigate this you need to exeute this command each time you become the system user:

`export XDG_RUNTIME_DIR=/run/user/"$(id -u)"`

# Random tips

## can't bind port 80 or 443 as rootless?

ports under 1024 are reserved and therefore need privilege escalation to be able to use it

the fix:

`echo "net.ipv4.ip_unprivileged_port_start=80" | sudo tee -a /etc/sysctl.conf && sudo sysctl -p /etc/sysctl.conf`

## where to put my quadlets files?

For rootless containers you want to put them in `$HOME/.config/containers/systemd/`
