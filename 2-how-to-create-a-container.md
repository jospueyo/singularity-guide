## 1. create a new container called "my_pipeline"
```sh
$ sudo singularity build --sandbox my_pipeline docker://ubuntu
```
This is telling Singularity to build a container called "my_pipeline" from a
new empty fresh ubuntu image.  The --sandbox option in the command above tells
Singularity that we want to build a special type of container (called a
sandbox) for development purposes.

## 2. enter the container
```sh
$ sudo singularity shell --writable my_pipeline
```
The --writable option makes it so we enable to install things inside the container

## 3. install things inside the container
```sh
Singularity> apt update
Singularity> apt install r-base
```
In this example we install R

## 4. building the .sif file for the container
```sh
Singularity> exit
$ sudo singularity build my_pipeline.sif my_pipeline
```

##5. test your build
```sh
# execute the program "R" inside the container
$ singularity exec my_pipeline.sif R
```

##6. ready
If you are happy with your .sif environment, just send it to the cluster and
execute it there using the same "exec" command. Note that in the cluster you
won't be able to modify it
