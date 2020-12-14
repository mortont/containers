# LXD (GPU) Build Scripts

## About
This is a collection of Packer templates to build LXD containers that I've found
useful for reproducable (and shippable) machine learning develepmont environments.
Develop on your local machine with your local GPU and then export the container
to a remote server for the final run(s) with no configuration change.
## Building

From any of the directories you can run
```
packer build template.json
```
And should get an LXD image of the corresponding environment.

_Note:_ any `*-cuda` requires you to build the corresponding parent of the same
name first.

## Running
After building the image(s) desired, launch with:
```
lxc launch <image> <name>
```
And if it's a GPU image, find the PCI id of your device and pass it through to the image:
```
lspci -nnn | grep -i nvidia
57:00.0 3D controller [0302]: NVIDIA Corporation TU117M [GeForce GTX 1650 Mobile / Max-Q] [10de:1f91] (rev ff)
lxc config device add <name> gpu gpu pci="0000:57:00.0"
```
## Usage
I typically ssh into the instance:
```
lxc list <name>
+---------+---------+---------------------+
|  NAME   |  STATE  |        IPV4         |
+---------+---------+---------------------+
| <name>  | RUNNING | 10.105.66.88 (eth0) |
+---------+---------+---------------------+
ssh 10.105.66.88
```
From there, you can treat it as any development environment. I've installed all my
dotfiles using `chezmoi` which I would highly recommend.

## Exporting
To move a container to another machine, you can either export it as a `.tar.gz`
```
lxc stop CONTAINER_NAME
lxc snapshot CONTAINER_NAME SNAPSHOT_NAME
lxc publish CONTAINER_NAME/SNAPSHOT_NAME --alias my-export
lxc image export my-export .

[on the remote machine]
lxc image import TARBALL --alias my-export
lxc init my-export NEW-CONTAINER
```
or use LXD peering and transfer over the network:
```
lxc stop CONTAINER_NAME
lxc snapshot CONTAINER_NAME SNAPSHOT_NAME
lxc copy CONTAINER_NAME/SNAPSHOT_NAME target:CONTAINER_NAME"
```
_Note: it's best to stop the container before running the snapshot_
