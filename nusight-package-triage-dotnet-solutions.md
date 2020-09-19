---
title: "Nuget package triage for your .net solutions & integrated to CI/CD pipelines"
date: 2020-08-01T13:45:06+06:00
image: images/blog/nuget.png
feature_image: images/blog/nuget-feature.png
author: Kevin Wang
summary: NuSight is tiny .net core console app I built to help your triage your .net solutions for the nuget packages. You can make it part of the CI/CD process, regularly perform checks against consumed nuget packages, for solutions/projects.
---

Do you often work on a .net solution with many projects?

Do you ever find Newtonsoft.Json is installed with different versions across the whole solution?

And do you find some nuget packages are 5 years old that never have been updated or even been no longer maintained and unpublished?

Or find your projects/solutions are using pre-released packages that are not very reliable for production usage?

I have been struggling with these stuff during most of my projects. It is not a difficult task to fix them up, but it definitely requires some sort of trigger to examine the above scenarios regularly from time to time.

So I am wondering if I can make it part of the CI/CD process, regularly perform checks against consumed nuget packages, for solutions/projects. With nuget.org’s public API and I chuck in some dotnet core to build a little console app to achieve sanity triage of all these installed nuget dependencies. And it works out really well for me.

## Here it comes NuSight

[NuSight](https://github.com/superwalnut/NuSight) is tiny .net core console app I built to help your triage your .net solutions for the nuget packages. It is open source and you can find the source code from [github](https://github.com/superwalnut/NuSight). Also it has been published to [nuget.org](https://www.nuget.org/packages/NuSight) as a dotnet tool
[**NuSight 1.2.2**
*This is a .net tool that analyze your solution folder, discover all your project files and diagnose all the nuget…*www.nuget.org](https://www.nuget.org/packages/NuSight)

To install it and use it, you need dotnet CLI and run this command,

    dotnet tool install --global NuSight

After successfully installed, just run nusight you should be able to all its available commands with parameters.

    Available commands are:

    list        - List nuget packages from selected solution & highlight the outdated packages
        import      - Import nuget package references of selected project.
        export      - Export nuget package references of selected project.
        remove      - remove specified nuget packages from selected solution.
        list        - List nuget packages from selected solution & highlight the outdated packages
        clone       - copy nuget packages from selected solution & install to the target project.

    help <name> - For help with one of the above commands


## How to use NuSight

### **list command**

can be used for your CI build to validate your nuget packages for the whole solution.

— help or -h to list all parameters and descriptions

    $ nusight list -h

— solution or -s to specify the selected solution path, if you haven’t specified any solution path, it will take current directory as selected path

    $ nusight list -s=<YOUR_PATH>

— inconsistency or -i to triage inconsistent nuget package versions installed in the selected solution

    $ nusight list -s=<YOUR_PATH> -i

— outdated or -o to triage outdated nuget packages installed in the selected solution

    $ nusight list -s=<YOUR_PATH> -o

— prereleased or -p to triage if any pre-released packages are installed in the selected solution

    $ nusight list -s=<YOUR_PATH> -p

— unpublished or -u to triage if any unpublished packages are installed in the selected solution.

    $ nusight list -s=<YOUR_PATH> -u

To use them all together, you can do

    $ nusight list -s=<YOUR_PATH> -i -o -p -u

If the triage finds any above errors, it will return with an error code, and your pipeline will fail. Otherwise return with a 0 code and pipeline will succeed.

### Use it with github actions

<iframe src="https://medium.com/media/38625db800b41e53be4a83bf5cd56283" frameborder=0></iframe>

### Use it with gitlab CI configuration

<iframe src="https://medium.com/media/0d4bf30cb7a2ad7748e7c06c681d05d5" frameborder=0></iframe>

### bulk update command

This command can be used for bulk updating packages in multiple projects under a solution folder.

— help or -h to list all parameters and descriptions

    $ nusight update -h

- — solution or -s to specify the selected solution path, if you haven’t specified any solution path, it will take current directory as selected path

    $ nusight update -s=<YOUR_PATH>

- — display or -d allows you only print ‘dotnet update’ commands on the screen, instead of executing it. If you don’t specify this, update commands will be run and update packages in the selected solution.

    $ nusight update -s=<YOUR_PATH> -d

### Bulk remove command

This can be used for bulk removing packages in multiple projects under a solution folder.

- — help or -h to list all parameters and descriptions

    $ nusight remove -h

- — solution or -s to specify the selected solution path, if you haven’t specified any solution path, it will take current directory as selected path

    $ nusight remove -s=<YOUR_PATH>

- — package or -p to specify the package name to be removed, if you want to select multiple packages, use comma separated values

    $ nusight remove -s=<YOUR_PATH> -p=Serilog

- — display or -d allows you only print ‘dotnet remove’ commands on the screen, instead of executing it. If you don’t specify this, remove commands will be run and remove packages in the selected solution.

    $ nusight remove -s=<YOUR_PATH> -p=Serilog -d

### Clone command

This can be used for cloning all nuget packages from a source solution/project to a target project, this is handy if you have a template project and its nuget packages will always be used for any future projects, you can always clone your them to the new projects.

- — help or -h to list all parameters and descriptions

    $ nusight clone -h

- — source or -s to specify the source solution path

    $ nusight clone -s=<YOUR_PATH>

- — target or -t to specify the target project path, it must be a csproj

    $ nusight clone -s=<YOUR_PATH> -t=<PROJECT>.csproj

- — display or -d allows you only print ‘dotnet install’ commands on the screen, instead of executing it. If you don’t specify this, install commands will be run and install packages in the target project.

    $ nusight clone -s=<YOUR_PATH> -t=<PROJECT>.csproj -d

- — latest or -l to specify if you want to copy nuget packages from source solution with latest version.

    $ nusight clone -s=<YOUR_PATH> -t=<PROJECT>.csproj -l

### Export command

This can be used for exporting nuget package reference as json file.

- — help or -h to list all parameters and descriptions

    $ nusight export -h

- — source or -s to specify the source solution path

    $ nusight export -s=<YOUR_PATH>

- — file or -f to specify the exported file path and name

    $ nusight export -s=<YOUR_PATH> -f=<FILE_NAME>.json

### Import command

This can be used for importing nuget packages from an exported json file.

- — help or -h to list all parameters and descriptions

    $ nusight import -h

- — solution or -s to specify the selected solution path to be imported to.

    $ nusight import -s=<YOUR_PATH>

- — file or -f to specify the exported file path and name

    $ nusight import -s=<YOUR_PATH> -f=<FILE_NAME>.json

- — display or -d allows you only print ‘dotnet install’ commands on the screen, instead of executing it. If you don’t specify this, install commands will be run and install packages in the target project.

    $ nusight import -s=<YOUR_PATH> -f=<FILE_NAME>.json -d

## Finally,

Congratulations on setup your NuSight & I hope you enjoy using it to triage your .net solutions.

If you find my article is helpful, please click **Applaud** to support me.

![](https://cdn-images-1.medium.com/max/2000/0*Zmw2TEzK0e9KOq0_.png)

Here is the link to GitHub to grab the complete source code for the app.
[**superwalnut/NuSight**
*This is a .net tool that analyze your solution folder, discover all your project files and diagnose all the nuget…*github.com](https://github.com/superwalnut/NuSight)
