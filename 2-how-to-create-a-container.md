This guide is a summary of this one:
https://singularity-tutorial.github.io/03-building/ which is more in depth.

## 1. create a new container called "my_pipeline"
In your local computer, where you have admin privileges (root access), do:
```sh
$ sudo singularity build --sandbox my_pipeline docker://ubuntu
```
This is telling Singularity to build a new container called "my_pipeline" from
a new empty fresh ubuntu image.  The --sandbox option in the command above
tells Singularity that we want to build a special type of container (called a
sandbox) for development purposes. You will see that a new folder called
"my_pipeline" appeared in your current directory.

## 2. enter the container with the "shell" command and the "--writable" option
```sh
$ sudo singularity shell --writable my_pipeline
```
The --writable option enables the user to install things inside the container

## 3. install things inside the container
```sh
Singularity> apt update
Singularity> apt install r-base
Singularity> apt install python3
```
In this example we install R and python3.

## 4. build the .sif file
Exit the container, and compile the container to a new .sif file:
```sh
Singularity> exit
$ sudo singularity build my_pipeline.sif my_pipeline
```

## 5. test your build
Execute the installed programs from outside the container:
```sh
$ singularity exec my_pipeline.sif Rscript -e "1+1"
$ singularity exec my_pipeline.sif python3 -c "print(42)"
```
You can also shell to the container and execute the programs there:
```sh
$ singularity shell my_pipeline.sif
Singularity> python3 -c "print(42)"
Singularity> Rscript -e "1+1"
```
Note that the "sudo" command here is not needed (now we don't install
anything), we just execute programs installed inside the .sif file.

## 6. ready to go
If you are happy with your .sif file, just send it to the cluster and execute
it there using the same "exec" or "shell" commands. Note that in the cluster
you won't be able to modify it, since you won't have access to the "sudo"
command there.
