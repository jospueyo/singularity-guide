# example of successful container creation and usage
A researcher needs a remote service running `postgresql` with `postgis`
installed, so he can access it and create databases, tables, install
extensions, and also let the cluster do all the computation.

This is the summarized list of steps that were needed to configure and run a
container in the ICRA's HPC cluster.

## Step 1: download the official postgis container image file and create a sandbox from it
```
$ sudo singularity build --sandbox pgcontainer docker://postgis/postgis
```
This command creates a folder called "pgcontainer",
but you can choose any name for it.

Reference for the `postgis` container:
https://registry.hub.docker.com/r/postgis/postgis/

## Step 2: enter the sandbox using --writable mode
```
$ sudo singularity shell --writable pgcontainer/
```
Now we are inside the sandbox, and we can modify things because of the `--writable` option.

## Step 3: configure postgresql files
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
Singularity> su postgres -c psql
```
Change the password and quit:
```
postgres=# \password postgres
Enter new password: <new-password>
postgres=# \q
```

## Step 5: exit the sandbox
Now everything is configured, so we can leave the sandbox.
```
Singularity> su postgres -c "pg_ctl stop"
Singularity> exit
```

## Step 6: create an image file from the sandbox
At this point we can compile the sandbox to a .sif file
(see "how to create a container" for more details).
```
$ sudo singularity build pgcontainer.sif pgcontainer/
```
Now we just need to send the `pgcontainer.sif` file to the cluster. There are a
lot of options to do this, but one I like is `magic-wormhole`
```
pip install magic-wormhole
```

## Step 7: run a new instance from the image file
In the cluster, we create an instance of the container, mapping the "internal"
port (5432) to the "external" (from the host) port 54320. You can choose
whatever external port you want, as long it is available:
```
$ singularity instance start --writable-tmpfs \
  --net --network-args "portmap=54320:5432/tcp" \
  pgcontainer/ pgcontainer
```
The instance will need to write to disk, so we use the
`--writable-tmpfs` argument to allocate some space in memory.

The --net and --network-args options allow the incoming traffic follow this path:
```
            +--------------------------------------------------------+
request --> | cluster_ip_address:54320 --> container_ip_address:5432 |
            +--------------------------------------------------------+
```

Finally, we enter the container instance and start the postgresql service:
```
$ singularity shell instance://pgcontainer
Singularity> su postgres -c "pg_ctl start"
```

## Step 8: final checks
We check the postgresql status inside the container:
```
Singularity> su postgres -c "pg_ctl status"
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
- Port: 54320 (or whatever you set in the step 6)
- Username: postgres
- Password: the one you set in step 4

