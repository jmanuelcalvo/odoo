Ejecucion de un Odoo en un ambiente de contenedores con RHEL + Podman

```
[root@localhost ~]# podman run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db postgres:13
✔ docker.io/library/postgres:13
Trying to pull docker.io/library/postgres:13...
Getting image source signatures
Copying blob 5e26c947960d done
Copying blob f1f26f570256 done
Copying blob 1c04f8741265 done
Copying blob dffc353b86eb done
Copying blob 18c4a9e6c414 done
Copying blob 81f47e7b3852 done
Copying blob a2c3dc85e8c3 done
Copying blob 17df73636f01 done
Copying blob 124bb42a3852 done
Copying blob dfb19482a052 done
Copying blob bbb12a596105 done
Copying blob fdbdb6eba8dc done
Copying blob aa8960c4e383 done
Copying config dd421ca1f7 done
Writing manifest to image destination
Storing signatures
ebd1a48d2125a15962ca04c5559ea4c4aa7968731f557e5421cb971174ab26a0
[root@localhost ~]#
```

```
[root@localhost ~]# mkdir /var/lib/clienteodoo01
[root@localhost ~]# setenforce 0
[root@localhost ~]# podman pod create -n clienteodoo01 -p 8069:8069
[root@localhost ~]# podman run -d --pod clienteodoo01 --name odoo -t odoo
[root@localhost ~]# podman run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db --volume /var/lib/clienteodoo01:/var/lib/postgresql/data:Z --pod clienteodoo01 postgres:13
[root@localhost ~]# podman  ps
CONTAINER ID  IMAGE                                    COMMAND     CREATED         STATUS             PORTS                   NAMES
05e222679247  localhost/podman-pause:4.2.0-1673617532              22 seconds ago  Up 11 seconds ago  0.0.0.0:8069->8069/tcp  de004550ee7c-infra
d535316181b0  docker.io/library/odoo:latest            odoo        11 seconds ago  Up 11 seconds ago  0.0.0.0:8069->8069/tcp  odoo
274dac653d01  docker.io/library/postgres:13            postgres    2 seconds ago   Up 3 seconds ago   0.0.0.0:8069->8069/tcp  db
[root@localhost ~]#

[root@localhost ~]# podman pod stop clienteodoo01
[root@localhost ~]# podman pod start clienteodoo01
```

Creacion de servicio de Odoo Cliente 01

```
[root@localhost ~]# vi /etc/systemd/system/clienteodoo01.service
[root@localhost ~]# cat /etc/systemd/system/clienteodoo01.service
[Unit]
Description=Cliente 01 Odoo container
Wants=syslog.service
[Service]
Restart=always
ExecStart=/usr/bin/podman pod start clienteodoo01
ExecStop=/usr/bin/podman pod stop -t 10 clienteodoo01
[Install]
WantedBy=multi-user.target
```


[root@localhost ~]# systemctl enable clienteodoo01.service
Created symlink /etc/systemd/system/multi-user.target.wants/clienteodoo01.service → /etc/systemd/system/clienteodoo01.service.
[root@localhost ~]# systemctl start clienteodoo01.service
