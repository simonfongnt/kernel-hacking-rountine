# kernel-hacking-rountine
Kernel Hacking Rountine
Continue from [kernel-hacking-quickstart](https://github.com/simonfongnt/kernel-hacking-quickstart)


## Use Screen
On the kernel hacking machine, run:
```
screen
```
Other remote machine, run:
```
screen -x
```

## Update the kernel
Run
```
git fetch origin                  # fetch the latest changes
git rebase origin/staging-testing # update branch to include the changes in the staging tree
```

## Modify an existing modules
Search for existing modules
```
lsmod | sort | less
```
Look for path of the modules, e.g:
```
modinfo ahci
> filename:       /lib/modules/6.13.0-rc3+/kernel/drivers/ata/ahci.ko
> ...
```
see which files complied into the kernel drivers, e.g.:
```
git grep ahci -- '*Makefile'
```
e.g locate files with probe function:
```
grep " ahci_probe" drivers/ata/ahci*.c
> drivers/ata/ahci_platform.c:static int ahci_probe(struct platform_device *pdev)
> drivers/ata/ahci_platform.c:	.probe = ahci_probe,
```
e.g. add a line into the function:
```
static int ahci_probe(struct platform_device *pdev)
{
        ...
        int rc;

        printk(KERN_DEBUG "I can modify the Linux kernel!\n");
        hpriv = ahci_platform_get_resources(pdev,
                                            AHCI_PLATFORM_GET_RESETS);
        ...
}
```
Recompile the changes
```
make -j`nproc`
```
Install the changes
```
sudo make -j`nproc` modules_install install
```
Reloading the modules
```
sudo modprobe -r ahci_platform
sudo modprobe ahci_platform
```
REBOOT, Verify the driver is loaded
```
lsmod | less
```
Verify the message
```
dmesg | less
```

## Undo the changes
```
git reset --hard HEAD        # revert ALL FILES, wiping out all uncommited changes
```

## Find a driver to clean up
```
less drivers/staging/*/TODO  # :n = next file; :p = previous file
scripts/checkpatch.pl -f --terse --show-types --strict path/to/source/file
```

