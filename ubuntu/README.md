# Ubuntu Packer Templates for MAAS

## Introduction

The Packer templates in this directory creates Ubuntu images for use with MAAS.

## Prerequisites (to create the image)

* A machine running Ubuntu 18.04+ with the ability to run KVM virtual machines.
* qemu-utils, libnbd-bin, nbdkit and fuse2fs
* qemu-system
* ovmf
* cloud-image-utils
* [Packer](https://www.packer.io/intro/getting-started/install.html), v1.7.0 or newer

## Requirements (to deploy the image)

* [MAAS](https://maas.io) 3.0+
* [Curtin](https://launchpad.net/curtin) 21.0+

## ubuntu-flat.pkr.hcl and ubuntu-lvm.pkr.hcl

These templates use an Ubuntu server image to install the image to the VM. It
takes longer than using a cloud image, but can be useful for certain use cases.

### Customizing the Image

It is possible to customize the image either during the Ubuntu installation or afterwards, before packing the final image. The former is done by providing [autoinstall config](https://ubuntu.com/server/docs/install/autoinstall), editing the _user-data-flat_ and _user-data-lvm_ files. The latter is performed by the _install-custom-packages_ script.

### Building the image using a proxy

The Packer template downloads the Ubuntu net installer from the Internet. To tell Packer to use a proxy set the HTTP_PROXY environment variable to your proxy server. Alternatively you may redefine iso_url to a local file, set iso_checksum_type to none to disable checksuming, and remove iso_checksum_url.

### Building an image

You can easily build the image using the Makefile:

```shell
make custom-ubuntu-lvm.dd.gz
```

to build a raw image with LVM, alternatively, you can build a TGZ image

```shell
make custom-ubuntu.tar.gz
```

You can also manually run packer. Your current working directory must
be in packer-maas/ubuntu, where this file is located. Once in
packer-maas/ubuntu you can generate an image with:

```shell
packer init .
PACKER_LOG=1 packer build -only=qemu.lvm .
```

Note: ubuntu-lvm.pkr.hcl and ubuntu-flat.pkr.hcl are configured to run Packer in headless mode. Only Packer output will be seen. If you wish to see the installation output connect to the VNC port given in the Packer output or change the value of headless to false in the HCL2 file.

Installation is non-interactive.  Note that the installation will attempt an SSH connection to the QEMU VM where the newly-built image is being booted.  This is the final provisioning step in the process.  Packer uses SSH to discover that the image has, in fact, booted, so there may be a number of failed tries -- over 3-5 minutes -- until the connection is successful.  This is normal behavior for packer.

### Default Username

The default username is ```ubuntu```

## Uploading images to MAAS

TGZ image

```shell
maas admin boot-resources create \
    name='custom/ubuntu-tgz' \
    title='Ubuntu Custom TGZ' \
    architecture='amd64/generic' \
    filetype='tgz' \
    content@=custom-cloudimg.tar.gz
```

LVM raw image

```shell
maas admin boot-resources create \
    name='custom/ubuntu-raw' \
    title='Ubuntu Custom RAW' \
    architecture='amd64/generic' \
    filetype='ddgz' \
    content@=custom-ubuntu-lvm.dd.gz
```
