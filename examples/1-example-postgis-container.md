# example of successful container creation and usage
A user needed a remote service running:
- postgresql
- postgis

So he could access it and have the ability to create databases,
tables, install extensions, and also let the cluster do all the computation.

This is the summarized list of steps that were needed to configure and run a
container in the ICRA's HPC cluster.

## Step 1: download the official postgis container image file and create a sandbox from it
```
$ sudo singularity build --sandbox pgcontainer docker://postgis/postgis
```
"pgcontainer" was the name chosen in this case, but it can be any name.
This command creates the folder "pgcontainer", which is the sandbox itself.

## Step 2: enter the sandbox container using --writable mode
```
$ sudo singularity shell --writable pgcontainer/
```

## Step 3: configure postgresql files
```
Singularity> su postgres -c initdb         #populate the $PGDATA folder (default is /var/lib/postgresql/data)
Singularity> cd $PGDATA                    #go to the $PGDATA folder
Singularity> vim postgresql.conf           #edit file: enable the listen port to "5432"
Singularity> vim pg_hba.conf               #edit file: allow remote access from all hosts ("0.0.0.0/0")
Singularity> su postgres -c "pg_ctl start" #start postgresql service
```

## Step 4: setup postgres' user password
log in into the psql console:
```
Singularity> su postgres -c psql
```
In the psql console, change the password and quit:
```
postgres=# \password postgres
Enter new password: <new-password>
postgres=# \q
```

## Step 5: exit the sandbox container
Now everything is configured, so we can leave the sandbox container.
```
Singularity> su postgres -c "pg_ctl stop"
Singularity> exit
```

## Step 6: run the container
At this point we can compile the sandbox to a .sif file and send it to the cluster (see "how to create a container" for instructions).

In the cluster, we create an instance of the container, mapping the "internal" port (5432) to the "external" port (from the host) 54320:
```
$ singularity instance start --writable-tmpfs \
  --net --network-args "portmap=54320:5432/tcp" \
  pgcontainer/ pgcontainer
```
The instance will need to write to disk, so we’ve used the
--writable-tmpfs argument to allocate some space in memory.

The --net and --network-args options allow the incoming traffic follow this path:

```
request --> [cluster_ip_address:54320  -->  container_ip_address:5432]
```

Now we (1) enter the container and (2) start the postgresql service:
```
$ singularity shell instance://pgcontainer
Singularity> su postgres -c "pg_ctl start"
```

## Step 7: final checks
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

## Step 8: We can connect remotely to the container using psql or pgAdmin in our local machine
If everything worked, now we should be able to connect to the container from the outside:

- Host name/address: the remote IP of your cluster
- Port: 54320 (or whatever you set in the step 6)
- Username: postgres
- Password: the one you set in step 4

