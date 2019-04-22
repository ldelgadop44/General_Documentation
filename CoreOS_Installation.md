# CoreOS Installation
This guide show the way to install CoreOS on virtualbox.

## About CoreOS
CoreOS Container Linux is the leading container operating system, designed to be managed and run at massive scale, with minimal operational overhead.
Applications with Container Linux run in containers, providing developer-friendly tools for deploying software.

Container Linux runs on nearly any platform whether physical, virtual, or private/public cloud.

## Before to begin
**1.** We have to create 3 virtual machines in virtualbox. It is recommended the next resources:

* vCPU: 1
* RAM: 2048 GB
* HDD: 30 GB

**2.** Download the ISO image from official web site https://coreos.com/os/docs/latest/booting-with-iso.html

**3.** Provide network connection to virtual machines, it is recommended to use the option "Adapter bridge" virtualbox

## Steps to installation
**1.** Place the ISO image on the main disk of the desired machines

**2.** When machines is up, execute the command ```sudo openssl passwd -1 > cloud-config-file``` Then, type and retype the desired password

**3.** Edit cloud-config-file with the command ```sudo vim cloud-config-file``` and type the next lines

```
#cloud-config
    users:
      - name: username
        passwd: [password]
        groups:
            - sudo
            - docker
```

**4.** Apply configuration, and install the operative system with the command

```sudo coreos-install -d /dev/sda -C stable -c cloud-config-file```

**5.** Finally, reboot virtual Machine

---
### Note:
When virtual machine is up, after reboot, verify internet connection and DNS configuration
