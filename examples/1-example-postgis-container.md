# example of successful container creation and usage
A researcher needs a remote service running `postgresql` database with
`postgis`.  He wants to create databases, tables, install extensions, etc.

This guide summarizes the steps needed to configure and run a container in the
ICRA's HPC cluster. It could be useful for cluster admins.

## Step 1: download the official postgis container image file and create a sandbox from it
```
$ sudo singularity build --sandbox pgcontainer docker://postgis/postgis
```
This command creates a new folder called "pgcontainer", but any name will work.

Reference for the `postgis` official container:
https://registry.hub.docker.com/r/postgis/postgis/

## Step 2: configure your sandbox
Enter the sandbox:
```
$ sudo singularity shell --writable pgcontainer/
```
Now we are inside the sandbox, and we can modify things because of the
`--writable` option.

## Step 3: configure postgresql files
The following commands are the typical used to configure `postgresql`:
```
Singularity> su postgres -c initdb         #populate the $PGDATA folder (default is /var/lib/postgresql/data)
Singularity> cd $PGDATA                    #go to the $PGDATA folder
Singularity> vim postgresql.conf           #edit file: set the listening port to "5432"
Singularity> vim pg_hba.conf               #edit file: allow remote access from all hosts (add the line "host all all 0.0.0.0/0 md5")
Singularity> su postgres -c "pg_ctl start" #start postgresql service
```

## Step 4: setup access password
Log in into the `psql` console:
```
Singularity> su postgres
Singularity> psql
```
Create a new password and quit:
```
postgres=# \password postgres
Enter new password: <new-password>
postgres=# \q
```

## Step 5: exit the sandbox
Now everything is configured, so we can leave the sandbox.
```
Singularity> pg_ctl stop
Singularity> exit #logout from user "postgres"
Singularity> exit #logout from user "root"
```

## Step 6: create an image file from the sandbox
At this point we can compile the sandbox folder to a .sif file, which is
readonly (see "how to create a container" for more details).
```
$ sudo singularity build pgcontainer.sif pgcontainer/
```
Now we just need to send the `pgcontainer.sif` file to the cluster. There are a
lot of options to do this: `ftp`, `scp`, `magic-wormhole`...

## Step 7: run a new instance from the image file
The container instance will need to write to disk. Since the .sif file is
read-only, we need to create a folder that will be used to store new data. This
folder is known as an "overlay".  A persistent overlay is a directory that
“sits on top” of your compressed, immutable SIF container. When you install new
software or create and modify files the overlay directory stores the changes.
```
$ mkdir pgcontainer_overlay
```

In the cluster, as a root user, we create an instance of the container.
```
$ singularity instance start \
  --overlay pgcontainer_overlay \
  --net --network-args "portmap=54320:5432/tcp" \
  pgcontainer.sif pgcontainer
```
The `--overlay` option uses the folder `pgcontainer_overlay` that we've just
created.

The `--net` and `--network-args` options allow the incoming traffic in the port
54320 reach the container in the port 5432.

Finally, we enter the container instance and start the postgresql service:
```
$ singularity shell instance://pgcontainer
Singularity> su postgres
Singularity> pg_ctl start
```

## Step 8: final checks
We check the postgresql status inside the container:
```
Singularity> pg_ctl status
```
the output should be something like:
```
pg_ctl: server is running (PID: 30)
/usr/lib/postgresql/15/bin/postgres
```
Now we check if port 5432 is listening inside the container:
```
Singularity> netstat -na
```
the output should be something like:
```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN
tcp6       0      0 :::5432                 :::*                    LISTEN
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     34074696 /var/run/postgresql/.s.PGSQL.5432
```

## Step 9: access the database
If everything worked fine, now we should be able to access the database in the
container using a client (e.g. psql or pgAdmin).

- Host name/address: the remote IP of your cluster
- Port: 54320 (or whatever you set in step 6)
- Username: postgres
- Password: the one you set in step 4

