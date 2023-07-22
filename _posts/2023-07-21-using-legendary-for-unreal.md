---
layout: single
author_profile: true
read_time: true
show_date: true
title: "Using Legendary for Unreal Engine Marketplace"
categories:
  - Unreal Engine
tags:
  - Unreal Engine
  - Marketplace
---

A common pitfall of working with Marketplace content for Unreal Engine is that you require an actual engine installation
for a compatible engine version in the Epic launcher to download content you've acquired from the Unreal Marketplace.
There are a few situations where this can be especially frustrating:
- You're on a source build.
- You're using Linux for your main development environment.
- You want to upgrade a third party plugin to the newest engine version.

This is where [Legendary](https://github.com/derrod/legendary) can help. Legendary is a command-line client for the
Epic Games Store which allows you to download content you've purchased (or got for free in a giveaway). This is
originally designed to enable downloading games you own on EGS to enable you to play them on Linux or macOS through
tools like WINE. However, a hidden feature of this CLI client is that it also allows you to download Unreal Engine
content, which is itself hidden behind a command line switch when listing content.

## Installing Legendary
You can grab the latest release on the [GitHub release page](https://github.com/derrod/legendary/releases/latest) where
you can download an executable for your system, or if you have [Python](https://python.org) installed  (and added to
`PATH`) then you can also install it with `pip install legendary-gl`.

The first thing you'll want to do after installing Legendary is to run the `legendary auth` command. This will open a
web browser which will prompt you to login to your Epic Games account if you've not already done so. Once this is
complete then you'll want to copy the `authorization_code` given post-login and paste it into the command prompt. From
here you should be ready to list and install your content.

## Listing content
The most basic command to list all content you own on EGS and acquired Unreal Engine content is to run
```powershell
legendary list --include-ue
```

This will give you a huge list of all the games and Unreal Engine content you own on EGS. Unfortunately, Legendary does
not provide any of its own command line switches to filter any of the results, but you can use `Select-String` in
Powershell or `grep` in Linux or macOS to filter out the results. For example entering

```powershell
legendary list --include-ue | Select-String "Lyra"
```
should provide a nice tidy list of the Lyra starter game only:

```
[cli] INFO: Logging in...
[Core] INFO: Trying to re-use existing login session...
[cli] INFO: Getting game list... (this may take a while)

 * Lyra Starter Game (App name: Lyra_5.0 | Version: 5.0.2-20293145+++UE5+Release-5.0-Windows)
 * Lyra Starter Game (App name: Lyra_5.1 | Version: 5.1.0-23058290+++UE5+Release-5.1-Windows)
 * Lyra Starter Game (App name: Lyra_5.2 | Version: 5.2.0-25360045+++UE5+Release-5.2-Windows)
```
For anything we want to download we should keep a note of the "App name" field in the results. In the case of Lyra for
UE5.2, this will be `Lyra_5.2`.

## Installing content
Now we have our app name, we're ready to install something from the Marketplace. The simplest way to get started is to
run the install command:

```powershell
legendary install Lyra_5.2
```

This will install Lyra for 5.2 inside `~/Games` (ie. `%HOMEPATH%\Games` on Windows), however there is a little bit of
control you can have over this:

```powershell
# Install to a specific directory
# This will install to C:\Unreal\Lyra_5.2
legendary install Lyra_5.2 --base-path C:\Unreal

# Force a redownload
legendary install Lyra_5.2 --force

# Verify an existing installation
legendary install Lyra_5.2 --repair
```

When content is installed, running `legendary install` against a piece of content will also download updates if they're
available.

## Downloading a subset of content
With Legendary, it's also possible to download only a subset of the content. This is useful if you're interested in one
thing (a specific asset, or one of the various plugins in Lyra) or you're low on disk space. You can see a listing of
files by trying the following in PowerShell:

```powershell
# List all the files in Lyra
legendary list-files Lyra_5.2

# List all the source code from the Lyra game module
legendary list-files Lyra_5.2 | Select-String Source/LyraGame/

# List all plugins in Lyra that aren't modular gameplay plugins
legendary list-files Lyra_5.2 | Select-String Plugins/ | Select-String "GameFeatures" -NotMatch
```

With this information, we can construct our command to install a part of the Lyra starter game:

```powershell
# Download LyraGame module source
legendary install Lyra_5.2 --prefix Source/LyraGame/

# Download Lyra plugins (excluding modular gameplay)
legendary install Lyra_5.2 --prefix Plugins/ --exclude Plugins/GameFeatures/
```

## Summary

You should now have a basic grasp of using Legendary to download Unreal Engine Marketplace content. The commands shown
work just as well for sample games as they do for plugins and assets. Feel free to reach out to me for any feedback
or anything you want clarifying.