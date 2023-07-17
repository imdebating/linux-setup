# List of Packages Installed
After installing the base system from the install iso, and after running `dnf update -y`, we installed the following using `dnf`.

  1. tmux
  2. https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
  3. snapd
     1. Brave Browser
     2. 1Password
  4. wireshark
  5. xrdp
  6. vdo
  7. kmod-kvdo
  8. dkms

We started and enabled each of the systemd usits, where possible.
`vdo` and `kmod-kvdo` are not part of the UEK kernel from Oracle, so we need to compile the module and and service for the current kernel.
To accomplish this, we first added a reminder to /usr/bin/dnf-3 to not update the kernel until the vdo dependencies have been addressed.

Following the instructions on [GitHub dm-vdo/kvdo](https://github.com/dm-vdo/kvdo#building), we built and installed the module.
We also used `dkms` to automagically build the module for future kernel installs.
