---
layout: post
title: "Passing Arguments to UnrealBuildTool"
categories: ["Unreal Engine", "DevOps"]
---

When compiling your packaged game in Unreal Engine, you may want to pass command line arguments that can be consumed by your Target.cs file. This can be useful for many applications, such as differenciating builds for PC-specific storefronts
(such as Steam and EGS builds), as well as toggling functionality between beta or demo builds.

<!--more-->

UnrealBuildTool (UBT) has out-of-the-box support for parsing and applying command line parameters that are passed to it through a simple attribute that can be placed on top of your build variables.

## CommandLineAttribute
The magic of this is marking up variables in your TargetRules class (ie. your Target.cs) with CommandLine attributes. These atttributes are iterated over by UnrealBuildTool, and if the command line attribute is specified then the value will be set to true, the simplest example is:

```csharp
[CommandLine("-BetaBuild")]
protected bool IsBetaBuild = false;
```

With the above, if `-BetaBuild` is passed to UBT, then `IsBetaBuild` will be set to true when running through your TargetRules class. You can use this to set preprocessor values, or define extra modules to build, for example:

```csharp
if (IsBetaBuild)
{
    ProjectDefinitions.Add("IS_BETA_BUILD=1");
}
else
{
    ProjectDefinitions.Add("IS_BETA_BUILD=0");
}
```

## Advanced Usage
In addition to opting into specific variables as an on switch, you can actual capture values. For example, if you wanted to capture `-Storefront=EGS` or `-Storefront=Steam` then you could use `CommandLineAttribute` like the following: 

```csharp
[CommandLine("-Storefront=", Required = true)]
protected string Storefront;
```

`Required` being set to true will require the command line argument to be specified for each build.

You can also create opt-out command line parameters by using a `Value` argument in the attribute:

```csharp
[CommandLine("-DisableFeatureA", Value = "false")]
protected bool EnableFeatureA = true;
```

## How do I pass the argument?

The common misconception is that you pass this to your build as a `BuildCookRun` argument, but the default behaviour is that it's expected as an UnrealBuildTool argument. In order for UAT to pass to argument to UBT, then you should use the
`ubtargs` command line parameter (eg. `-ubtargs="-BetaBuild"`). In addition, you must also make sure `-build` is specified as a parameter.

If you are struggling to get your command line argument to detect (which maybe the case for a binary build), then it maybe necessary to add the following to the constructor of your TargetRules class:

```csharp
CommandLine.ParseArguments(Environment.GetCommandLineArgs(), this);
```