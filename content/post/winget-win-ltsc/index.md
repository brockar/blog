+++
author = "brockar"
title = "Install winget on Windows LTSC"
description = "How to install Winget on Windows LTSC"
date = '2025-05-10'
categories = [
    "Windows"
]
tags = [
    "windows",
    "winget",
    "windows-ltsc"
]
image = "win.png"

[[links]]
title = "Reddit Post"
website = "https://www.reddit.com/r/Windows10LTSC/comments/t7hg44/guide_how_to_install_winget_on_ltsc_iot_without/"
image = "https://is1-ssl.mzstatic.com/image/thumb/Purple221/v4/27/45/6f/27456fe6-77fb-aee1-b1e5-531e146f718a/AppIcon-0-0-1x_U007epad-0-1-0-85-220.png/1200x630wa.png"

[[links]]
title = "GH Post"
website = "https://github.com/muradbu/winget-script"
image = "https://upload.wikimedia.org/wikipedia/commons/thumb/c/c2/GitHub_Invertocat_Logo.svg/1200px-GitHub_Invertocat_Logo.svg.png"

[[links]]
title = "VC++ v14"
website = "https://learn.microsoft.com/en-gb/troubleshoot/developer/visualstudio/cpp/libraries/c-runtime-packages-desktop-bridge#how-to-install-and-update-desktop-framework-packages"
image = "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRo2V5A_wcCWLnUfZja5Bhdl6TgoqeG2xTS_w&s"

[[links]]
title = "UI.Xaml.2.7"
website = "https://www.nuget.org/packages/Microsoft.UI.Xaml/"
image = "https://upload.wikimedia.org/wikipedia/commons/2/25/NuGet_project_logo.svg"

+++



## Dependencies

Open `PowerShell` as an Administrator.  
Go to Downloads  

```powershell
cd 'C:\Users\user\Downloads\'
```

### VC++ v14 Desktop Framework Package

Download [VC++ v14 Desktop Framework Package](https://learn.microsoft.com/en-gb/troubleshoot/developer/visualstudio/cpp/libraries/c-runtime-packages-desktop-bridge#how-to-install-and-update-desktop-framework-packages) for your architecture, usually x64.  
Install it.  

```powershell
Add-AppxPackage -Path "Microsoft.VCLibs.x64.14.00.Desktop.appx"
```

Or change `Microsoft.VCLibs.x64.14.00.Desktop.appx` to your package name.  

### Microsoft.UI.Xaml.2.7

Download the [Nuget package](https://www.nuget.org/packages/Microsoft.UI.Xaml/) manually by clicking "Download package" on the right hand side under the "About" section.  
Change the file extension from .nuget to .zip and open it with any archiving program.  
Within the archive navigate to `tools\AppX\x64\Release` and extract Microsoft.UI.XAML.2.7.appx.  (or change x64 for your architecture)  
Install with:  

```powershell
Add-AppxPackage -Path "Microsoft.UI.XAML.2.7.appx"  
```

## Install Winget

From the latest release download the `.msixbundle` and `.xml` files: [Winget GH](https://github.com/microsoft/winget-cli/releases).  
Install with:  

```powershell
Add-AppxProvisionedPackage -Online -PackagePath "PATH TO MSIXBUNDLE" -LicensePath "PATH TO XML" -Verbose
```

Wait for the install to finish and you're done. You may need to restart the terminal, or reboot your pc.  
Verify that the installation succeeded by running `winget` in PowerShell.  
If no errors occur then you're done!  
