
# Unraid7 Persistent Root Folder with omyzsh
Guide on how to install Oh my zsh on Unraid with persistent root directory

I've tried a bunch of different solutions to making Unraid allow for persistent root, especially to install utilities such as oh-my-zsh.  After trying a bunch of different methods I think I've settled on what I think is the most straightforward, which is to establish a writable ext4 volume on the boot drive and mounting it as root.

Here are my scripts to accomplish this.  I think the risk is rather low because you can always remove the image by plugging the usb boot drive into another computer and deleting it.  This will revert back to unraid's default /root.

Please let me know how this works for you and if you run into any issues.  Please be sure to backup your new root partition frequently as this new approach has not been widely tested.

First create the image that will become your /root folder.  WARNING:  If this image already exists it will be overwritten, especially if it's not alredy mounted.



```
  

# Create the base image file that will house the contents of /root
# This creates a 1.5GB image, you can edit the 1500 to your liking

dd if=/dev/zero  of=/boot/root.img  bs=1M  count=1500

# Format it w/ ext4 file system

mkfs.ext4 /boot/root.img

# Mount the image and copy the current default unraid content of /root

mkdir -p  /tmp/root
mount -o  loop  /boot/root.img  /tmp/root
rsync -av  /root/  /tmp/root
umount /tmp/root
```


Now let's install the zsh package into /boot/extra so that it is installed each time unraid starts.

```
# Make sure /boot/extra exists and install zsh

mkdir /boot/extra

wget -p  /boot/extra  http://mirrors.slackware.com/slackware/slackware64-current/slackware64/ap/zsh-5.9-x86_64-1.txz
```

Reboot then test that zsh is properly installed before enabling the script below.  You can simply login in and type 'zsh' to see if you enter the shell.

Now use the user scripts plugin to create a script that will run on first array mount.  This script will mount the root directory and change the default root shell to zsh.  Disabling this script will go back to the unraid default behavior.

Script should run 'At First Array Start Only'.

```
#!/bin/bash

# Mount our persistent root image over the default /root
mount -o loop /boot/root.img /root

#change root shell to /bin/zsh
sed -i '/^root:/s#/bin/bash#/bin/zsh#' /etc/passwd
```

At this point you now have a persietent root folder.  You can reboot to try it out.



Finally, you can install ohmyzsh

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Hope this works!


#How to remove the mount in case something goes wrong?

A large portion of unraid runs off of a ram drive, including /root.  You can delete /root, reboot, and it will appear again unscathed.  This is what makes using utilities like ohmyzsh problematic since they want to store things in the home directory, which for root is /root.

If something breaks you'll likely just get unraid's default /root folder, however you can remove the mount of your image by manually removing the mount script you created.

Plug the usb drive into another computer, look in /boot/config/plugins/user.scripts/scripts\<your script name\>. You can edit or delete the file named 'script' in this folder.


