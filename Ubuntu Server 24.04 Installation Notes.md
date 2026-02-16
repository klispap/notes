

## Install with BTRFS

## Configure swap (it is disabled by default on BTRFS installations)
From: http://askubuntu.com/questions/1206157/ddg#1206161

It is possible to use a swap file on btrfs, but there are some considerations that need taking care of.
btrfs filesystem doesn't let to create snapshots if there is a working swap file on the subvolume. That means that it is highly recommended to place a swap file on a separate subvolume.

Let's assume that the current swap is already off, the / is on /dev/sda1 and Ubuntu is installed with / on @ subvolume and /home is on @home subvolume.

Mount `/dev/sda1` to `/mnt`.
```
sudo mount /dev/sda1 /mnt
```

If you run ls `/mnt`, you'll see @, @home and other subvolumes that may be there.

Create a new @swap subvolume.

```
sudo btrfs sub create /mnt/@swap
```

Unmount /dev/sda1 from /mnt.
```
sudo umount /mnt
```

Create /swap directory where we plan to mount the @swap subvolume.
```
sudo mkdir /swap
```

Mount the @swap subvolume to /swap.
```
sudo mount -o subvol=@swap /dev/sda1 /swap
```

Create the swap file.
```
sudo touch /swap/swapfile
```

Set 600 permissions to the file.
```
sudo chmod 600 /swap/swapfile
```

Disable COW for this directory.
```
sudo chattr +C /swap
```
Set size of the swap file to 4G as an example.
```
sudo dd if=/dev/zero of=/swap/swapfile bs=1M count=4096
```

Format the swapfile.
```
sudo mkswap /swap/swapfile
```

Turn the swap file on.
```
sudo swapon /swap/swapfile
```

Now the new swap should be working.

You also need to update /etc/fstab to mount all this on boot. Add there two lines:
```
UUID=XXXXXXXXXXXXXXX /swap btrfs subvol=@swap 0 0 /swap/swapfile none swap sw 0 0
```
The UUID is the one of your /dev/sda1.

Swap file can't be located on a btrfs raid of any sort.

Validate the changes
```
sudo findmnt --verify --verbose
```
