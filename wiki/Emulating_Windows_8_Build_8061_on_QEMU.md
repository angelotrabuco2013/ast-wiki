# Emulating Windows 8 Build 8061 on QEMU
## Patching files
HAL: 
```diff
- HalpAcpiGetTable(HalpTimerLoaderBlock, 'TIGE', NULL, NULL);
+ HalpAcpiGetTable(HalpTimerLoaderBlock, 'TDTG', NULL, NULL);
```

HAL functions: HalpGitDiscover()
```diff
- NewTimer.Interrupt.Gsi = *((ULONG *)(Table + 52));
- NewTimer.Interrupt.Polarity = InterruptActiveLow;
- BYTE Flags = *((BYTE *)(Table + 62));
- if (Flags & 2)
- {
-     NewTimer.Interrupt.Polarity = InterruptActiveHigh;
- }
- NewTimer.Interrupt.Mode = (Flags & 1);
+ NewTimer.Interrupt.Gsi = *((ULONG *)(Table + 64));
+ NewTimer.Interrupt.Polarity = InterruptActiveHigh;
+ ULONG Flags = *((ULONG *)(Table + 68));
+ if (Flags & 2)
+ {
+     NewTimer.Interrupt.Polarity = InterruptActiveLow;
+ }
+ NewTimer.Interrupt.Mode = (Flags & 1);
```

Registry fix:
```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\ACPI]

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\ACPI\PNP0D40]

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\ACPI\PNP0D40\0]
"Capabilities"=dword:00000030
"ContainerID"="{00000000-0000-0000-ffff-ffffffffffff}"
"HardwareID"=hex(7):41,00,43,00,50,00,49,00,5c,00,50,00,4e,00,50,00,30,00,44,\
  00,34,00,30,00,00,00,2a,00,50,00,4e,00,50,00,30,00,44,00,34,00,30,00,00,00,\
  00,00
"ClassGUID"="{a0a588a4-c46f-4b37-b7ea-c82fe89870c6}"
"Service"="sdbus"
"DeviceDesc"="@sdbus.inf,%acpi\\pnp0d40.devicedesc%;SDA Standard Compliant SD Host Controller (compatible)"
"Driver"="{a0a588a4-c46f-4b37-b7ea-c82fe89870c6}\\0000"
"Mfg"="@sdbus.inf,%generic%;SDA Standard Compliant SD Host Controller Vendor"
"UINumberDescFormat"="@sdbus.inf,%SDBUSSlot%;SD Host Slot %1!u!"
"ConfigFlags"=dword:00000000
```

DWM:
```cpp
DWORD CDwmAppHost::VerifyGraphicsCapabilities(MIL_CHANNEL hChannel, BOOL fIsBitmapRemoting, GraphicsCapabilities *pCaps)
{
	pCaps->fValidCompositionMode = 1;
	pCaps->dwDisabledEvent = 0;
	return 1;
}
```
## Tips
* The original binaries for the ACPI, SD bus and partition management drivers, as well as binaries for the boot manager and boot loader can be used under emulation.
* Reapply the registry values outlined above after setup completes the second phase of setup to prevent an ```INACCESSIBLE_BOOT_DEVICE``` bugcheck loop, as the respective driver enumerator is removed from the Windows registry during hardware detection. This issue only occurs once and will not recur in subsequent boots after reapplication.

##  Installation
Install the OS normally. Due to slowdowns on QEMU, installation can take a long time to complete.

## Conclusion
If done correctly, you should have 8061 running on QEMU.
