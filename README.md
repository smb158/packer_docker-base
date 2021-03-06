## Overview

This repository contains configurations for using Packer to build machine images that define
a base Docker image, including AMIs (for AWS) and VirtualBox-based Vagrant boxes.

Boxes provisioned with these images will have the Docker engine running and ready to go as well
 as some base utilities and configurations that we find useful in all our environments.

This project attempts to create *equivalent* images across providers by starting from *similar*
base images and applying the *same* provisioning steps using Ansible. This approach allows us to
switch between AWS and Vagrant with little overhead and push common elements of Ansible-based
deploys into the initial image to save time during deployment.

## Source AMI

We are using Ubuntu Trusty 14.04 cloud images as our source AMI. To update to a newer
version go to https://cloud-images.ubuntu.com/trusty/current/ and find
the 64-bit image for the region you need (e.g. us-west-2). For EBS builds pick the `hvm-ssd`
image, for instance-store builds use the `instance` image.

## Release Process

Once your changes are merged to master you need to get the definitions built by [Atlas][].
The process differs a bit for the EBS/Virtualbox and instance-store definitions:

### EBS/Virtualbox definitions
You're done. Atlas will trigger a build automatically when a push to master occurs.

### Instance-store definitions
Here Atlas needs some secure information, namely a X.509 key/cert that can only be passed by file,
so for this to work you need to get access to those, place them in the root of this repo as
`key.pem` and `cert.pem` (which are gitignored), and run

```
export ATLAS_TOKEN=<your Atlas token>
packer push ec2-instance-us-west-2.json
```

## Artifacts

Once Atlas is done building you can find the latest images here:

* [AWS AMIs][]
* [Virtualbox Boxes][]

For an AMI, you can find it's id by looking at the metadata in Atlas.

## Subtrees
This repo is using git subtrees to pull in dependencies. Why not submodules?
Well, they're horrible for one. Subtrees are much simpler, you don't need
to run any extra commands, just clone as usual and everything is there.

### chef/bento subtree
The vagrant image is heavily based on [Chef]'s excellent work on their [bento][]
packer definitions. So much so that we use their scripts directly from our definitions.

To update the `bento` subtree:
```
git subtree pull --prefix bento https://github.com/chef/bento.git 2.2.2 --squash
```

Only use [bento release tags][] when updating `bento`.

[bento]: https://github.com/chef/bento
[bento release tags]: https://github.com/chef/bento/tags
[Chef]: http://chef.io
[Atlas]: https://atlas.hashicorp.com
[Virtualbox Boxes]: https://atlas.hashicorp.com/llabs/boxes/docker-base
[AWS AMIs]: https://atlas.hashicorp.com/llabs/artifacts/docker-base/types/amazon.ami
