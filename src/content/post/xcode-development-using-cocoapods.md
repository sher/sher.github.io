---
title: "Xcode development using Cocoapods"
description: "Manage 3rd party libraries in Objective-C projects using Cocoapods."
publishDate: 2013-05-06
tags: ["xcode", "cocoapods", "objective-c"]
---

### Problem

Manage 3rd party libraries in Objective-C projects.

### Solution

Use [Cocoapods](http://cocoapods.org "Cocoapods") for library management in Objective-C projects.

#### Step 1

Install **cocoapods** gem and initialize.

```bash
$ gem install cocoapods
# If you are using rbenv do not forget to rehash
$ rbenv rehash
$ pod setup
```

#### Step 2

Create a Podfile and list dependencies.

```bash
$ cd ~/foo    # your xcode project root folder
$ vim Podfile
```

```ruby
# ~/foo/Podfile
platform :ios, '5.0'            # deployment target SDK
pod 'AFNetworking', '~> 1.2.0'  # means we need AFNetworking version 1.2.0 or higher
```

#### Step 3

Now install dependencies.

```bash
$ pod install
Analyzing dependencies
Downloading dependencies
Installing AFNetworking (1.2.1)
Generating Pods project
Integrating client project

[!] From now on use `foo.xcworkspace`.
```

Cocoapods creates an Xcode workspace and **Pods** static library project, then links it with your project.
All your dependencies will be available through that library.
As the output suggests, use **foo.xcworkspace** from now on.

```bash
$ open foo.xcworkspace
```

### Discussion

Cocoapods certainly makes it easier to manage libraries in your project. You can search the library you want to use.

```bash
$ pod search TestFlightSDK
```

As well as Xcode **workspaces** have some advantages described in [Xcode Concepts](http://developer.apple.com/library/ios/#featuredarticles/XcodeConcepts/Concept-Workspace.html "Xcode Concepts") which we discuss in [Core Data: Automate master data preloading](/posts/core-data-automate-master-data-preloading/).
