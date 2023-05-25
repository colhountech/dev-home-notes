# Notes on Microsoft Dev Home

Dev Home is Developer Experience (DX) Dashboard that helps remove the onboarding friction for Developers building on Windows 11.

It's a DX control center to help you get to the innter loop faster so you can experience the joy of designing, building, testing and deploying apps that make you happy.

It's a way of reducing the friction of the outer loop, by giving developers the tools, configuration and workflows to get out of the outer lopp and into the innerloop.

![the Inner Loop]()

Every Developer know the friction of starting on a new project or a new client and the challenges of getting your dev box working setup. Dev Home aims to fix this with a series of Windows based innovations that have been in the works for over 5 years. 

Innovations such as:

* Running populur Linux Distructions as a first class citizen of Windows, fully integgrated and supported by Microsoft - WSL
* Ability to auto update these Linux Distrution with latest security patches and updates via the Windows Store
* Giving developer a modern Terminal Experience so you can spend more time in the console writig code and less time clicking on buttons
* Integrating Developer Identity as first class Citizend into Windows, to give accesse to GitHub and Azure Devops, etc.
* Proving a modern fast Filesystem (ReisFS) that is ideally suited to many small fast writes as is required for speeing up build and compilation
* Providing extensible plugins to developer for building their own dashboards
* Providing fully supported Desired State Configuration (DSC) tooling  such WinGet and support for other tools such as chocoately
* providing a protocol for installing and setting up the stack for builing complex application via .vsconfig to get the right apps and tools configured the right way on your Dev Box quickly
* Making all of this as open as possible (Open Source)

Dev Home started as a tweet Scott Hanselman in 2018, and has been described as the challenge of Yak Shaving - Doing all the things you need to do before you do the things that you need to do

![The Origional Tweet]()

At it's core, Dev Home is built on an Out-of-Process COM Server Host that allows safe, fast and deep integration with Windows (e.g. Windows Credential Manager for GitHub ) while giving developer a lightweight was to extend functionality via Developer Plugins.

# Where are the Links

* The Dev Home Repo : https://github.com/microsoft/devhome
* Dev Home Docs : https://learn.microsoft.com/en-us/windows/dev-home/
  - Includes the Link to Download Dev Home (Preview) from the Windows Stoage
* Dev Home Extension Repo: https://github.com/microsoft/devhome/blob/main/docs/extensions.md
* Dev Home Extension Docs: https://learn.microsoft.com/en-us/windows/dev-home/extensions#dev-home-github-extension
* Writing Extensiosn for Dev Home: https://github.com/microsoft/devhome/blob/main/docs/extensions.md
* Dev Home Tools Docs (WinUI3 Class Library) https://github.com/microsoft/devhome/blob/main/docs/tools.md

# What's inside the box

Dev Home has a modular architecture that consists of multiple components. These components are:

* Dev Home Core - the control centre responsible for managing and defining package locations and configuraiton
* Dev Home Common - shared functionality across extensions and core, including telemetry and logging
* Settings - a single place for all your settings and permissions
* Tools - tools that extend the capability of Dev Home by consuming the functionality of the dev home extensions. E.g. Dashboard and Setup Flow
* Extensions - separate packages that live out of process and 


# Writing Extensions

Dev Home currently exposes interfaces that can be extended via the Extension SDK.

Each extension lives as a separate, packaged Windows application and manages its own lifecycle and is defined via the `package.appxmanifest` Manifest file.

These applictions define a `COM Server` for the extension and also `Application Extension Properties` which defines which interfaces the extension supports.

(Who could have prediced that learning about COM Servers in 2003 would turn out to be useful in 2023 - thank God for seniority! :-) 

> For those uninitiated, here is a quick history lesson: A COM Server is like a 2000 era web server, without the web bit, that involved careful referenec counting or you would get memory leaks. It's how most internal windows components communicate internally.e.g if you want to control Powerpoint from another app.

## Dev Home Extension Interfaces

each plugin exposes a IPlugin Interface, which Dev Home creates an instace of, based on the Guid defined in the manifest.

1. Developer IDs: Allow developers to sign in and out of a service by implementing the IDeveloperIdProvider interface.
1. Repositories: Allow developers to get available repositories associated with their Developer IDs or parse repositories from URLs and clone them by implementing the IRepositoryProvider interface.


### Developer ID Scenarios

Firstly the manifest needs to declaring `IDeveloperId` as one of the supported interfaces

Secondly, the extension needs to implement the `IDeveloperIdProviderInterface`

The `IDeveloperIdProviderInterface` defines a series of entry points for events such as `LoggedIn`, `LoggedOut`, and `Update`, as well as returning the Name and Logged in Developer Ids.
You also need to implement a logout for a developer account via it's developerId with this interface.

The `IDeveloperId` must implement the `LoginId()` and `Url()` endpoints

### Repositories

Firstly the manifest needs to declaring `IRepository` as one of the supported interfaces

Secondly, the extension needs to implement the `IRepositoryProvider`

The `IRepositoryProvider` defines a series of entry points for returning the list of Repositories for a logged in Developer, and a helper method to parse the repository from the Repo URL.

The `IRepository` exposes methods to either  clone public repos or private repos under the logged in developer IDs, and expose some other details such as whether the repo is private and the owner name of the repo.


## How Extensions Work 

See extensions currently as a way of extening git repos into your dev box in a more integrated way.

This is the basic workflow of an extension.

1. A plugin is registed with Dev Home
1. Dev Home lists all extensions
1. Each plugin extension is registered
1. Dev Home holds a reference to the IPlugin instance
1. Dev Home requests a Provider from the plugin for a specific plugin interface, e.g. `IDeveloperId` or `IRepository`
1. Dev Home calls methods on the Provider e.g. calls implementation methods of the `IRepositoryProvider` or `IDeveloperIdProviderInterface` interfaces
1. When Dev Home quits, it calls `SignalIDispose` to tell the extension to close and release resources **This will cause memory leaks if not done correctly**

Dev Home maintains this list of extensions, and any internal tool can access  an extension via the `App.GetSevice().GetExtension()` methods

## Writing Tools

Tools are WinUI3 class libraries. Each tool has associated with it a Page within the Dev Home navitaion view. Tools are internal to Dev Home at this time. Tools use extensions for data and functionality.

1. Create a WinUI3 Class library within the following structure of the devhome source:

Folder structure:

```txt

+--common
   +-- DevHome.Common
+--tools
    +--mytool
       +--src
          +--Strings
             +--en-us
                +--Resources.resw
          +--myTool Project
       +--test
       +--uitest

```

1. Next Setup your references.

1. Next, Create XAML view and viewmodel. inherit from ToolPage.


Build and Test.

[more details here](https://github.com/microsoft/devhome/blob/main/docs/tools.md)


That's it for today. I'll update this as I learn more.

If you fould this useful, [message me on twitter](https://twitter.com/colhountech)

Micheal.

















