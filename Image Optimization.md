# Windows 10 Optimizations for VDI #
## Optimizing the Default Profile ##
The default profile is used as a template for all new users accessing a system. Making it leaner will enhance login times for new users. To do so, follow the following steps outline by James Rankin<sup>1,2</sup>:

1) Install Windows 10 on your virtual machine. When the "Let's start with region. Is this right?" screen appears, press **Ctrl+Shift+F3**.
2) The system will enter Audit Mode. Close the sysprep window.
3) Open an administrative PowerShell session and run the following commands. Each command will open a list of Appx packages. Select the ones you'd like to remove (hold Ctrl), and press OK when satisfied. Most packages will be removed; some will produce errors. Be careful not to remove important components such as libraries or Edge (if you want to keep it).

    - `Get-AppxProvisionedPackage -online | Out-GridView -PassThru | Remove-AppxProvisionedPackage -online -ErrorAction SilentlyContinue`

    - `Get-AppxPackage -AllUsers | Out-GridView -PassThru | Remove-AppxPackage -ErrorAction SilentlyContinue`

4) Open the Start Menu and remove any remaining junk.
5) We need to save the cleaned-up Start Menu for future users. Return to an administrative PowerShell session, run the following command:
    - `Export-StartLayout -Path $ENV:LOCALAPPDATA\Microsoft\Windows\Shell\LayoutModification.xml`
6) To finish up, create a file called unattend.xml with the following content (or create your own):
``` <?xml version=”1.0″ encoding=”utf-8″?><unattend xmlns=”urn:schemas-microsoft-com:unattend”>
<settings pass=”specialize”>
<component name=”Microsoft-Windows-Shell-Setup” processorArchitecture=”amd64” publicKeyToken=”31bf3856ad364e35″ language=”neutral” versionScope=”nonSxS” xmlns:wcm=”http://schemas.microsoft.com/WMIConfig/2002/State” xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance”>
<CopyProfile>true</CopyProfile>
</component>
</settings>
</unattend> 
```
7) Run SysPrep (be sure you point to the correct path for **unattend.xml**):
    - `c:\windows\system32\sysprep\sysprep.exe /generalize /oobe /reboot /unattend:c:\optimizations\unattend.xml`
8) If you get an error, check **C:\Windows\System32\Sysprep\Panther\setupact.log**. Most likely, you need to use **Remove-AppxPackage** to remove a few residual applications. 
9) When the system reboots, finish the installation wizard.

## Running Optimizers ##
There are a number of optimizers available for VDI desktops. They do many of the same things, but each optimizer usually performs unique optimizations. Three useful ones are:
- [Citrix Optimizer](https://support.citrix.com/article/CTX224676)
- [VMware OS Optimization Tool](https://flings.vmware.com/vmware-os-optimization-tool)
- [JGSpiers Windows 10 Optimization Script](https://www.jgspiers.com/windows-10-1803-optimisation-script/)

I like to keep these in a .zip file on a network share along with the unattend.xml file and the following script that cleans up the default profile (again, thanks to James Rankin<sup>1</sup>).
```
takeown /f c:\users\default\appdata\local\Microsoft\WindowsApps /r /a /d Y
icacls c:\users\default\appdata\local\Microsoft\WindowsApps /grant Administrators:F /T /C /L
get-childitem C:\Users\Default\AppData\LocalLow -force -ErrorAction SilentlyContinue | foreach ($_) {remove-item $_.fullname -force -recurse -ErrorAction SilentlyContinue -confirm:$false}
get-childitem C:\Users\Default\AppData\Local\Microsoft\Windows -exclude “Shell”,”WinX” -Force -ErrorAction SilentlyContinue | foreach ($_) {remove-item $_.fullname -force -recurse -ErrorAction SilentlyContinue -confirm:$false}
get-childitem C:\Users\Default\AppData\Local\Microsoft -exclude “Windows” -Force -ErrorAction SilentlyContinue | foreach ($_) {remove-item $_.fullname -force -recurse -ErrorAction SilentlyContinue -confirm:$false}
get-childitem C:\Users\Default\AppData\Local -exclude “Microsoft” -Force -ErrorAction SilentlyContinue | foreach ($_) {remove-item $_.fullname -force -recurse -ErrorAction SilentlyContinue -confirm:$false}
get-childitem C:\Users\Default\AppData\Roaming\Microsoft\Windows -exclude “Start Menu”,”SendTo” -Force -ErrorAction SilentlyContinue | foreach ($_) {remove-item $_.fullname -force -recurse -ErrorAction SilentlyContinue -confirm:$false}
get-childitem C:\Users\Default\AppData\Roaming\Microsoft -exclude “Windows” -Force -ErrorAction SilentlyContinue | foreach ($_) {remove-item $_.fullname -force -recurse -ErrorAction SilentlyContinue -confirm:$false}
get-childitem C:\Users\Default\AppData\Roaming -exclude “Microsoft” -Force -ErrorAction SilentlyContinue | foreach ($_) {remove-item $_.fullname -force -recurse -ErrorAction SilentlyContinue -confirm:$false}
Get-ChildItem c:\users\default -Filter “*.log*” -Force -ErrorAction SilentlyContinue | Remove-Item -Force -ErrorAction SilentlyContinue
Get-ChildItem c:\users\default -Filter “*.blf*” -Force -ErrorAction SilentlyContinue | Remove-Item -Force -ErrorAction SilentlyContinue
Get-ChildItem c:\users\default -Filter “*.REGTRANS-MS” -Force -ErrorAction SilentlyContinue | Remove-Item -Force -ErrorAction SilentlyContinue
```

After running an optimizer (or all of them) and James Rankin's default profile script, you can proceed through the normal VDI image setup. 

## Further Reading ##

- James Rankin: [How to get the fastest possible Citrix logon times](https://james-rankin.com/articles/how-to-get-the-fastest-possible-citrix-logon-times/)

- JG Spiers: [Golden image/image performance](https://www.jgspiers.com/citrix-tips-tricks-tweaks-suggestions/#Golden-Image)

- Microsoft: [Recommended settings for VDI desktops](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/rds-vdi-recommendations)

- Sven Huisman: [Windows 10 in non-persistent VDI - Login speed](https://svenhuisman.com/2017/03/windows-10-in-non-persistent-vdi-login-speed-part-1/)


## Footnotes ##

[1] https://james-rankin.com/articles/creating-a-custom-default-profile-on-windows-10-v1803/

[2] https://james-rankin.com/articles/how-to-remove-uwp-apps-on-windows-10-v1803/

