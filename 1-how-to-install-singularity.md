# install singularity in ubuntu22
The following steps come from the official install guide: https://github.com/sylabs/singularity/blob/main/INSTALL.md

## 1. ensure repositories are up-to-date and install packages for dependencies:
```sh
sudo apt update;
sudo apt install -y \
  build-essential \
  libseccomp-dev \
  libglib2.0-dev \
  pkg-config \
  squashfs-tools \
  cryptsetup \
  crun \
  uidmap \
  golang
```

## 2. download singularity:
```sh
git clone --recurse-submodules https://github.com/sylabs/singularity.git
cd singularity
git checkout --recurse-submodules v3.11.0
```

## 3. compile singularity:
```sh
./mconfig
make -C builddir
sudo make -C builddir install
```

## 4. check version by running:
```sh
singularity --version
```
