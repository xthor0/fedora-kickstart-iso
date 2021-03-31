# Tools to create a custom Fedora kickstart ISO

Eventually, this'll be all wrapped up in a nice Bash or Python script.

## Extracting the Fedora netinst ISO

1. Download the Fedora Server netinst ISO here: https://getfedora.org/en/server/download/
1. Extract the ISO to a temp directory: `7z x -oiso -x!'[BOOT]' Fedora-Server-netinst-x86_64-33-1.2.iso`
1. Modify the proper boot menu files to account for using kickstart (more on that below)
1. Copy your customized kickstart file to the root folder where the ISO was extracted. If you ran the `7z` command above, your kickstart file should be at `iso/ks.cfg`

## Modifying the ISO boot menus
There are two files that need to be modified:
* For MBR installations: `isolinux/isolinux.cfg`
* For UEFI installations: `EFI/BOOT/grub.cfg`

If doing this by hand, first I use `sed` to change the volume of the ISO the config files are looking for, to match my `genisoimage` command below: `sed -i 's/Fedora-S-dvd-x86_64-33/Fedora-33-ks/g' isolinux.cfg` (repeat for `EFI/BOOT/grub.cfg`)

Then, edit both files and add desired menu entries. I like to keep the original menu entries, just in case I need to perform a stock installation. So, `grub.cfg` ends up looking something like this:

~~~
menuentry 'Install Fedora 33 Kickstart' --class fedora --class gnu-linux --class gnu --class os {
	linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=Fedora-33-ks quiet inst.ks=hd:LABEL=Fedora-33-ks:/ks.cfg
	initrdefi /images/pxeboot/initrd.img
}
menuentry 'Install Fedora 33' --class fedora --class gnu-linux --class gnu --class os {
	linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=Fedora-33-ks quiet
	initrdefi /images/pxeboot/initrd.img
}
~~~

And, `isolinux.cfg` looks like this:

~~~
label linux
  menu label ^Install Fedora 33 via Kickstart
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=Fedora-33-ks quiet inst.ks=hd:LABEL=Fedora-33-ks:/ks.cfg

label linuxorig
  menu label ^Install Fedora 33
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=Fedora-33-ks quiet
~~~

## Kickstart files
There are example files in the repo. Have a look.

One problem I have had is, if I specify using the `fedora-updates` (which, you'll see is commented out), my systems have exhibited very strange behavior. Wifi sometimes doesn't work, and one system would lock up at random times. I found it safer to point at the default repo and update it after the fact. No idea why this happens.

## Creating a new ISO file
Use `genisoimage`:
~~~
genisoimage -o fedora33kickstart.iso -V 'Fedora-33-ks' -J -R -c isolinux/boot.cat -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e images/efiboot.img -no-emul-boot iso
~~~

