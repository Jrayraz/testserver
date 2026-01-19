Lightweight Functional Test Server
This repository provides a safe, lightweight, configuration‑accurate clone of your production Linux server for testing changes before applying them to the real system.

It is designed for environments where:

Safety is critical

Storage space is limited

Production must never be risked

Changes must be tested in a faithful environment

You want a real bootable VM, not a container

You want GUI access to the filesystem (Disks, GParted, Files, etc.)

You want CLI access to the filesystem (ls, cp, mv, rm, chroot, etc.)

You want to run commands inside the VM

You want snapshots and reversibility

You want a clone that mirrors your OS, configs, and package set

You do NOT want to copy large production data

This system is built around a single script:
testserver (or testserver.sh if you prefer).

It creates and manages a qcow2 disk image that:

behaves like a real disk

appears in GUI disk tools

can be mounted via NBD

can be inspected and modified

can be booted as a VM

can be cloned from your production environment

can be snapshotted and restored

The result is a functional twin of your production server —
but lightweight, safe, and disposable.

Why This Exists
Traditional test environments fail you in one of two ways:

❌ Full clones
Too large

Too slow

Too risky

Too tied to production data

Too expensive in disk space

❌ Generic ISOs
Not faithful to your real server

Not configured like production

Not running the same packages

Not running the same services

Not useful for real testing

✔ This system solves both problems
It creates a Minimal Functional Clone (MFC):

Same OS version

Same package set

Same systemd units

Same configs

Same environment

Same behavior

But:

No large data

No user files

No logs

No containers

No VM images

No media

No backups

This keeps the test server tiny (10–20 GB), fast, and safe.

Core Features
1. qcow2 disk creation
Lightweight (default 25 GB)

GPT partition table

Single ext4 filesystem

Ready for mounting or VM boot

2. NBD exposure
The qcow2 disk can be attached as:

Code
/dev/nbd0
/dev/nbd0p1
This makes it visible to:

GNOME Disks (“Disks”)

GParted

File managers (“Files”)

Any GUI filesystem tool

3. CLI filesystem access
Mount the filesystem at:

Code
/mnt/testserver
Then use:

ls

cp

mv

rm

grep

sed

chroot

systemctl (inside chroot)

apt (inside chroot)

4. VM booting
Boot the qcow2 as a VM:

6 GB RAM

2 vCPUs

KVM acceleration

QEMU Guest Agent socket prewired

Optional GUI window

5. Offline command execution
Run commands inside the test server’s filesystem without booting the VM:

Code
testserver exec-offline "apt-get update"
testserver exec-offline "systemctl enable my-service"
testserver exec-offline "ls /"
6. Functional clone of production
The command:

Code
testserver build-functional-clone
Creates a lightweight clone of your production server by:

Bootstrapping a minimal OS (same codename as host)

Copying your configs (/etc, /usr/local, /opt)

Syncing your installed package set

Installing GRUB

Excluding all large data

7. Snapshots
Before testing something risky:

Code
testserver snapshot before-change
If something breaks:

Code
testserver restore before-change
8. Safety-first design
Never attaches NBD while VM is running

Never boots VM while NBD is attached

Never copies large data

Never touches production system files

Always reversible

Requirements
This subsystem is designed for Debian/Ubuntu‑based hosts because it relies on:

debootstrap

apt-mark

dpkg

grub-install

update-grub

It will work on:

Ubuntu (all modern LTS releases)

Debian (Bullseye, Bookworm, etc.)

Linux Mint (with minor adjustments)

Pop!_OS

Other Debian derivatives

Host Packages Required
Install these before using the system:

bash
sudo apt install \
  qemu-system-x86 \
  qemu-utils \
  qemu-kvm \
  debootstrap \
  parted \
  gparted \
  gnome-disk-utility \
  rsync \
  grub-pc \
  socat
Optional but recommended:

bash
sudo apt install \
  udisks2 \
  udisksctl \
  qemu-guest-agent
Kernel Requirements
Your host must support:

/dev/nbdX devices

KVM virtualization

Load the NBD module:

bash
sudo modprobe nbd max_part=16
Installation
Clone your repository:

