

## Bug - not handling user variables correctly in boot cmd

I have come across an odd issue. It seems that user variables in the boot string aren't being replaced correctly for the hyperv-iso builder. The files in this repo can reproduce the problem.

Initially it was stuck in GRUB as if the boot command was never being processed. Eventually I figured out that the part of the boot string that was supposed to drop to the GRUB command line was never entered. I rolled it into boot_cmd, and it got further.

As of this point, 
`packer.exe build -only=hyperv-iso -var-file=ubuntu1204.json ubuntu.json`

This will hang at the language chooser screen in the installer. 


I brought up `dmesg` on the second VT. It shows that user vars weren't handled correctly

[dmesg boot string shows missing user variables](screenshots/missing_vars.png)


