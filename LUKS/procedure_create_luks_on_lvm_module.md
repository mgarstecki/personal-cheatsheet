# Procedure : create a LUKS-on-LVM volume and reference it for mounting

This procedure allows to:
* Create a LVM volume
* Create a random key in a file, used to unlock the volume.
* Create the LUKS volume locked with that key and sensible parameters.
* Unlock the volume and mount it on the running system.
* Add it to dmcrypt configuration and fstab for automated unlocking and mounting on boot.

Personal naming conventions :
* The base LVM volume name is prefixed by `cr_`.
* The LUKS volume name is the same as above without the `cr_` prefix.

## Creation and mount on running kernel

```bash
# Create the LVM volume
lvcreate -LsizeG -n cr_name vg-name

# Create the encryption key as a file (4k)
dd if=/dev/urandom of=/path/to/key/file bs=1024 count=4

# Format the volume as LUKS, with 2s of key derivation
cryptsetup luksFormat --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 2 /dev/mapper/vg-name-cr_name /path/to/key/file

# Open the created LUKS volume
cryptsetup luksOpen -d /path/to/key/file  /dev/mapper/vg-name-cr_name name

# Format the LUKS volume
mkfs.ext4 /dev/mapper/name

# Mount it
mount /dev/mapper/name /wherever
```

## Configuration of dmcrypt and fstab

Add a section to `/etc/conf.d/dmcrypt`. For LVM volume `vg-name/cr_name`, with LUKS mapping `name` and key `/path/to/key/file`:

```
target=name
source='/dev/mapper/vg-name-cr_name'
key='/path/to/key/file'
```

Add a line to `/etc/fstab`:

```
/dev/mapper/name				/wherever		ext4		noatime		0 2
```

