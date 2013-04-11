---
layout: post
title: "Automated ClickOnce Build and Deploy using Powershell and MSBuild"
date: 2013-04-11 12:14
comments: true
categories:
- c#
- powershell
- automation
- build
- deployment
- msbuild
---
As I was nearing completion on the [WPF app I described in an earlier post]() I became focused on how to easily build and deploy it to our developer and QA team. I had decided to use [ClickOnce](http://en.wikipedia.org/wiki/ClickOnce) to facilitate easy updates, but I wanted to make it dead-simple to build and deploy the installer/updater to the network share so that anyone could easily contribute to the tool development.

At the time I'd been doing quite a bit of Powershell work and coincidentally I stumbled on the [Github post](https://github.com/blog/1271-how-we-ship-github-for-windows) on how they build the [Github for Windows application](http://windows.github.com/). In that post I saw a tantalizing screen capture of their build/deploy script output and knew at once that I must have it.

<p style="text-align:center;"><img src="/images/github-for-windows.png" height="183" width="673"/></p>

From that image I reverse engineered the steps my script needed to take, and then I had to figure out how to implement each step. It's worth noting that **ClickOnce setting manipulation and deployment is not available via scripting or MSBuild commands**. The code below includes my solution to these issues.

## Build & Deployment Script Output
Below is the output of my own build and deployment script. A couple of important notes:

* My application consisted of one .exe file and one .dll file corresponding to two projects in a single Visual Studio solution. In the example code below I've replaced my .exe project name with **Executable** and my .dll project name with **Library**. The ClickOnce settings are maintained in the Executable project file.
* **I should note that a decision I made is that the installer version should always be the same as the executable version.** For a small tool like this it makes things simpler than to version the installer independently of the application it installs.

{% codeblock %}
PS C:\dev\tools\Executable> .\Deploy.ps1
 Checking prerequisites...
 Checking out the AssemblyInfo.cs files for version increment...
 Cleaning the build directory...
 Building Executable application...
 Building ClickOnce installer...
 Deploying updates to network server...
 Comitting version increments to Perforce...
PS C:\dev\tools\Executable>
{% endcodeblock %}

The script is written to be very silent unless an error occurs, so below is a description in more detail. The script does the following:

* Checks to ensure that the current user has prerequisites installed (in this case perforce).
* Checks out the appropriate files needed to increment the version number of the DLL, executable, and ClickOnce installer.
* Cleans the build directory.
* Builds the DLL and executable.
* Retrieves the (file) version of the newly-build executable.
* Forces (hacks) the executable version into the executable's .csproj definition of the ClickOnce settings.
* Builds the installer with the new version and the just-built binaries.
* Copies all of the installer files to the appropriate network fileserver.
* Commit the changes to the AssemblyInfo and csproj files (i.e., the version changes).

## Build & Deployment Script
**Please note** that I've changed a few things about the script below:

* Perforce repository paths
* Network deployment paths
* Binary names

I've included the entire (sanitized) script below, and after it describe in greater detail the interesting parts.

{% codeblock %}
Write-Host "Tool Build/Deployment Script"

$outputPrefix = " "
$msbuild = "C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.EXE"

Write-Host $outputPrefix"Checking prerequisites..."

$p4Output = p4
if($p4Output -match "'p4' is not recognized as an internal or external command")
{
	Write-Error "Cannot find p4.exe in your PATH."
	Exit
}

Write-Host $outputPrefix"Checking out the AssemblyInfo.cs files for version increment..."
p4 edit //my/project/tool/path/Library/Properties/AssemblyInfo.cs | Out-Null
p4 edit //my/project/tool/path/Executable/Properties/AssemblyInfo.cs | Out-Null
p4 edit //my/project/tool/path/Executable/Executable.csproj | Out-Null

Write-Host $outputPrefix"Cleaning the build directory..."
Invoke-Expression "$msbuild Executable\Executable.csproj /p:Configuration=Release /p:Platform=AnyCPU /t:clean /v:quiet /nologo"

Write-Host $outputPrefix"Building Executable application..."
Invoke-Expression "$msbuild Executable\Executable.csproj /p:Configuration=Release /p:Platform=AnyCPU /t:build /v:quiet /nologo"

$newExeVersion = Get-ChildItem .\Executable\bin\Release\Executable.exe | Select-Object -ExpandProperty VersionInfo | % { $_.FileVersion }
$newLibVersion = Get-ChildItem .\Executable\bin\Release\Library.dll | Select-Object -ExpandProperty VersionInfo | % { $_.FileVersion }

Write-Host $outputPrefix"Building ClickOnce installer..."
#
# Because the ClickOnce target doesn't automatically update or sync the application version
# with the assembly version of the EXE, we need to grab the version off of the built assembly
# and update the Executable.csproj file with the new application version.
#
$ProjectXml = [xml](Get-Content Executable\Executable.csproj)
$ns = new-object Xml.XmlNamespaceManager $ProjectXml.NameTable
$ns.AddNamespace('msb', 'http://schemas.microsoft.com/developer/msbuild/2003')
$AppVersion = $ProjectXml.SelectSingleNode("//msb:Project/msb:PropertyGroup/msb:ApplicationVersion", $ns)
$AppVersion.InnerText = $newExeVersion
$TargetPath = Resolve-Path "Executable\Executable.csproj"
$ProjectXml.Save($TargetPath)

Invoke-Expression "$msbuild Executable\Executable.csproj /p:Configuration=Release /p:Platform=AnyCPU /t:publish /v:quiet /nologo"

Write-Host $outputPrefix"Deploying updates to network server..."
$LocalInstallerPath = (Resolve-Path "Executable\bin\Release\app.publish").ToString() + "\*"
$RemoteInstallerPath = "\\network\path\Executable\DesktopClient\"
Copy-Item $LocalInstallerPath $RemoteInstallerPath -Recurse -Force

Write-Host $outputPrefix"Committing version increments to Perforce..."
p4 submit -d "Updating Executable ClickOnce Installer to version $newExeVersion" //my/project/tool/path/Executable/Executable.csproj | Out-Null
p4 submit -d "Updating Library to version $newLibVersion" //my/project/tool/path/Library/Properties/AssemblyInfo.cs | Out-Null
p4 submit -d "Updating Executable to version $newExeVersion" //my/project/tool/path/Executable/Properties/AssemblyInfo.cs | Out-Null
{% endcodeblock %}

### Automated Version Increment
You may have noticed that I don't take any specific action to manage the version numbers of Executable.exe and Library.dll even though I explicitly check out the AssemblyInfo.cs files.

The [MSBuild Extension Pack](http://msbuildextensionpack.codeplex.com/) is an open-source collection of MSBuild targets that make things like version management much easier. After adding the extensions to a relative path to my projects I just needed to add the following near the bottom of `Executable.csproj`.

{% codeblock lang:xml %}
<PropertyGroup>
  <ExtensionTasksPath>..\contrib\ExtensionPack\4.0.6.0\</ExtensionTasksPath>
</PropertyGroup>
<Import Project="$(ExtensionTasksPath)MSBuild.ExtensionPack.VersionNumber.targets"
  Condition=" '$(BuildingInsideVisualStudio)'!='true' " />
<PropertyGroup Condition=" '$(BuildingInsideVisualStudio)'!='true' ">
  <AssemblyMajorVersion>1</AssemblyMajorVersion>
  <AssemblyMinorVersion>3</AssemblyMinorVersion>
  <AssemblyFileMajorVersion>1</AssemblyFileMajorVersion>
  <AssemblyFileMinorVersion>3</AssemblyFileMinorVersion>
  <AssemblyInfoSpec>Properties\AssemblyInfo.cs</AssemblyInfoSpec>
</PropertyGroup>
{% endcodeblock %}

A couple things to note here:

* The `Condition` attributes on lines 5 & 6 ensure that the version increments only occur when I run the `Deploy.ps1` script, as opposed to every time I build through the Visual Studio IDE.
* I am holding the Major and Minor versions fixed via lines 7-10, so that only the Build and Revision numbers are auto-incremented.

The above code is used **both** in `Executable.csproj` and `Library.csproj`, so that both the executable and the library have their version numbers managed. In doing this I can also change the major/minor versions of the executable and library independently.

### Propagate Exe Version to ClickOnce Installer
As I mentioned earlier, I wanted to keep the installer version the same as the executable version. The problem was that there's no way to manage the ClickOnce settings via MSBuild or other API. Lines 35-41 of the script are the, ahem, workaround that I devised.

Since we want to set the ClickOnce installer version to the same as the executable, we must first fetch the executable version:

{% codeblock %}
$newExeVersion = Get-ChildItem .\Executable\bin\Release\Executable.exe | Select-Object -ExpandProperty VersionInfo | % { $_.FileVersion }
{% endcodeblock %}

This line uses the [powerful object piping capabilities in Powershell](http://technet.microsoft.com/en-us/library/ee176927.aspx) to fetch the FileVersion property from the assembly itself.

Once we have the executable version, we must then somehow insert it into `Executable.csproj` where the ClickOnce settings are defined. For reference, the associated XML from the csproj file is:

{% codeblock lang:xml %}
<PropertyGroup>
  <ApplicationVersion>1.3.0407.01</ApplicationVersion>
</PropertyGroup>
{% endcodeblock %}

Lines 35-41 read in the csproj file as XML and extracts the `ApplicationVersion` node. It then replaces the contents of that node with the assembly version we read from the executable and saves the entire XML structure back to the csproj file.

## Summary
Through automating the build and deployment process I've learned a lot about Powershell and MSBuild and I'll definitely be improving this in the future. The great thing about this particular combination of tools is that Powershell provides the glue that holds together the powerful build automation (and logging) that MSBuild offers.

While it's unfortunate that ClickOnce has so many manual aspects to it (and I think I know why) the ease of XML manipulation and file processing from Powershell make it easy to work around ClickOnce's lack of automation.

In the future I may look at moving the install/upgrade process to the [WiX Toolset](http://www.wixtoolset.org/) as it's much more configurable and automatable. ClickOnce was really a stop-gap solution because it's for an internal tool and simple enough for my bootstrapping needs.