bash
git clone https://github.com/<yourname>/<yourrepo>.git
cd <yourrepo>
Make the script executable:

bash
chmod +x testserver
(Optional) Install it system‑wide:

bash
sudo cp testserver /usr/local/bin/
Now you can run:

bash
testserver help
Initial Setup
1. Create the qcow2 disk
This creates a lightweight 25 GB qcow2 image with:

GPT partition table

Single ext4 filesystem

Ready for mounting or cloning

Run:

bash
sudo testserver init-disk
This produces:

Code
testserver.qcow2
2. Build the functional clone
This is the heart of the system.

It creates a lightweight functional clone of your production server by:

Bootstrapping a minimal OS matching your host’s codename

Copying your configs

Syncing your installed package set

Installing GRUB

Excluding all large data

Run:

bash
sudo testserver build-functional-clone
This step:

Does not copy user data

Does not copy logs

Does not copy containers

Does not copy VM images

Does not copy large directories

It only copies:

/etc (minus secrets)

/usr/local

/opt

Your installed package list

Your systemd units

Your OS version

Your environment

This produces a faithful but tiny clone of your server.

3. Boot the VM
Once the clone is built:

bash
testserver run
This launches:

A VM with 6 GB RAM

2 vCPUs

KVM acceleration

A GUI window

QEMU Guest Agent socket

You now have a safe, disposable, faithful test server.

Safety Rules
These are critical:

Rule 1 — Never attach NBD while the VM is running
If the VM is running, do not run:

attach-gui

attach-cli

exec-offline

Rule 2 — Never boot the VM while NBD is attached
If NBD is attached, do not run:

run

Rule 3 — Always detach when done
bash
sudo testserver detach
Rule 4 — Use snapshots before risky changes
bash
testserver snapshot before-change
Rule 5 — Restore if something breaks
bash
testserver restore before-change
Command Reference
This section documents every command provided by the testserver subsystem.
Each command is designed to be:

safe

atomic

reversible

predictable

production‑friendly

1. init-disk
Create a brand‑new qcow2 disk image with:

GPT partition table

Single ext4 filesystem

Lightweight default size (25 GB)

Ready for cloning or mounting

Usage
bash
sudo testserver init-disk
What it does
Creates testserver.qcow2

Connects it to /dev/nbd0

Creates a GPT partition table

Creates a single ext4 partition

Formats it as ext4 with label testroot

Disconnects NBD

When to use
First‑time setup

Rebuilding from scratch

Creating a fresh test environment

2. attach-gui
Expose the qcow2 disk to GUI tools such as:

GNOME Disks (“Disks”)

GParted

File managers (“Files”)

Any GUI filesystem browser

Usage
bash
sudo testserver attach-gui
What it does
Connects testserver.qcow2 to /dev/nbd0

Runs partprobe so the kernel sees /dev/nbd0p1

Does not mount anything (so GUI tools can mount it themselves)

What you can do after running it
Open Disks → see /dev/nbd0

Open GParted → inspect partitions

Click “Mount” in Disks → browse in file manager

Resize partitions (if needed)

Run filesystem checks

When to use
You want to visually inspect the filesystem

You want to mount it via GUI

You want to use GParted or Disks

3. attach-cli
Attach and mount the qcow2 filesystem for command‑line access.

Usage
bash
sudo testserver attach-cli
What it does
Connects testserver.qcow2 to /dev/nbd0

Runs partprobe

Mounts /dev/nbd0p1 at /mnt/testserver

What you can do after running it
Use normal Linux commands:

