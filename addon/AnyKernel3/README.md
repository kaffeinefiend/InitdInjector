----------------------------------------------------------------------------------
AnyKernel3 - Flashable Zip Template for Kernel Releases with Ramdisk Modifications
----------------------------------------------------------------------------------
### by osm0sis @ xda-developers ###

"AnyKernel is a template for an update.zip that can apply any kernel to any ROM, regardless of ramdisk." - Koush

AnyKernel3 pushes the format even further by allowing kernel developers to modify the underlying ramdisk for kernel feature support easily using a number of included command methods along with properties and variables.

_A script based on Galaxy Nexus (tuna) is included for reference._

## // Properties / Variables ##
```
block=/dev/block/platform/omap/omap_hsmmc.0/by-name/boot;
is_slot_device=0;
ramdisk_compression=auto;
OG_AK=false;
```

`block=auto` instead of a direct block filepath enables detection of the device boot partition for use with broad, device non-specific zips. Also accepts specifically `boot` or `recovery`.

`is_slot_device=1` enables detection of the suffix for the active boot partition on slot-based devices and will add this to the end of the supplied `block=` path. Also accepts `auto` for use with broad, device non-specific zips.

`ramdisk_compression=auto` allows automatically repacking the ramdisk with the format detected during unpack. Changing `auto` to `gz`, `lzo`, `lzma`, `xz`, `bz2`, `lz4`, or `lz4-l` (for lz4 legacy) instead forces the repack as that format, and using `cpio` or `none` will (attempt to) force the repack as uncompressed.

`customdd="<arguments>"` may be added to allow specifying additional dd parameters for devices that need to hack their kernel directly into a large partition like mmcblk0, or force use of dd for flashing.

`slot_select=active|inactive` may be added to allow specifying the target slot. If omitted the default remains `active`.

`OG_AK=true|false` set to true for OG AK (no ramdisk patching - ramdisk is never unpacked/repacked). Note that this needs to be set to false if using Direct Sepolicy patching since sepolicy is found in the ramdisk

`SYSTEM_ROOT_RD=true|false` set to true if SYSTEM_ROOT device detected with ramdisk in /system (all non-A/B devices with Android Pie+ have this setup along with possibly others)

## // Important Notes ##

```
Currently pushing modules is to the active slot only.

If $SYSTEM_ROOT is true, then instead of having the ramdisk in the boot img, it's stored in the system partition.
This means that all ramdisk files will be stored at `/system_root` rather than $ramdisk.
Make sure you account for this in your script if applicable

```

## // Command Methods ##
```
dump_boot
split_boot
unpack_ramdisk
replace_string <file> <if search string> <original string> <replacement string> <scope>
replace_section <file> <begin search string> <end search string> <replacement string>
remove_section <file> <begin search string> <end search string>
insert_line <file> <if search string> <before|after> <line match string> <inserted line>
replace_line <file> <line replace string> <replacement line>
remove_line <file> <line match string>
prepend_file <file> <if search string> <patch file>
insert_file <file> <if search string> <before|after> <line match string> <patch file>
append_file <file> <if search string> <patch file>
replace_file <file> <permissions> <patch file>
patch_fstab <fstab file> <mount match name> <fs match type> block|mount|fstype|options|flags <original string> <replacement string>
patch_cmdline <cmdline entry name> <replacement string>
patch_prop <prop file> <prop name> <new prop value>
patch_ueventd <ueventd file> <device node> <permissions> <chown> <chgrp>
repack_ramdisk
flash_boot
flash_dtbo
write_boot
reset_ak [keep]
setup_ak
```

__"if search string"__ is the string it looks for to decide whether it needs to add the tweak or not, so generally something to indicate the tweak already exists. __"cmdline entry name"__ behaves somewhat like this as a match check for the name of the cmdline entry to be changed/added by the _patch_cmdline_ function, followed by the full entry to replace it. __"prop name"__ also serves as a match check in _patch_prop_ for a property in the given prop file, but is only the prop name as the prop value is specified separately.

Similarly, __"line match string"__ and __"line replace string"__ are the search strings that locate where the modification needs to be made for those commands, __"begin search string"__ and __"end search string"__ are both required to select the first and last lines of the script block to be replaced for _replace_section_, and __"mount match name"__ and __"fs match type"__ are both required to narrow the _patch_fstab_ command down to the correct entry.

__"scope"__ may be specified as __"global"__ to force all instances of the string targeted by _replace_string_ to be replaced. Omitted or set to anything else and it will perform the default first-match replacement.

