

## Bug - not handling user variables correctly in boot cmd

I have come across an odd issue. It seems that user variables in the boot string aren't being replaced correctly for the hyperv-iso builder. The files in this repo can reproduce the problem.

Initially it was stuck in GRUB as if the boot command was never being processed. Eventually I figured out that the part of the boot string that was supposed to drop to the GRUB command line was never entered. I rolled it into boot_cmd, and it got further.

As of this point, 
`packer.exe build -only=hyperv-iso -var-file=ubuntu1204.json ubuntu.json`

This will hang at the language chooser screen in the installer. 


I brought up `dmesg` on the second VT. It shows that user vars weren't handled correctly

[dmesg boot string shows missing user variables](screenshots/missing_vars.png)


## Note: Cannot get SSH IP address for Ubuntu 12.04 LTS

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

I added this to Ubuntu.json, but it still fails for 12.04. It looks like the package was never added to that release.


## Bug: shutdown command fails

The VM is shut down as expected, but it triggers a failure. It's happening both with my changes to the boxcutter images, and also to [@taliesin's packer-basebox](https://github.com/taliesins/packer-baseboxes/blob/master/hyperv-ubuntu-16.04.json) .

This seems to be tracked at https://github.com/taliesins/packer-baseboxes/issues/2



```none
==> hyperv-iso: Pausing after run of step 'StepProvision'. Press enter to continue. ==> hyperv-iso: Gracefully halting virtual machine...
2016/12/11 23:56:13 packer.exe: 2016/12/11 23:56:13 Executing shutdown command: echo 'vagrant'|sudo -S shutdown -P now
2016/12/11 23:56:13 packer.exe: 2016/12/11 23:56:13 opening new ssh session
2016/12/11 23:56:13 packer.exe: 2016/12/11 23:56:13 starting remote command: echo 'vagrant'|sudo -S shutdown -P now
2016/12/11 23:56:13 packer.exe: 2016/12/11 23:56:13 Remote command exited without exit status or exit signal.
2016/12/11 23:56:13 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepProvision'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepProvision'. Press enter to continue.
2016/12/11 23:56:34 ui: scan err: unexpected newline
2016/12/11 23:56:34 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepConnect'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepConnect'. Press enter to continue.
2016/12/11 23:56:36 ui: scan err: unexpected newline
2016/12/11 23:56:36 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepTypeBootCommand'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepTypeBootCommand'. Press enter to continue.
2016/12/11 23:56:37 ui: scan err: unexpected newline
2016/12/11 23:56:37 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepRun'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepRun'. Press enter to continue.
2016/12/11 23:56:39 ui: scan err: unexpected newline
2016/12/11 23:56:40 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepConfigureVlan'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepConfigureVlan'. Press enter to continue.
2016/12/11 23:56:42 ui: scan err: unexpected newline
2016/12/11 23:56:42 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepMountSecondaryDvdImages'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepMountSecondaryDvdImages'. Press enter to continue.
2016/12/11 23:56:43 ui: scan err: unexpected newline
2016/12/11 23:56:43 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepMountGuestAdditions'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepMountGuestAdditions'. Press enter to continue.
2016/12/11 23:56:44 ui: scan err: unexpected newline
2016/12/11 23:56:44 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepMountFloppydrive'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepMountFloppydrive'. Press enter to continue.
2016/12/11 23:56:44 ui: scan err: unexpected newline
==> hyperv-iso: Cleanup floppy drive...
2016/12/11 23:56:44 ui: ==> hyperv-iso: Cleanup floppy drive...
2016/12/11 23:56:46 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepMountDvdDrive'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepMountDvdDrive'. Press enter to continue.
2016/12/11 23:56:47 ui: scan err: unexpected newline
==> hyperv-iso: Clean up os dvd drive...
2016/12/11 23:56:47 ui: ==> hyperv-iso: Clean up os dvd drive...
2016/12/11 23:56:49 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepEnableIntegrationService'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepEnableIntegrationService'. Press enter to continue.
2016/12/11 23:56:59 ui: scan err: unexpected newline
2016/12/11 23:56:59 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepCreateVM'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepCreateVM'. Press enter to continue.
2016/12/11 23:56:59 ui: scan err: unexpected newline
==> hyperv-iso: Unregistering and deleting virtual machine...
2016/12/11 23:56:59 ui: ==> hyperv-iso: Unregistering and deleting virtual machine...
2016/12/11 23:57:00 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepCreateSwitch'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepCreateSwitch'. Press enter to continue.
2016/12/11 23:57:05 ui: scan err: unexpected newline
2016/12/11 23:57:05 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepHTTPServer'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepHTTPServer'. Press enter to continue.
2016/12/11 23:57:05 ui: scan err: unexpected newline
2016/12/11 23:57:05 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepCreateFloppy'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepCreateFloppy'. Press enter to continue.
2016/12/11 23:57:06 ui: scan err: unexpected newline
2016/12/11 23:57:06 packer.exe: 2016/12/11 23:57:06 Deleting floppy disk: C:\Users\Patrick\AppData\Local\Temp\packer807449581
2016/12/11 23:57:06 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepDownload'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepDownload'. Press enter to continue.
2016/12/11 23:57:06 ui: scan err: unexpected newline
2016/12/11 23:57:06 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepOutputDir'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepOutputDir'. Press enter to continue.
2016/12/11 23:57:07 ui: scan err: unexpected newline
==> hyperv-iso: Deleting output directory...
2016/12/11 23:57:07 ui: ==> hyperv-iso: Deleting output directory...
2016/12/11 23:57:07 ui: ask: ==> hyperv-iso: Pausing before cleanup of step 'StepCreateTempDir'. Press enter to continue.
==> hyperv-iso: Pausing before cleanup of step 'StepCreateTempDir'. Press enter to continue.
2016/12/11 23:57:07 ui: scan err: unexpected newline
2016/12/11 23:57:07 ui: ==> hyperv-iso: Deleting temporary directory...
==> hyperv-iso: Deleting temporary directory...
2016/12/11 23:57:07 ui error: Build 'hyperv-iso' errored: Shutdown command has not successful.

ExitStatus: 2300218

Stdout:

Stderr:
2016/12/11 23:57:07 Waiting on builds to complete...
2016/12/11 23:57:07 Builds completed. Waiting on interrupt barrier...
2016/12/11 23:57:07 machine readable: error-count []string{"1"}
2016/12/11 23:57:07 ui error:
==> Some builds didn't complete successfully and had errors:
2016/12/11 23:57:07 machine readable: hyperv-iso,error []string{"Shutdown command has not successful.\n\nExitStatus: 2300218\n\nStdout: \n\nStderr: "}
2016/12/11 23:57:07 ui error: --> hyperv-iso: Shutdown command has not successful.

ExitStatus: 2300218

Stdout:

Stderr:
2016/12/11 23:57:07 ui:
==> Builds finished but no artifacts were created.
2016/12/11 23:57:07 waiting for all plugin processes to complete...
Build 'hyperv-iso' errored: Shutdown command has not successful.

ExitStatus: 2300218

Stdout:

Stderr:

==> Some builds didn't complete successfully and had errors:
--> hyperv-iso: Shutdown command has not successful.

ExitStatus: 2300218

Stdout:

Stderr:

==> Builds finished but no artifacts were created.
```


## 4/8/2017 trying ubuntu1604.json with Packer v1.0.0


Note: prepend this with `cmd /c` if running from PowerShell

```
packer.exe build -only=hyperv-iso -var-file=ubuntu1604.json ubuntu.json
```

