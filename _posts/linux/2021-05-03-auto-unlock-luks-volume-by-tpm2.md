---
title: Auto-unlock LUKS root volume by TPM2
categories: linux ubuntu luks tpm encryption
---

It is so common to have full disk encryption to protect our data especially when we dispose our old hard drives. However, our PC/server will prompt for passphrase every boot after we enabled the full disk encryption on root volume. It's a bit annoying. If your PC/server got a TPM (Trusted Platform Module) chip, you can get rid of it by saving the encryption key inside TPM (Please noted that this action may let someone access your data if he can get the encryption key from your TPM directly. e.g. boot to your system physically and then send command to TPM for getting the key).

This article is talking about how to auto-unlock `LUKS` root volume by `TPM2` in `Ubuntu Server 20.04 LTS` (Please noted that Ubuntu Core 20 [for embedded] stated that it support TPM to unlock encrypted volume natively).

### Pre-requisite - Encrypt your disk

Of course we need to have an encrypted disk for unlocking but how LUKS work will not be discussed here and LUKS support multiple passphrases / keys for the encryption. i.e. You can have both passphrase and a random key to protect a disk such that either one can unlock the data. For more information, please refer this article for explaining [how LUKS works](https://gist.github.com/joonas-fi/b6bfc564a1a7f3d6b120728db5d48b61).

### Install tpm2_tools and clear (reset)

To interact with your TPM chip, we have to install `tpm2-tools` command line tools.

``` bash
#Update the apt repository first
apt update

#Install the tpm2-tools
apt-get install tpm2-tools
```

After that, we would like to perform a reset on TPM chip before use (Please noted that some TPM chips may required to have reset on BIOS, instead of command).

``` bash
#Try to remove "disable tpm clear" state on owner hierarchy (o: owner), c: clear, s: set disable
tpm2_clearcontrol -V -C o c

#TPM clear for owner hierarchy
tpm2_clear -V -c o
```

### Generate random key and then put it in TPM

Supposed that we already have primary passphrase / key to unlock the disk volume (we did it when we encrypt the disk) and now we are trying to add one more random key (512 bits) for our system to auto-unlock it during boot.

``` bash
#We use TPM to generate a random key instead of /dev/random
#To generate 32 bytes random data x 2 and then combine to 512 bits (because max size is 48 bytes one time)
tpm2_getrandom -V -o random1.bin 32
tpm2_getrandom -V -o random2.bin 32

#Combin two 32 bytes files to form a 54 bytes file
cat random1.bin random2.bin > random.bin

#Can remove the temp file after generated
shred random1.bin
shred random2.bin
rm random1.bin
rm random2.bin
```

### Seal the random key into TPM

For sealing the data into TPM, we have to generate a root key and a seal key. The steps are as follows:

``` bash
#Generate root key for owner (default: rsa + 'fixedtpm|fixedparent|sensitivedataorigin|userwithauth|restricted|decrypt')
tpm2_createprimary -V -C o -c primary.ctx

#Generate a key to seal data blob of 'random.bin' under root key "primary" (Seal data must be EDHASH as algo)
tpm2_create -V -g sha256 -u seal.pub -r seal.priv -i random.bin -C primary.ctx

#Load the seal key (public and private) part into TPM under under root key "primary" and then given 'seal.ctx' context
tpm2_load -V -C primary.ctx -u seal.pub -r seal.priv -c seal.ctx

#Persist the seal key into TPM (it will return the address)
tpm2_evictcontrol -V -C o -c seal.ctx
#Say return 0x81000000 as the address

#Unseal the data blob by persistent address and output as 'datablob.dat'
tpm2_unseal -V -c 0x81000000 -o datablob.dat

#Compare the output data with 'random.bin' to ensure if the data is matched
diff datablob.dat random.bin

#If both 'datablob.dat' and 'random.bin' are matched, then we can remove all generated files as it already persisted inside TPM
#We keep 'datablob.dat' for later use
shred random.bin
rm random.bin
rm primary.ctx
rm seal.*
```

### Add the random key for LUKS volume

As the random key is ready and already loaded into TPM, we can add this key to LUKS disk for unlock volume usage.

``` bash
#Suppose the device is /dev/sda and add 'datablob.dat' to LUKS disk
cryptsetup -v luksAddKey /dev/sda datablob.dat
#It will prompt for passphrase to unlock volume first

#After that, we can remove 'datablob.dat'
shred datablob.dat
rm datablob.dat
```

### Unlock LUKS volume during boot

Supposed that our system already has cryptsetup related libraries inside 'initramfs' during boot, otherwise, it cannot recognize and unlock LUKS volume during boot (Ubuntu server 20.04 LTS already did it). Therefore, we just need to make the system can run `tpm2_unseal` for getting the key during boot is fine. 

Firstly, as we will make changes on 'initramfs' image, so it is better to backup the original image in case we got some mistake and cause system cannot boot up.

``` bash
#Assumed that our boot mount is on /boot and its 'initramfs' image is 'initrd.img-5.4.0-72-generic', you can check it in grub
#Just copy it as backup
cp /boot/initrd.img-5.4.0-72-generic /boot/initrd.img-5.4.0-72-generic.original.bak
```

Check where is the program of `tpm2_unseal` located and where is its dependency library.

``` bash
which tpm2_unseal
#It will return the full path of tpm2_unseal and for example, /usr/bin/tpm2_unseal

#Check its shared libraries location
ldd /usr/bin/tpm2_unseal
#It will return the path of tpm2_unseal share libraries for example, /usr/lib/libtss2-xxxxxxxx
```

Make a TPM hook script `/etc/initramfs-tools/hooks/tpm2` to include `tpm2_unseal` for building 'initramfs'

``` bash
#!/bin/sh -e

PREREQS=""

prereqs() { echo "$PREREQS"; }

case "$1" in
    prereqs)
    prereqs
    exit 0
    ;;
esac

. /usr/share/initramfs-tools/hook-functions

#Copy this binary into 'initramfs' and update-initramfs will include most of its dependent libraries
copy_exec /usr/bin/tpm2_unseal
#This one need explicit copy as well due to dynamic linked and not found by ldd
copy_exec /usr/lib/x86_64-linux-gnu/libtss2-tcti-device.so.0
```

Set the hook script executable.

``` bash
chmod a+x /etc/initramfs-tools/hooks/tpm2
```

Make a unlock luks disk script `/usr/local/bin/unlock-luks-from-tpm.sh` to call `tpm2_unseal` at boot.

``` bash
#!/bin/sh
set -e

echo "Unlock LUKS volume via TPM ..." >&2
#Replace the persistent address if necessary (-Q: slient, only return unseal data)
/usr/bin/tpm2_unseal -Q -c 0x81000000
if [ $? -eq 0 ]; then
        exit
fi
/lib/cryptsetup/askpass "Unlock LUKS volume fallback $CRYPTTAB_SOURCE ($CRYPTTAB_NAME)\nEnter passphrase: "
```

Set the unlock script executable.

``` bash
chmod a+x /usr/local/bin/unlock-luks-from-tpm.sh
```

Update `/etc/crypttab` to use our unlock script for getting key.

``` bash
#Just add ",keyscript=/usr/local/bin/unlock-luks-from-tpm.sh" at the end
dm_crypt-1 UUID=xxxx-xxxxx-xxxx-xxxx none luks,keyscript=/usr/local/bin/unlock-luks-from-tpm.sh
```

Build the final 'initramfs' image and reboot to test

``` bash
#Update the current image (-u) and verbose the output (-v)
update-initramfs -u -v

#If you not update the current image and build a new image, then remember to call 'update-grub' to update grub config
#update-grub

#Reboot to test
reboot
```

References:
- [How LUKS works](https://gist.github.com/joonas-fi/b6bfc564a1a7f3d6b120728db5d48b61)
- [TPM-JS - How TPM works](https://google.github.io/tpm-js)
- [TPM2](https://www.slideshare.net/BrandonArvanaghi/practical-trusted-platform-module-tpm2-programming)
- [Disk Encryption Tutorial by TPM](https://tpm2-software.github.io/2020/04/13/Disk-Encryption.html)
- [Configure Secure Boot + TPM](https://threat.tevora.com/secure-boot-tpm-2)
