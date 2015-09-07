---
layout: post
title: Versioning binaries with Visual Studio Online / TFS 2015 new build system
date: 2015-09-07 18:20:00 +03:00
tags: "vso tfs build versioning"
---

One central requirement for e.g. reliable bug tracking is to know which version of the application the user is using. This is easier for hosted apps like web sites but a bit harder to track with mobile apps. In both cases, a common way of solving the issue is to establish traceability between the application itself and source codes. A common way to achieve this is to have a build server compile each change set in version control and produce a build, which has a number. The build information is stored on the server so it is easy to find from what source code version the build was compiled from.

There are of course many ways to embed the build identifier into the binaries. For my problem, I chose to use the AssemblyVersion last number to indicate build number. For build processes based on traditional MSBuild, there are even custom tasks like [AssemblyInfoTask][1] to do the job. However, I had just started using the Visual Studio Online new build system so I was looking for something else than MSBuild.

After some inspection I settled on using powershell to do plain find-replace in the source files before compiling. The resulting code is shown below and has been adapted from the code in this [GitHub gist][gist]. 

{% highlight powershell %}
Param(
    [Parameter(Mandatory=$False)]
    [string]$path=$pwd,
    [Parameter(Mandatory=$False,Position=1)]
    [string]$Revision=$env:BUILD_BUILDID
)
 
function Help {
    "Sets the AssemblyVersion and AssemblyFileVersion of AssemblyInfo.cs files`n"
    ".\SetAssemblyVersion.ps1 -Revision [Revision] -path [SearchPath]`n"
    "   [Revision]     The assembly revision number to set. Revision is the last number in the 4-part version number. E.g. 4 in 1.2.3.4"
    "   [SearchPath]   The path to search for AssemblyInfo files. Defaults to current folder. `n"
}

function Update-SourceVersion
{
    Param ([string]$rev)
    
    foreach ($o in $input) 
    {
        $versionPattern = "([0-9]+)\.([0-9]+)\.([0-9]+)\.([0-9]*)"
        $replacePattern = "`$1.`$2`.`$3.$rev"

        Write-Host "Updating revision of '$($o.FullName)' to $rev"

        $assemblyVersionPattern = "AssemblyVersion\(`"$versionPattern`"\)"
        $fileVersionPattern = "AssemblyFileVersion\(`"$versionPattern`"\)"
        $assemblyVersion = "AssemblyVersion(`"$replacePattern`")";
        $fileVersion = "AssemblyFileVersion(`"$replacePattern`")";
        
        (Get-Content -encoding UTF8 $o.FullName) | ForEach-Object  { 
           % {$_ -replace $assemblyVersionPattern, $assemblyVersion } |
           % {$_ -replace $fileVersionPattern, $fileVersion }
        } | Out-File $o.FullName -encoding UTF8 -force
    }
}
function Update-AllAssemblyInfoFiles ( $rev )
{
    Write-Host "Searching '$path'"   
    Get-ChildItem -Path $path -Recurse "AssemblyInfo.cs" `
	| ? { $_.fullname -notmatch '\\packages\\' } `
	| Update-SourceVersion -rev $rev
}

# validate arguments   
if (($Revision -eq '/?') -or ($Revision -notmatch "[0-9]+")) {
    Help
    exit 1;
}
 
Update-AllAssemblyInfoFiles $Revision
{% endhighlight %}


Few main points about the code:

- Get VSO build ID from the environment variable
- Find all AssemblyInfo.cs files and the AssemblyVersion declarations in them
- Contrary to the gist I started with, this only modifies the revision part so the caller doesn't have to set (know) the remaining numbers
- Exclude files under packages folder. This is because I had some (I think ADAL) libraries with source code in them, modifying those would've been at least misleading.

One thing on what I stumbled a bit on was finding the build ID. It seemed that the powershell script can see the environment variable ($env:BUILD_BUILDID) but when I tried to use that directly in the "invoke powershell script" build task parameters, the value was not set. That has probably something to do with how they invoke the powershell scripts etc.

Bonus
----
If you want to see all environment variables in Powershell, e.g. to see what values are provided by VSO, you can use this script:

{% highlight powershell %}
write-host "======================================================"
write-host "Listing available PS variables"
get-variable
write-host ""
write-host "======================================================"
write-host "Listing environment variables"
dir env:\ | ft
{% endhighlight %}

[1]: http://www.msbuildextensionpack.com/help/4.0.11.0/?topic=html/d6c3b5e8-00d4-c826-1a73-3cfe637f3827.htm 
[gist]: https://gist.github.com/derekgates/4678882