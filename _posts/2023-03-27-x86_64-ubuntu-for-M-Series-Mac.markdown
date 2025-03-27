---
layout: post
title:  "Running x86_64 Ubuntu on Apple Silicon (M-Series) MacBooks"
date:   2023-03-27 09:00:00 +0900
categories:
---

# Running x86_64 Ubuntu on Apple Silicon (M-Series) MacBooks

Pwners who are using ARM based Macbooks have two ways to run ubuntu which runs on x86_64 (Amd64) architecture.
1. Rosetta
2. Other emulation

In the case of `Rosetta`, for example, Docker uses it to run images on x86_64. However, due to the characteristics of Docker, this may not be a perfect way to play pwnable. You can easily find it out by running `cat /proc/self/maps` in Docker.

Therefore, I decided to the second choice, which is emulation. Almost emulation uses QEMU to emulate various architecture and I chose one too.
Most people use UTM, which is also based on QEMU, but it is so expensive not only in hardware-cost but in software-cost and so slow for pwners who are supposed to run various assorted ubutu versions.
Because of this issue, I finally chose vagrant which I used to run on x86_64 before Apple Silicon.Vagrant supports QEMU virtualization also users can run images which other users previously deployed, like Docker.

## Installment

1. Run `brew install vagrant`
2. Run `vagrant plugin install vagrant-vbguest\nvagrant plugin install vagrant-qemu`

## Create a new OS & Config shared folder

1. Run `mkdir [dirname] ; cd [dirname]`
2. Find and copy image box name in [https://portal.cloud.hashicorp.com/vagrant/discover](https://portal.cloud.hashicorp.com/vagrant/discover)
    - For example, ubuntu 24.04 is `generic/ubuntu2404"`
3. Copy and modify the below script and save as `Vagrantfile` into the `[dirname]`
```bash
BOX_IMAGE = "[BOX NAME]" # TODO, Boxname
HOST_NAME = "[HOST NAME]" # TODO, HOSTNAME

Vagrant.configure("2") do |config|

 config.vm.define HOST_NAME do |subconfig|
   subconfig.vm.box = BOX_IMAGE
   subconfig.vm.hostname = HOST_NAME
   subconfig.vm.provider "qemu" do |qe|
     qe.extra_qemu_args = %w(-virtfs local,path=[HOST PATH],mount_tag=shared,security_model=none) #TODO, FILL HOST PATH
     qe.arch = "x86_64"
     qe.machine = "q35"
     qe.cpu = "max"
     qe.net_device = "virtio-net-pci"
   end
 end
end
```

4. Run `vagrant up`
    - The boot takes some time. Be patient even if a number of `ubuntu2204: Warning: Remote connection disconnect. Retrying...` are emitted.
5. In **VM**, run `echo 'sudo mkdir /mnt/shared 2>/dev/null;sudo mount -t 9p -o trans=virtio shared /mnt/shared 2>/dev/null'  >> ~/.bashrc; source ~/.bashrc`.
6. In **VM**, Now, the `/mnt/shared` links to [HOST PATH] which is written in **[3]** step.
