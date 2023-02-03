# singularity-guide
guide under development (2023-02-03)

## what is this guide?
Researchers can create their own environments (also called "pipelines",
"workflows", "software suites", "sandboxes", etc) locally in their own
computer, and later send this environment (which gets compiled to a single
file) to the cluster where it can be executed, with the following advantages:
- You don't need admin privileges (root access, sudo, etc)
- You don't need to ask the admin to install anything

## singularity workflow for a researcher (summary)
1. install singularity on your local computer
2. create a writable container (called a "sandbox")
3. shell ("enter") into the container with the --writable option and tinker
   with it interactively (install whatever you want)
4. record changes that you like in your definition file
5. rebuild the container from the definition file if you break it
6. rinse and repeat until you are happy with the result
7. rebuild the container from the final definition file as a read-only
   singularity image format (SIF) image for use in the cluster
8. send this .SIF file to the cluster, and execute it there using singularity
