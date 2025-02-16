<!-- <div align="center"> -->
<img src="https://github.com/archlinux/archinstall/raw/master/docs/logo.png" alt="drawing" width="200"/>

<!-- </div> -->
# Arch Installer
[![Lint Python and Find Syntax Errors](https://github.com/archlinux/archinstall/actions/workflows/flake8.yaml/badge.svg)](https://github.com/archlinux/archinstall/actions/workflows/flake8.yaml)

Just another guided/automated [Arch Linux](https://wiki.archlinux.org/index.php/Arch_Linux) installer with a twist.
The installer also doubles as a python library to install Arch Linux and manage services, packages, and other things inside the installed system *(Usually from a live medium)*.

* archinstall [discord](https://discord.gg/aDeMffrxNg) server
* archinstall [#archinstall:matrix.org](https://matrix.to/#/#archinstall:matrix.org) Matrix channel
* archinstall [#archinstall@irc.libera.chat:6697](https://web.libera.chat/?channel=#archinstall)
* archinstall [documentation](https://archinstall.archlinux.page/)

# Installation & Usage
```shellgy
sudo pacman -S archinstall
```

Alternative ways to install are `git clone` the repository or `pip install --upgrade archinstall`.

## Running the [guided](https://github.com/archlinux/archinstall/blob/master/archinstall/scripts/guided.py) installer

Assuming you are on an Arch Linux live-ISO or installed via `pip`:
```shell
archinstall
```

## Running the [guided](https://github.com/archlinux/archinstall/blob/master/archinstall/scripts/guided.py) installer using `git`

```shell
    # cd archinstall-git
    # python -m archinstall
```

#### Advanced
Some additional options that most users do not need are hidden behind the `--advanced` flag.

## Running from a declarative configuration file or URL

`archinstall` can be run with a JSON configuration file. There are 2 different configuration files to consider,
the `user_configuration.json` contains all general installation configuration, whereas the `user_credentials.json`
contains the sensitive user configuration such as user password, root password, and encryption password.

An example of the user configuration file can be found here
[configuration file](https://github.com/archlinux/archinstall/blob/master/examples/config-sample.json)
and an example of the credentials configuration here
[credentials file](https://github.com/archlinux/archinstall/blob/master/examples/creds-sample.json).

**HINT:** The configuration files can be auto-generated by starting `archinstall`, configuring all desired menu
points and then going to `Save configuration`.

To load the configuration file into `archinstall` run the following command
```shell
archinstall --config <path to user config file or URL> --creds <path to user credentials config file or URL>
```

# Help or Issues

If you come across any issues, kindly submit your issue here on Github or post your query in the
[discord](https://discord.gg/aDeMffrxNg) help channel.

When submitting an issue, please:
* Provide the stacktrace of the output if applicable
* Attach the `/var/log/archinstall/install.log` to the issue ticket. This helps us help you!
  * To extract the log from the ISO image, one way is to use<br>
    ```shell
    curl -F'file=@/var/log/archinstall/install.log' https://0x0.st
    ```


# Available Languages

Archinstall is available in different languages which have been contributed and are maintained by the community.
The language can be switched inside the installer (first menu entry). Bear in mind that not all languages provide
full translations as we rely on contributors to do the translations. Each language has an indicator that shows
how much has been translated.

Any contributions to the translations are more than welcome,
to get started please follow [the guide](https://github.com/archlinux/archinstall/blob/master/archinstall/locales/README.md)

## Fonts
The ISO does not ship with all fonts needed for different languages.
Fonts that use a different character set than Latin will not be displayed correctly. If those languages
want to be selected then a proper font has to be set manually in the console.

All available console fonts can be found in `/usr/share/kbd/consolefonts` and set with `setfont LatGrkCyr-8x16`.


# Scripting your own installation

## Scripting interactive installation

There are some examples in the `examples/` directory that should serve as a starting point.

The following is a small example of how to script your own *interactive* installation:

```python
from pathlib import Path

from archinstall import Installer, ProfileConfiguration, profile_handler, User
from archinstall.default_profiles.minimal import MinimalProfile
from archinstall.lib.disk.device_model import FilesystemType
from archinstall.lib.disk.encryption_menu import DiskEncryptionMenu
from archinstall.lib.disk.filesystem import FilesystemHandler
from archinstall.lib.interactions.disk_conf import select_disk_config

fs_type = FilesystemType('ext4')

# Select a device to use for the installation
disk_config = select_disk_config()

# Optional: ask for disk encryption configuration
data_store = {}
disk_encryption = DiskEncryptionMenu(disk_config.device_modifications, data_store).run()

# initiate file handler with the disk config and the optional disk encryption config
fs_handler = FilesystemHandler(disk_config, disk_encryption)

# perform all file operations
# WARNING: this will potentially format the filesystem and delete all data
fs_handler.perform_filesystem_operations()

mountpoint = Path('/tmp')

with Installer(
        mountpoint,
        disk_config,
        disk_encryption=disk_encryption,
        kernels=['linux']
) as installation:
    installation.mount_ordered_layout()
    installation.minimal_installation(hostname='minimal-arch')
    installation.add_additional_packages(['nano', 'wget', 'git'])

    # Optionally, install a profile of choice.
    # In this case, we install a minimal profile that is empty
    profile_config = ProfileConfiguration(MinimalProfile())
    profile_handler.install_profile_config(installation, profile_config)

    user = User('archinstall', 'password', True)
    installation.create_users(user)
```

This installer will perform the following actions:

* Prompt the user to configure the disk partitioning
* Prompt the user to setup disk encryption
* Create a file handler instance for the configured disk and the optional disk encryption
* Perform the disk operations (WARNING: this will potentially format the disks and erase all data)
* Install a basic instance of Arch Linux *(base base-devel linux linux-firmware btrfs-progs efibootmgr)*
* Install and configures a bootloader to partition 0 on UEFI. On BIOS, it sets the root to partition 0.
* Install additional packages *(nano, wget, git)*
* Create a new user

> **To create your own ISO with this script in it:** Follow [ArchISO](https://wiki.archlinux.org/index.php/archiso)'s guide on creating your own ISO.

## Script non-interactive automated installation

For an example of a fully scripted, automated installation please refer to the example
[full_automated_installation.py](https://github.com/archlinux/archinstall/blob/master/examples/full_automated_installation.py)

## Unattended installation based on MAC address

Archinstall comes with an [unattended](https://github.com/archlinux/archinstall/blob/master/examples/mac_address_installation.py)
example which will look for a matching profile for the machine it is being run on, based on any local MAC address.
For instance, if the machine the code is executed on has the MAC address `52:54:00:12:34:56` it will look for a profile called
[52-54-00-12-34-56.py](https://github.com/archlinux/archinstall/blob/master/archinstall/default_profiles/tailored.py).
If it's found, the unattended installation will begin and source that profile as its installation procedure.

# Profiles

`archinstall` comes with a set of pre-configured profiles available for selection during the installation process.

- [Desktop](https://github.com/archlinux/archinstall/tree/master/archinstall/default_profiles/desktops)
- [Server](https://github.com/archlinux/archinstall/tree/master/archinstall/default_profiles/servers)

The profiles' definitions and the packages they will install can be directly viewed in the menu, or
[default profiles](https://github.com/archlinux/archinstall/tree/master/archinstall/default_profiles)


# Testing

## Using a Live ISO Image

If you want to test a commit, branch, or bleeding edge release from the repository using the standard Arch Linux Live ISO image,
replace the archinstall version with a newer one and execute the subsequent steps defined below.

*Note: When booting from a live USB, the space on the ramdisk is limited and may not be sufficient to allow
running a re-installation or upgrade of the installer. In case one runs into this issue, any of the following can be used
- Resize the root partition https://wiki.archlinux.org/title/Archiso#Adjusting_the_size_of_the_root_file_system
- The boot parameter `copytoram=y` (https://gitlab.archlinux.org/archlinux/mkinitcpio/mkinitcpio-archiso/-/blob/master/docs/README.bootparams#L26)
can be specified which will copy the root filesystem to tmpfs.*

1. You need a working network connection
2. Install the build requirements with `pacman -Sy; pacman -S git python-pip gcc pkgconf`
   *(note that this may or may not work depending on your RAM and current state of the squashfs maximum filesystem free space)*
3. Uninstall the previous version of archinstall with `pip uninstall --break-system-packages archinstall`
4. Now clone the latest repository with `git clone https://github.com/archlinux/archinstall`
5. Enter the repository with `cd archinstall`
   *At this stage, you can choose to check out a feature branch for instance with `git checkout v2.3.1-rc1`*
6. To run the source code, there are 2 different options:
   - Run a specific branch version from source directly using `python -m archinstall`, in most cases this will work just fine, the
      rare case it will not work is if the source has introduced any new dependencies that are not installed yet
   - Installing the branch version with `pip install --break-system-packages .` and `archinstall`

## Without a Live ISO Image

To test this without a live ISO, the simplest approach is to use a local image and create a loop device.<br>
This can be done by installing `pacman -S arch-install-scripts util-linux` locally and doing the following:

    # truncate -s 20G testimage.img
    # losetup --partscan --show --find ./testimage.img
    # pip install --upgrade archinstall
    # python -m archinstall --script guided
    # qemu-system-x86_64 -enable-kvm -machine q35,accel=kvm -device intel-iommu -cpu host -m 4096 -boot order=d -drive file=./testimage.img,format=raw -drive if=pflash,format=raw,readonly,file=/usr/share/ovmf/x64/OVMF_CODE.fd -drive if=pflash,format=raw,readonly,file=/usr/share/ovmf/x64/OVMF_VARS.fd

This will create a *20 GB* `testimage.img` and create a loop device which we can use to format and install to.<br>
`archinstall` is installed and executed in [guided mode](#docs-todo). Once the installation is complete, ~~you can use qemu/kvm to boot the test media.~~<br>
*(You'd actually need to do some EFI magic in order to point the EFI vars to the partition 0 in the test medium, so this won't work entirely out of the box, but that gives you a general idea of what we're going for here)*

There's also a [Building and Testing](https://github.com/archlinux/archinstall/wiki/Building-and-Testing) guide.<br>
It will go through everything from packaging, building and running *(with qemu)* the installer against a dev branch.


# FAQ

## Keyring out-of-date
For a description of the problem see https://archinstall.archlinux.page/help/known_issues.html#keyring-is-out-of-date-2213 and discussion in issue https://github.com/archlinux/archinstall/issues/2213.

For a quick fix the below command will install the latest keyrings

```pacman -Sy archlinux-keyring```

## How to dual boot with Windows

To install Arch Linux alongside an existing Windows installation using  `archinstall`, follow these steps:

1. Ensure some unallocated space is available for the Linux installation after the Windows installation.
2. Boot into the ISO and run `archinstall`.
3. Choose `Disk configuration` -> `Manual partitioning`.
4. Select the disk on which Windows resides.
5. Select `Create a new partition`.
6. Choose a filesystem type.
7. Determine the start and end sectors for the new partition location (values can be suffixed with various units).
8. Assign the mountpoint `/` to the new partition.
9. Assign the `Boot/ESP` partition the mountpoint `/boot` from the partitioning menu.
10. Confirm your settings and exit to the main menu by choosing `Confirm and exit`.
11. Modify any additional settings for your installation as necessary.
12. Start the installation upon completion of setup.


# Mission Statement

Archinstall promises to ship a [guided installer](https://github.com/archlinux/archinstall/blob/master/archinstall/scripts/guided.py) that follows
the [Arch Linux Principles](https://wiki.archlinux.org/index.php/Arch_Linux#Principles) as well as a library to manage services, packages, and other Arch Linux aspects.

The guided installer ensures a user-friendly experience, offering optional selections throughout the process. Emphasizing its flexible nature, these options are never obligatory.
In addition, the decision to use the guided installer remains entirely with the user, reflecting the Linux philosophy of providing full freedom and flexibility.

---

Archinstall primarily functions as a flexible library for managing services, packages, and other elements within an Arch Linux system.
This core library is the backbone for the guided installer that Archinstall provides. It is also designed to be used by those who wish to script their own custom installations.

Therefore, Archinstall will try its best to not introduce any breaking changes except for major releases which may break backward compatibility after notifying about such changes.


# Contributing

Please see [CONTRIBUTING.md](https://github.com/archlinux/archinstall/blob/master/CONTRIBUTING.md)
