# 02: Getting nested VT-x working for EVE-NG

*2026-11-25*

EVE-NG runs its routers and servers as VMs inside itself, and EVE-NG is a VM in VMware. That needs VMware to pass VT-x down to it, which it can't do if Windows has started its own hypervisor. Mine had.

## Checking Windows

```
(Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard).VirtualizationBasedSecurityStatus
2

systeminfo | Select-String "Hyper-V"
Hyper-V Requirements: A hypervisor has been detected.
```

VBS was running and a hypervisor was loaded.

## Turning off the Hyper-V features

* Memory integrity off (Windows Security, Core isolation)
* Unticked Hyper-V, Virtual Machine Platform, Windows Hypervisor Platform and Windows Sandbox in `optionalfeatures`
* `bcdedit /set hypervisorlaunchtype off` so the hypervisor doesn't start at boot
* Restart

Then I checked which services were still running on VBS:

```
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard |
  Select-Object VirtualizationBasedSecurityStatus, SecurityServicesConfigured, SecurityServicesRunning

2   {3, 4}   {3, 4}
```

3 is System Guard Secure Launch (4 comes with it). That was the last thing holding VBS on.

## Turning off Secure Launch

No Group Policy key under `HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard`, so a registry change would stick. I backed up the key, then turned off Secure Launch and VBS:

```
reg export "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" "C:\Lab\backups\deviceguard-backup.reg"
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\SystemGuard -Name Enabled -Value 0
New-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard -Name EnableVirtualizationBasedSecurity -Value 0 -PropertyType DWord -Force
```

Then a full Restart, not Shut down, since Fast Startup would bring the old boot state back.

```
VirtualizationBasedSecurityStatus SecurityServicesRunning
                                2 {0}
```

Nothing running on VBS anymore.

## Result inside EVE-NG

```
egrep -c '(vmx|svm)' /proc/cpuinfo
12

kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```

EVE-NG sees VT-x, so the nodes will run with hardware acceleration.