__"before|after"__ requires you simply specify __"before"__ or __"after"__ for the placement of the inserted line, in relation to __"line match string"__.

__"block|mount|fstype|options|flags"__ requires you specify which part (listed in order) of the fstab entry you want to check and alter.

_dump_boot_ and _write_boot_ are the default method of unpacking/repacking, but for more granular control, or omitting ramdisk changes entirely ("OG AK" mode), these can be separated into _split_boot; unpack_ramdisk_ and _repack_ramdisk; flash_boot_ respectively. _flash_dtbo_ can be used to flash a dtbo image. It is automatically included in _write_boot_ but can be called separately if using "OG AK" mode or creating a dtbo only zip.

Multi-partition zips can be created by removing the ramdisk and patch folders from the zip and including instead "-files" folders named for the partition (without slot suffix), e.g. boot-files + recovery-files, or kernel-files + ramdisk-files (on some Treble devices). These then contain zImage, and ramdisk, patch, etc. subfolders for each partition. To setup for the next partition, simply set `block=` (without slot suffix) and `ramdisk_compression=` for the new target partition and use the _reset_ak_ command.

Similarly, multi-slot zips can be created with the normal zip layout for the active (current) slot, then resetting for the inactive slot by setting `block=` (without slot suffix) again, `slot_select=inactive` and `ramdisk_compression=` for the target slot and using the _reset_ak keep_ command, which will retain the patch and any added ramdisk files for the next slot.

You may also use _ui_print "\<text\>"_ to write messages back to the recovery during the modification process, _abort "\<text>"_ to abort with optional message, and _file_getprop "\<file\>" "\<property\>"_ and _contains "\<string\>" "\<substring\>"_ to simplify string testing logic you might want in your script.

## // Binary Inclusion ##

The AK3 repo includes current ARM/x86 builds of `magiskboot` and `magiskpolicy` by default to keep the basic package small. Builds for other architectures and optional binaries (see below) are available from the latest Magisk zip, or my latest AIK-mobile and FlashIt packages, respectively, here:

https://forum.xda-developers.com/showthread.php?t=2073775 (Android Image Kitchen thread)  
https://forum.xda-developers.com/showthread.php?t=2239421 (Odds and Ends thread)

Optional supported binaries which may be placed in /tools to enable built-in expanded functionality are as follows:
* `mkbootfs` - for broken recoveries, or, booted flash support for a script/app via bind mount to /tmp (deprecated/use with caution)
* `flash_erase`, `nanddump`, `nandwrite` - MTD block device support for devices where the `dd` command is not sufficient
* `dumpimage`, `mkimage` - DENX U-Boot uImage format support
* `mboot` - Intel OSIP Android image format support
* `unpackelf`, `mkbootimg` - Sony ELF kernel.elf format support, repacking as AOSP standard boot.img for unlocked bootloaders
* `elftool` (with `unpackelf`) - Sony ELF kernel.elf format support, repacking as ELF for older Sony devices
* `mkmtkhdr` (with `unpackelf`) - MTK device boot image section headers support for Sony devices
* `futility` + `chromeos` test keys directory - Google ChromeOS signature support
* `BootSignature_Android.jar` + `avb` keys directory - Google Android Verified Boot (AVB) signature support
* `rkcrc` - Rockchip KRNL ramdisk image support

## // Instructions ##

1. Place final kernel build product, e.g. Image.gz-dtb or zImage to name a couple, in the zip root (separate dt, dtb or recovery_dtbo, and/or dtbo should also go here for devices that require custom ones, each will fallback to the original if not included)

2. Place any required ramdisk files in /ramdisk

3. Place any required patch files (generally partial files which go with commands) in /patch

4. Modify the anykernel.sh to add boot partition location, permissions for added ramdisk files, and any custom install logic

5. Modify the uninstall.sh to add any custom uninstall logic

If supporting a recovery that forces zip signature verification (like Cyanogen Recovery) then you will need to also sign your zip using the method I describe here:

http://forum.xda-developers.com/android/software-hacking/dev-complete-shell-script-flashable-zip-t2934449

Not required, but any tweaks you can't hardcode into the source (best practice) should be added with an additional init.tweaks.rc or bootscript.sh to minimize the necessary ramdisk changes.

It is also extremely important to note that for the broadest AK3 compatibility it is always better to modify a ramdisk file rather than replace it.

For further support and usage examples please see the AnyKernel3 XDA thread: https://forum.xda-developers.com/showthread.php?t=2670512

Have fun!