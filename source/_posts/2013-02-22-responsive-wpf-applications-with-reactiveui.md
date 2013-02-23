---
layout: post
title: "Responsive WPF applications with ReactiveUI"
date: 2013-02-22 21:25
comments: true
categories:
- xaml
- reactiveui
- mahapps.metro
- c#
- wpf
---
Learn about my newfound love for the [ReactiveUI MVVM framework](http://reactiveui.net/).

My day job is to wrangle 12 million lines of 20+ year-old C and C++ using a custom Win32-based UI library that was built in the mid-90s and never fundamentally improved. It does what it was designed to do really well (bind transactions to views, zoom between list items, the transactions they compose, and the reports on those transactions) but sometimes I idly wonder what's been going on in bleeding edge Windows app development in the interim.

Last week I downloaded and played around with [GitHub's excellent Windows client](http://windows.github.com/). I vaguely remembered [a blog post](https://github.com/blog/1151-designing-github-for-windows) on the gear underlying the desktop app awhile back, and I was interested to see where it was at these days. You know you're a geek when you read the entire list of licenses in the About view to get a sense of the underlying technology.

After looking around at the various .NET libraries involved, and reading [some follow-up](https://github.com/blog/1127-github-for-windows) [blog posts](https://github.com/blog/1420-github-for-windows-recent-improvements) I decided to **build a front-end to a developer tool** I whipped up for our developers and QA engineers at work.

Essentially, in debug mode QuickBooks Payroll can talk to a variety of backend environments. Configuring the various endpoints involves editing several config files that are in different places depending on whether you have an installed build or a developer build. To make it easier to configure things I wrote a command-line tool that can tell you what environments you're currently configured to talk to, as well as list available environments and change the current environment. I built the command-line tool on top of a library that implemented all the core logic because I knew that I'd eventually want to build an easier-to-use GUI on top of it.

So as a little weekend project I took inspiration from GitHub for Windows Metro/Modern visual design and a desire to look further into ReactiveUI's take on multi-threaded UI.

Multi-threaded UI development typically has one sticking point; it's easy enough to define a lambda or function and set it up to run on a separate thread but one has to be very careful when moving the result of that lambda back onto the UI thread to display it. **Reactive's** secret sauce is to simplify this dangerous activity and deliver a true 'fire-and-forget' multi-thread solution.

Thus far I've found that it's very easy to chain async commands AND provide a nice responsive user experience. In cases where you have certain application behaviors that are gated upon other actions completing the ReactiveUI framework makes it extremely simple.

An example:

* App bootstraps.
* Check for environment definition updates.
* If updates are available; download and install updates.
* When updates are complete, or if no updates are available then validate current saved environment settings.
* If settings are invalid or missing show the special UI telling user to edit the app settings.
* If settings are valid then discover the current environment.
* Once the current environment has been determined; set the properties on the view model that are bound to the UI that describe the current environment.

The state machine representing the above workflow was implemented with three View Models bound to one XAML MainWindow. The cleanest one is below, and I've commented how the commands to **check for updates** and **download and install updates** are implemented (I'm still working on tightening up the other two).

I've included the entire file below because I think it's valuable to view the initialization of the commands and observers and their targets in context. This view model encapsulates all of the updater functionality and how the different states of updating are exposed to the UI.

{% codeblock lang:csharp EnvironmentsUpdaterViewModel.cs %}
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Reactive.Linq;
using System.Text;
using System.Threading.Tasks;
using Intuit.Payroll.Tools.SetPayrollEnvironment;
using ReactiveUI;
using ReactiveUI.Xaml;

namespace Intuit.Payroll.Tools.PayrollEnvironments
{
    public enum UpdateState
    {
        Uninitialized,      // No action has been taken yet
        CheckingForUpdates, // Checking the update source for possible updates
        UpdateAvailable,    // An update is available for download
        ApplyingUpdates,    // Currently downloading and installing updates
        Completed           // Used when updates have completed or when no updates are available
    }

    public class EnvironmentsUpdaterViewModel : ReactiveObject
    {
        private PayrollEnvironmentConfiguration _EnvConfig;
        private QuickBooksInformation _LocalSettings;

        public EnvironmentsUpdaterViewModel(QuickBooksInformation localInfo)
        {
            _LocalSettings = localInfo;
            Status = _LocalSettings.IsValid ? Properties.Resources.InitialUpdateMsg
                         : Properties.Resources.InvalidSettingsUpdaterMsg;

            UpdateState = UpdateState.Uninitialized;

            InitializeConfiguration();

            // Creating the EnvironmentsFileUpdater implicitly goes out to the update location
            // (likely on a local or VPN network share), so we want to make sure it's async.
            CheckForUpdates = new ReactiveAsyncCommand(null, 1);
            var updaterFuture = CheckForUpdates.RegisterAsyncFunction(_ =>
                {
                    UpdateState = UpdateState.CheckingForUpdates;
                    return new EnvironmentsFileUpdater(_EnvConfig);
                });
            _Updater = new ObservableAsPropertyHelper<EnvironmentsFileUpdater>(updaterFuture, _ => raisePropertyChanged("Updater"));

            // When the updater has been created, initialized and set then check if any updates
            // are available on the update server.
            this.ObservableForProperty(x => x.Updater).Subscribe(_ =>
                {
                    if (Updater != null)
                    {
                        UpdateState = Updater.UpdateAvailable ? UpdateState.UpdateAvailable : UpdateState.Completed;
                        Status = Updater.UpdateAvailable ? Properties.Resources.EnvironmentUpdatesAvailableMsg
                                        : string.Format(Properties.Resources.CurrentEnvironmentMessage, Updater.LatestVersion);
                        // If updates are available; download and install them
                        if (UpdateState == UpdateState.UpdateAvailable)
                        {
                            UpdateState = UpdateState.ApplyingUpdates;
                            DownloadUpdates.Execute(null);
                        }
                    }
                });

            // After we download and apply updates, update the status and mark the workflow as completed.
            DownloadUpdates = new ReactiveAsyncCommand(null, 1);
            DownloadUpdates.RegisterAsyncAction(_ =>
                {
                    if (Updater.UpdateToVersion(Updater.LatestVersion))
                    {
                        Status = string.Format(Properties.Resources.CurrentEnvironmentMessage, Updater.LatestVersion);
                    }
                    else
                    {
                        Status = Properties.Resources.EnvironmentUpdateErrorMsg;
                    }
                    UpdateState = UpdateState.Completed;
                });
        }

        private void InitializeConfiguration()
        {
            try
            {
                _EnvConfig = EnvironmentManager.GetEnvironmentConfiguration(_LocalSettings);
            }
            catch (Exception)
            {
                _EnvConfig = new PayrollEnvironmentConfiguration { Version = 0,
                                                                   UpdatesLocation = string.Empty };
            }
        }

        private ObservableAsPropertyHelper<EnvironmentsFileUpdater> _Updater;
        private EnvironmentsFileUpdater Updater
        {
            get { return _Updater.Value; }
        }

#pragma warning disable 0649
        private UpdateState _UpdateState;
#pragma warning restore 0649
        public UpdateState UpdateState
        {
            get { return _UpdateState; }
            set { this.RaiseAndSetIfChanged(value); }
        }

#pragma warning disable 0649
        private string _Status;
#pragma warning restore 0649
        public string Status
        {
            get { return _Status; }
            set { this.RaiseAndSetIfChanged(value); }
        }

        public ReactiveAsyncCommand CheckForUpdates { get; protected set; }
        public ReactiveAsyncCommand DownloadUpdates { get; protected set; }
    }
}

{% endcodeblock %}

The data that the updater object fetches from is located on an intranet share, and so it was important to check for and apply updates asynchronously (especially if, like me, you're working over the VPN).

I'm unsure at this point if it's ok that I'm updating the `UpdateState` property from within many of the lambdas. I feel that there's something risky going on there but everything seems to work fine for me as it is.

I'll break down the two main uses of Reactive's async commands below. Note that I'm using the `UpdateState` property primarily outside of this class; the window binds certain elements' visibility to the state of the updater object, but only when the state has certain values.

{% codeblock Initialize the Check for Updates Command lang:csharp %}
CheckForUpdates = new ReactiveAsyncCommand(null, 1);
var updaterFuture = CheckForUpdates.RegisterAsyncFunction(_ =>
    {
        // Set the current state
        UpdateState = UpdateState.CheckingForUpdates;
        // The network access occurs in the updater constructor; this line is
        // why we want this to be done asynchronously
        return new EnvironmentsFileUpdater(_EnvConfig);
    });
// Connect the future object to the helper object that backs the Updater property.
_Updater = new ObservableAsPropertyHelper<EnvironmentsFileUpdater>(updaterFuture, _ => raisePropertyChanged("Updater"));
{% endcodeblock %}

When the async function returns the new `EnvironmentsFileUpdater` instance and sets the backing field of the `Updater` property, I then lean on an Observable. Below you can see that I set it up such that when the `Updater` field changes we check whether an update is available and then either kick off the `DownloadUpdates` command or set the overall state to `UpdateState.Completed` to signal to the external listeners that the update process is complete.

The `Status` property is the string value that's bound to the UI that updates the user as to what's happening in the update process.

{% codeblock Initialize the Updater Observer lang:csharp %}
// When the updater has been created, initialized and set then check if any updates
// are available on the update server.
this.ObservableForProperty(x => x.Updater).Subscribe(_ =>
    {
        if (Updater != null)
        {
            UpdateState = Updater.UpdateAvailable ? UpdateState.UpdateAvailable : UpdateState.Completed;
            Status = Updater.UpdateAvailable ? Properties.Resources.EnvironmentUpdatesAvailableMsg
                            : string.Format(Properties.Resources.CurrentEnvironmentMessage, Updater.LatestVersion);
            // If updates are available; download and install them
            if (UpdateState == UpdateState.UpdateAvailable)
            {
                UpdateState = UpdateState.ApplyingUpdates;
                DownloadUpdates.Execute(null);
            }
        }
    });
{% endcodeblock %}

The `DownloadUpdates` command actually handles the work of downloading the latest environment definitions and installing them to the appropriate local storage mechanism. Again, it must reach out over the intranet and so it's best to make this action asynchronous.  Most of the code you see below is concerned with updating the UI-bound status value; line 5 is the key method.

{% codeblock Initialize the Download Updates Command lang:csharp %}
DownloadUpdates = new ReactiveAsyncCommand(null, 1);
DownloadUpdates.RegisterAsyncAction(_ =>
    {
        // Update to the latest version of the environment definitions
        if (Updater.UpdateToVersion(Updater.LatestVersion))
        {
            Status = string.Format(Properties.Resources.CurrentEnvironmentMessage, Updater.LatestVersion);
        }
        else
        {
            Status = Properties.Resources.EnvironmentUpdateErrorMsg;
        }
        // Finalize the state of the update process
        UpdateState = UpdateState.Completed;
    });
{% endcodeblock %}

I hope this has been a decent non-trivial example on how to use the ReactiveUI framework to build WPF view models; if it looks interesting have a look at the [great docs](http://reactiveui.net/welcome/pdf) that have been synthesized from [Paul Betts blog post examples](http://blog.paulbetts.org/) of different usages of the framework.