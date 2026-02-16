
# Install with BTRFS

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

Creating a swap file on Btrfs is trickier than other filesystems because it requires specific attributes (like NOCOW) to be set before the file is populated. If it’s not visible after a reboot, you likely missed the persistent configuration step or the file attributes are invalid for Btrfs. 

Create the swap file.
```
sudo touch /swap/swapfile
```

Set 600 permissions to the file.
```
sudo chmod 600 /swap/swapfile
```

Disable COW for this directory and file.
```
sudo chattr +C /swap
sudo chattr +C /swap/swapfile
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
# 1. Mount the dedicated swap subvolume
UUID=UUID_OF_PARTITION /swap btrfs subvol=@swap,noatime,nodatacow 0 0

# 2. Activate the swap file within that subvolume
/swap/swapfile none swap sw 0 0
```
The UUID is the one of your /dev/sda1.

Swap file can't be located on a btrfs raid of any sort.

Validate the changes
```
sudo findmnt --verify --verbose
```

# Enable automatic snapshots

This method uses systemd timers to automatically handle snapshots and cleanup. 
Preparation (Crucial): Ensure your Ubuntu installation is on a Btrfs filesystem.

Install Snapper:
```bash
sudo apt update && sudo apt install -y snapper
```

Create Your Configuration:
Create a config for your root partition:
```bash
sudo snapper -c myconfig create-config -f btrfs /
```

Configure Automation:
Automating Snapshots
To automate snapshot creation, you can enable timeline snapshots in your configuration file. Edit the configuration file located in /etc/snapper/configs/myconfig and set:
Edit the config file at `/etc/snapper/configs/` to set your retention limits:
```
TIMELINE_CREATE="yes": Enables hourly snapshots.
TIMELINE_LIMIT_HOURLY="5": Keeps the last 5 hours.
TIMELINE_LIMIT_DAILY="7": Keeps the last 7 days.
```

Enable the Timers:
Run these commands to start the automatic background tasks:
```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

Creating Snapshots
You can create different types of snapshots:

Single Snapshots: Created manually.
Pre/Post Snapshots: Automatically created before and after specific operations.
To create a snapshot, use:
```
sudo snapper -c myconfig create
```

Listing and Managing Snapshots
To view your snapshots, run:
```
sudo snapper -c myconfig list
```

You can also view differences between snapshots:
```
sudo snapper -c myconfig diff <snapshot_id1>..<snapshot_id2>
```

Rolling Back to a Snapshot
If you need to revert to a previous state, use:

```
sudo snapper -c myconfig rollback <snapshot_id>
```

# Timezone

Find the desired timezone
```
timedatectl list-timezones | grep "London"
```

Install the desired timezone
```
sudo timedatectl set-timezone Europe/London
```

Verify
```
timedatectl
```
