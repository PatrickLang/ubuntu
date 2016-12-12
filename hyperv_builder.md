

## Bug - not handling user variables correctly in boot cmd

I have come across an odd issue. It seems that user variables in the boot string aren't being replaced correctly for the hyperv-iso builder. The files in this repo can reproduce the problem.

Initially it was stuck in GRUB as if the boot command was never being processed. Eventually I figured out that the part of the boot string that was supposed to drop to the GRUB command line was never entered. I rolled it into boot_cmd, and it got further.

As of this point, 
`packer.exe build -only=hyperv-iso -var-file=ubuntu1204.json ubuntu.json`

This will hang at the language chooser screen in the installer. 


I brought up `dmesg` on the second VT. It shows that user vars weren't handled correctly

[dmesg boot string shows missing user variables](screenshots/missing_vars.png)


## Bug? Cannot get SSH IP address for some distros

Ubuntu 12.04 doesn't have the updated Linux integration components by default. Getting the IP address of a running VM won't work.


```none
016/12/11 21:47:44 packer.exe: 2016/12/11 21:47:44 Special code '<enter>' found, replacing with: [1c 9c]
2016/12/11 21:48:02 packer.exe: 2016/12/11 21:48:02 [INFO] Waiting for SSH, up to timeout: 2h46m40s
2016/12/11 21:48:02 ui: ==> hyperv-iso: Waiting for SSH to become available...
==> hyperv-iso: Waiting for SSH to become available...
2016/12/11 21:48:06 packer.exe: 2016/12/11 21:48:06 [DEBUG] Error getting SSH address: No ip address.
2016/12/11 21:48:16 packer.exe: 2016/12/11 21:48:16 [DEBUG] Error getting SSH address: No ip address.
2016/12/11 21:48:25 packer.exe: 2016/12/11 21:48:25 [DEBUG] Error getting SSH address: No ip address.
2016/12/11 21:48:34 packer.exe: 2016/12/11 21:48:34 [DEBUG] Error getting SSH address: No ip address.
```

Even once installation is complete, the IP address cannot be found


```none
get-vm -name ubuntu1204 | Get-VMNetworkAdapter

Name            IsManagementOs VMName     SwitchName MacAddress   Status                      IPAddresses
----            -------------- ------     ---------- ----------   ------                      -----------
Network Adapter False          ubuntu1204 Wifi       00155D016FA0 {Degraded, ProtocolVersion} {}
```

[this thread](http://stackoverflow.com/questions/33027204/how-can-i-get-hyper-v-to-detect-my-ubuntu-vms-ip-address) seems to show its a known issue resolved by running
`sudo apt-get install "linux-cloud-tools-$(uname -r)"`