Code
ls /mnt/testserver
cp file /mnt/testserver/etc/
rm /mnt/testserver/tmp/*
grep -R "pattern" /mnt/testserver
You can also:

edit configs

inspect logs (if any)

modify systemd units

prepare the filesystem before boot

When to use
You want CLI access

You want to script changes

You want to prepare the test server before booting

4. detach
Cleanly unmount and disconnect the qcow2 disk from NBD.

Usage
bash
sudo testserver detach
What it does
Unmounts /mnt/testserver (if mounted)

Disconnects /dev/nbd0 from the qcow2

Leaves the qcow2 file clean and ready for VM boot

When to use
After attach-gui

After attach-cli

After exec-offline

Before running the VM

Before shutting down the host

Safety
This command is mandatory before running:

bash
testserver run
Running the VM while NBD is attached risks filesystem corruption.

5. run
Boot the lightweight functional clone as a virtual machine.

Usage
bash
testserver run
What it does
Boots the qcow2 disk as a VM

Uses KVM acceleration for near‑native speed

Allocates:

6 GB RAM

2 vCPUs

Opens a GUI window for the VM

Exposes a QEMU Guest Agent socket for future online command execution

Boots directly from the cloned filesystem (no ISO required)

When to use
Testing configuration changes

Observing system behavior

Running services in a safe environment

Verifying package installs

Debugging systemd units

Experimenting with kernel modules

Running your agentic subsystems safely

Safety
Before running:

bash
sudo testserver detach
Never boot the VM while NBD is attached.

6. snapshot
Create a qcow2 snapshot for safe rollback.

Usage
bash
testserver snapshot
Or with a name:

bash
testserver snapshot before-upgrade
What it does
Creates a qcow2 internal snapshot

Does not duplicate the entire disk

Uses qcow2’s copy‑on‑write mechanism

Extremely fast and lightweight

When to use
Before testing a risky change

Before modifying systemd units

Before upgrading packages

Before kernel experiments

Before modifying your agentic subsystems

Example
bash
testserver snapshot pre-change
7. restore
Restore a qcow2 snapshot.

Usage
bash
testserver restore pre-change
What it does
Reverts the qcow2 disk to the snapshot state

Discards all changes made after the snapshot

Fully restores the test server to a known‑good state

When to use
After a failed experiment

After a misconfiguration

After a broken package install

After a kernel or systemd failure

Example
bash
testserver restore before-upgrade
8. exec-offline
Run commands inside the test server’s filesystem without booting the VM.

Usage
bash
sudo testserver exec-offline "<command>"
Examples
bash
sudo testserver exec-offline "ls /"
sudo testserver exec-offline "apt-get update"
sudo testserver exec-offline "systemctl enable my-service"
sudo testserver exec-offline "sed -i 's/foo/bar/' /etc/myconfig"
What it does
Connects qcow2 → /dev/nbd0

Mounts /dev/nbd0p1 at /mnt/testserver

Runs:

Code
chroot /mnt/testserver /bin/bash -c "<command>"
Unmounts

Disconnects NBD

When to use
Preparing the filesystem before boot

Installing packages offline

Editing configs offline

Fixing broken boot environments

Enabling/disabling services

Running scripts inside the clone without launching the VM

Why this is powerful
You can modify the test server even if it won’t boot.

This is your “surgical repair mode.”

9. build-functional-clone
Create a lightweight, configuration‑accurate clone of your production server inside the qcow2 disk.

This is the command that transforms the qcow2 image from “empty disk” into a functional twin of your real server — without copying any large data.

Purpose
This command builds a Minimal Functional Clone (MFC) of your host system.
It mirrors:

OS version

Package set

Systemd units

Kernel modules

Configurations

Custom scripts

Operational behavior

But does not copy:

user data

logs

containers

VM images

media

backups

large directories

This keeps the clone tiny, fast, and safe.

Usage
bash
sudo testserver build-functional-clone
What It Does (Step-by-Step)
1. Attaches the qcow2 disk via NBD
Connects to /dev/nbd0

Mounts the ext4 filesystem at /mnt/testserver

2. Bootstraps a minimal OS
Uses debootstrap to install a minimal base system matching your host’s codename:

If host is Ubuntu 22.04 → clone uses Jammy

If host is Debian Bookworm → clone uses Bookworm

etc.

This ensures OS parity with production.

3. Binds host pseudo-filesystems
Inside the chroot, it binds:

/dev

/proc

/sys

This allows package installation and bootloader installation to work correctly.

4. Copies host configuration
Copies lightweight, essential directories:

/etc (minus secrets and host‑specific files)

/usr/local (custom scripts, binaries)

/opt (optional, usually small)

This ensures configuration parity with production.

5. Syncs installed package set
Runs:

Code
apt-mark showmanual
Then installs the same manually-installed packages inside the clone.

This ensures package parity with production.

Packages that would break the clone (e.g., kernel images, grub variants) are automatically excluded.

6. Installs GRUB bootloader
Installs GRUB into the qcow2 disk so the VM can boot normally.

7. Cleans up
Unmounts /dev, /proc, /sys

Unmounts the filesystem

Disconnects NBD

8. Leaves you with a fully bootable, faithful test server
You can now run:

bash
testserver run
And boot into your lightweight clone.

Why This Is Safe
No user data is copied

No large directories are copied

No secrets are copied

No host‑specific hardware configs are copied

No risk to production

Clone is fully isolated

Clone is disposable

Clone is reversible via snapshots

Why This Is Lightweight
A typical clone is:

10–20 GB total

Boots in seconds

Runs fast under KVM

Uses minimal RAM

Contains only what is needed to replicate system behavior

This is ideal for:

testing configuration changes

testing package upgrades

testing systemd units

testing kernel modules

testing your agentic subsystems

validating operational rituals

verifying scripts

experimenting safely

When to Use It
After creating a new qcow2 disk

After major host upgrades

When you want a fresh test environment

When you want a faithful but tiny clone

When you want to test changes before applying them to production

Example Workflow
bash
sudo testserver init-disk
sudo testserver build-functional-clone
testserver run
Then:

test changes

break things

observe behavior

snapshot/restore

iterate safely


Full Workflow Examples
This section provides complete, real‑world workflows that engineers can follow step‑by‑step.
These examples assume the script is installed as testserver.

Workflow A — Build a Fresh Test Server From Scratch
This is the most common workflow.

1. Create the qcow2 disk
bash
sudo testserver init-disk
2. Build the functional clone
bash
sudo testserver build-functional-clone
3. Boot the VM
bash
testserver run
You now have a lightweight, faithful clone of your production server.

Workflow B — Inspect or Modify the Filesystem Using GUI Tools
1. Attach for GUI
bash
sudo testserver attach-gui
2. Open GUI tools
GNOME Disks (“Disks”)

GParted

File manager (“Files”)

You will see /dev/nbd0 appear as a real disk.

3. Mount the partition in the GUI
Click “Mount” in Disks or GParted.

4. Browse or edit files
Use your file manager to inspect or modify the filesystem.

5. Unmount in GUI
6. Detach
bash
sudo testserver detach
Workflow C — Modify the Filesystem Using CLI Tools
1. Attach for CLI
bash
sudo testserver attach-cli
2. Modify files
bash
ls /mnt/testserver
cp new.conf /mnt/testserver/etc/
rm /mnt/testserver/tmp/*
3. Detach
bash
sudo testserver detach
Workflow D — Run Commands Inside the Test Server Without Booting It
This is “offline surgery mode.”

Example: Update packages
bash
sudo testserver exec-offline "apt-get update"
Example: Enable a service
bash
sudo testserver exec-offline "systemctl enable my-service"
Example: Fix a broken config
bash
sudo testserver exec-offline "sed -i 's/foo/bar/' /etc/myconfig"
Detach happens automatically
The command handles mounting and unmounting for you.

Workflow E — Test a Risky Change Safely
1. Snapshot before the change
bash
testserver snapshot before-change
2. Apply the change offline
bash
sudo testserver exec-offline "apply-your-change-here"
3. Boot the VM
bash
testserver run
4. Test behavior
5. If it breaks, restore
bash
testserver restore before-change
Workflow F — Rebuild the Test Server After Host Upgrades
If you upgrade your production server:

bash
sudo testserver build-functional-clone
This rebuilds the clone with:

new OS version

new package set

updated configs

Your test server stays in sync with production.

Safety Model
This subsystem is designed with a strict safety model to protect production.

Rule 1 — Only one owner of the qcow2 at a time
Allowed:
VM running

OR NBD attached

Forbidden:
VM running and NBD attached

NBD attached and VM running

This prevents filesystem corruption.

Rule 2 — Never copy large data
The functional clone intentionally excludes:

/home

/var/log

/var/lib/docker

/var/lib/libvirt

/srv

/mnt

/media

backups

datasets

VM images

containers

media files

This keeps the clone lightweight and safe.

Rule 3 — Never copy secrets
The clone excludes:

/etc/shadow

/etc/gshadow

SSH host keys

password backups

machine‑specific identifiers

This prevents accidental credential leakage.

Rule 4 — Always detach when done
bash
sudo testserver detach
This ensures:

clean unmount

clean NBD disconnect

no dangling devices

no risk of corruption

Rule 5 — Use snapshots before experiments
Snapshots are:

instant

lightweight

reversible

safe

Always snapshot before testing something risky.

Best Practices
1. Treat the test server as disposable
If it breaks:

bash
sudo testserver build-functional-clone
Rebuilds it from scratch in minutes.

2. Keep the qcow2 small
25 GB is enough for:

OS

configs

packages

logs

experiments

If you need more, resize the qcow2 — but keep it lean.

3. Use offline mode for most changes
Offline mode is safer:

bash
sudo testserver exec-offline "<command>"
It avoids:

booting

runtime conflicts

service interference

4. Use GUI mode for visual inspection
attach-gui is perfect for:

partition inspection

filesystem checks

browsing configs

verifying changes visually

5. Rebuild after major host changes
If you:

upgrade the OS

install major packages

change systemd units

modify core configs

Rebuild the clone:

bash
sudo testserver build-functional-clone
6. Keep the script in version control
This subsystem is a core operational tool.
Version it like any other critical infrastructure.

7. Document your own exclusions
If you have custom large directories, add them to the exclusion list in the clone builder.


Advanced Topics
This subsystem is intentionally modular and extensible.
The following sections describe advanced capabilities and optional enhancements.

QEMU Guest Agent Integration
The VM is pre‑wired to expose a QEMU Guest Agent (QGA) socket on the host:

Code
/tmp/testserver-qga.sock
This allows online command execution inside the running VM.

1. Enable QEMU Guest Agent inside the VM
Inside the VM:

bash
sudo apt install qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
2. Send commands from the host
Example: list root directory inside the running VM:

bash
echo '{"execute":"guest-exec","arguments":{"path":"/bin/sh","arg":["-c","ls /"],"capture-output":true}}' \
  | socat - UNIX-CONNECT:/tmp/testserver-qga.sock
What this enables
Running commands inside the VM without SSH

Gathering logs

Querying VM state

Automating tests

Building agentic subsystems

Integrating with orchestration tools

This is optional but extremely powerful.

Customizing the Functional Clone
The clone builder (build-functional-clone) is intentionally modular.
You can customize:

which directories are copied

which packages are installed

which exclusions are applied

which services are enabled

how the bootloader is installed

how the filesystem is structured

Common Customizations
1. Add additional directories
If you have small but important directories:

bash
rsync -aHAX /my/small/dir/ "${MOUNT_POINT}/my/small/dir/"
2. Exclude additional large directories
Add exclusions to the rsync commands.

3. Add custom packages
Append to /tmp/testserver-manual-packages.txt before installation.

4. Add custom systemd units
Copy them into:

Code
/etc/systemd/system/
Then enable them offline:

bash
sudo testserver exec-offline "systemctl enable my-unit"
5. Add custom kernel modules
Copy them into:

Code
/lib/modules/<kernel-version>/
Then rebuild module dependencies:

bash
sudo testserver exec-offline "depmod -a"
Troubleshooting
Problem: VM won’t boot
Fix
Use offline mode to inspect logs:

bash
sudo testserver exec-offline "journalctl -xb --no-pager | head -200"
Or fix configs:

bash
sudo testserver exec-offline "nano /etc/fstab"
If all else fails:

bash
testserver restore <snapshot>
Problem: NBD device is busy
Fix
Unmount manually:

bash
sudo umount /mnt/testserver || true
Disconnect NBD:

bash
sudo qemu-nbd --disconnect /dev/nbd0 || true
Then retry:

bash
sudo testserver detach
Problem: GUI tools don’t show the disk
Fix
Ensure NBD is loaded:

bash
sudo modprobe nbd max_part=16
Ensure the disk is attached:

bash
sudo testserver attach-gui
Open Disks or GParted again.

Problem: GRUB install fails
Fix
Install GRUB inside the clone:

bash
sudo testserver exec-offline "apt-get install grub-pc -y"
Then reinstall:

bash
sudo testserver exec-offline "grub-install /dev/nbd0"
sudo testserver exec-offline "update-grub"
Problem: Clone is too large
Fix
Increase exclusions in the clone builder:

/var/cache

/var/lib/apt/lists

/usr/share/doc

/usr/share/man

Or shrink qcow2:

bash
qemu-img convert -O qcow2 testserver.qcow2 compact.qcow2
mv compact.qcow2 testserver.qcow2
Problem: Clone is missing a package
Fix
Install it offline:

bash
sudo testserver exec-offline "apt-get install <package>"
Or add it to the clone builder.

Extending the System
This subsystem is designed to be extended.
Common extensions include:

automated test runners

online command orchestration

snapshot‑based CI pipelines

agentic subsystems that test themselves

multi‑node cluster simulation

unified node orchestration

reproducible operational rituals

If you want, I can help you design:

a clone-sync command

a clone-diff command

a clone-verify command

a clone-upgrade command

a clone-export command

a multi‑node test cluster

a snapshot‑based CI/CD pipeline


Philosophy
This subsystem is built on a simple but powerful idea:

You don’t need a full clone of your server.
You need a functional clone of your server.

A full clone is:

heavy

slow

risky

expensive

unnecessary

A functional clone is:

lightweight

faithful

safe

disposable

reproducible

operationally sound

This system gives you the behavior of your production server
without the baggage of your production server.

It mirrors:

your OS

your package set

your configs

your systemd units

your operational environment

But it intentionally avoids:

user data

logs

containers

VM images

media

backups

large directories

This keeps the clone small, fast, and safe —
and lets you test changes with confidence.

Design Principles
This subsystem is built around a few core principles:

1. Safety First
No command in this system touches production data.
No command risks corruption.
No command runs without explicit user intent.

2. One Owner at a Time
The qcow2 disk is either:

attached via NBD
or

owned by the VM

Never both.

3. Lightweight by Default
The clone is intentionally minimal:

minimal OS

minimal filesystem

minimal data footprint

But fully functional.

4. Reproducibility
You can rebuild the clone at any time:

bash
sudo testserver build-functional-clone
This ensures your test environment always matches production.

5. Auditability
Every action is explicit.
Every step is visible.
Nothing is hidden.

6. Reversibility
Snapshots allow you to:

test

break

observe

roll back

All without risk.

Why This Matters
Modern systems are complex.
Configuration drift is real.
Package interactions are unpredictable.
Systemd behavior can be subtle.
Kernel modules can conflict.
Upgrades can break things.
Your agentic subsystems need a safe place to evolve.

This subsystem gives you:

a safe lab

a faithful mirror

a disposable sandbox

a reproducible environment

a controlled testing ground

It lets you test changes before they hit production.
It lets you break things without consequences.
It lets you explore without fear.

Who This Is For
This subsystem is for engineers who:

care about safety

care about reproducibility

care about operational clarity

care about system behavior

care about testing before deploying

care about not breaking production

care about understanding their systems deeply

It’s for people who want:

a real VM

a real filesystem

a real OS

real packages

real configs

real behavior

But without the weight of a full clone.

Final Notes
This subsystem is intentionally simple.

It is intentionally explicit.

It is intentionally modular.

It is intentionally safe.

It is intentionally lightweight.

It is not a replacement for:

backups

snapshots

full system imaging

disaster recovery

It is a testing tool, not a recovery tool.

But for testing?
For experimentation?
For operational confidence?
For safe iteration?

It is exactly what you need.

Closing
If you want to extend this subsystem —
add commands, add automation, add orchestration, add multi‑node support —
the architecture is ready for it.

You can build:

cluster simulations

agentic test harnesses

snapshot‑based CI pipelines

automated config verifiers

reproducible operational rituals

unified node simulations

This subsystem is the foundation.

You can build anything on top of it.
