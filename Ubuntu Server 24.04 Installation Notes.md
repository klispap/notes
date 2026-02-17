
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

**IMPORTANT NOTE:** To diagnose why Snapper cannot detect your environment (ambit), you need to verify your current Btrfs subvolume layout and the default subvolume ID.
1. Check the Current Default Subvolume 
Snapper's rollback command works by changing the default subvolume of the Btrfs partition. If your current default is the top-level (ID 5) rather than a snapshot, Snapper may lose its reference point. 
```
sudo btrfs subvolume get-default /
```
What to look for:
If it says ID 5 (top level), you are booting from the root of the Btrfs partition, which is common but can cause "unknown ambit" errors if Snapper expects a specific snapshot-based boot.
If it lists a specific path like @/.snapshots/X/snapshot, you are currently running in a snapshot. 

2. List Your Subvolume Layout
Snapper often expects a "nested" or "flat" layout depending on your distribution (e.g., openSUSE vs. Arch). 
```
sudo btrfs subvolume list -t /
```
What to look for:
Check if you have a .snapshots subvolume and where it is located relative to your root (/).
Ensure your root subvolume (often named @) has its own entry. 

3. Verify Boot Parameters 
If your /etc/fstab or GRUB configuration explicitly forces a subvolume (e.g., rootflags=subvol=@), it can override the "default" subvolume Snapper tries to set, causing the "unknown ambit" error. 
Check current boot flags:
```
cat /proc/cmdline
```
Check fstab: (Look for the subvol= or subvolid= options for the root / mount). 
```
cat /etc/fstab
```
 
Summary of "Healthy" Snapper Layout
For a standard Snapper rollback to work seamlessly, your layout should typically look like this:
Mount Point: / should be mounted without a specific subvol= ID in fstab, allowing the Btrfs "default" to dictate which subvolume boots.
Config: Your snapper config (/etc/snapper/configs/myconfig) must point to the correct subvolume. 

if your default subvolume is ID 5 (the top-level), Snapper is refusing to rollback because it doesn't "own" the root subvolume layout. This usually happens on non-openSUSE systems (like Arch, Debian, or Fedora) where Snapper wasn't pre-configured to manage the boot process. 
To fix this and complete your rollback to snapshot 16, use one of these methods:
Option 1: Force Rollback with the Classic Ambit 
This is the most direct solution for your specific error. It tells Snapper to ignore the "unknown" environment and treat it as a standard Btrfs setup. 
Run the command:
```
sudo snapper -c myconfig --ambit classic rollback <snapshot_id>
```
Crucial: After the command finishes, check your fstab. If your / entry has a subvol= or subvolid= option, you must remove it for the rollback to work. Snapper uses btrfs subvolume set-default to change what boots, but an explicit mount option in fstab will override it and keep booting your old system. 
What is the output of `cat /etc/fstab`? Knowing if your root is mounted by subvol name or ID will determine if the automatic rollback will actually take effect after you run the command.

```
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/nvme0n1p2 during curtin installation
/dev/disk/by-uuid/cce6d89d-71e2-4254-b29b-a0f6681b2175 / btrfs defaults 0 1
# /boot/efi was on /dev/nvme0n1p1 during curtin installation
/dev/disk/by-uuid/5934-864C /boot/efi vfat defaults 0 1
```
Great news—your /etc/fstab is configured perfectly for Snapper rollbacks.
Because your / mount does not have a subvol= or subvolid= option, the system boots whichever subvolume is currently set as the "default" via Btrfs. This is exactly how Snapper's rollback mechanism is designed to function.
The Solution
Since you are currently on ID 5 (the top-level) and Snapper is confused by the environment, run the rollback with the explicit ambit flag:
```bash
sudo snapper -c myconfig --ambit classic rollback <snapshot_id>
```

What happens next?
Snapper will create two new snapshots (one of your current state and one writable copy of snapshot 16).
It will then run btrfs subvolume set-default to point to the new writable copy of snapshot 16.
You must reboot.
After you reboot
Once you are back in the system, check your default subvolume again:
```bash
sudo btrfs subvolume get-default /
```

It should no longer say ID 5; instead, it should point to a path inside .snapshots. From that point forward, the standard sudo snapper rollback command should work without needing the --ambit flag, as Snapper will now recognize it is running within a snapshot-managed environment.

After you reboot, follow these steps to verify that Snapper successfully updated your environment and that your swap configuration remains active:

1. Verify the Default Subvolume
Ensure the system is no longer using the top-level (ID 5) but is now running from a specific snapshot. 
```
sudo btrfs subvolume get-default /
```
Expected Result: You should see a path like `@/.snapshots/<snapshot_id>/snapshot` (Snapper usually creates a new writable snapshot ID for the rollback). 

2. Check the Snapshots List 
Verify that Snapper has marked the new subvolume as the current and default one.
```
sudo snapper -c myconfig list
```
Expected Result: Look for the snapshot entry marked with an asterisk (*), which indicates the currently booted snapshot. 

3. Verify Swap Activity 
Since your swap is on a dedicated subvolume (@swap), you should confirm it is mounted and active. 
Check Active Swap:
```
sudo swapon --show
```
This should display your /swap/swapfile with its size and priority.
Check Mount Point: 
```
findmnt /swap
```
This confirms that the @swap subvolume is correctly mounted to the /swap directory as defined in your fstab. 
5. Verify No CoW on Swapfile 
Btrfs swapfiles must have the No Data Copy-on-Write (NOCOW) attribute to function correctly. 
```
lsattr /swap/swapfile
```
Expected Result: The output should include the C attribute (e.g., ---------------C--). 
Pro-Tip: If swapon --show is empty after the reboot, your system might have failed to activate the swap due to the subvolume change. In that case, simply run sudo swapon -a to manually trigger the activation based on your fstab. 


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
