---
title: "Core Data: Automate master data preloading"
description: "How to automate SQLite database preloading for iOS Core Data applications."
publishDate: 2013-05-06
tags: ["core-data", "objective-c", "xcode"]
---

### Problem

Given a generic iOS application 'Blog' with data model as below:

![data model](/images/blog-data-model.png)

We want to prepopulate data for 'Categories' table in our SQLite database.

### Solution

This solution uses the approach described in
[Core Data on iOS 5 Tutorial: How To Preload and Import Existing Data](http://www.raywenderlich.com/12170/core-data-tutorial-how-to-preloadimport-existing-data-updated).
We will create an OS X Command Line Tool application that references the data model in our 'Blog'
application and prepopulates the data. We automate the building process.

### Step 1

Make sure you are using **Xcode Workspace** with your project and skip to **Step 2**.
If not, follow the steps below to create a new workspace:

1. Open your project in Xcode
2. Goto File > New > Workspace... or use the shortcut Command + Control + N
3. Name the **new workspace** the same as your project and save it in your project folder.
4. ![create workspace](/images/blog-create-workspace.png)
5. Quit Xcode
6. Open the Workspace: `$ open Blog.xcworkspace`
7. Drag your project from Finder to the Workspace
8. ![drag project to workspace](/images/blog-drag-project-to-workspace.png)
9. You have successfully added your project into a workspace.

### Step 2

1. While your workspace is open, create a new project by selecting **File > New > Project**
2. From the prompt window select **OS X > Application > Command Line Tool** and click **Next**
3. ![create commandline app](/images/blog-create-commandline-app.png)
4. Enter **MasterDataLoader** for the **Product Name**, select **Core Data** in the **Type** pulldown and click **Next**
5. ![commandline app name](/images/blog-commandline-app-name.png)
6. Select the folder where your workspace is stored, select your workspace for **Add to** and **Group** pulldowns. Click **Create**.
7. ![commandline app save](/images/blog-commandline-app-save.png)
8. Now you have two applications in your workspace
9. ![commandline app created](/images/blog-commandline-app-created.png)
10. Remove the **MasterDataLoader.xcdatamodeld** file from the **MasterDataLoader** application and **Move to Trash**
11. Open the `main.m` file in the **MasterDataLoader** application
12. Change the lines in `managedObjectModel()` and `managedObjectContext()` from:

```objective-c
NSString *path = @"MasterDataLoader";
path = [path stringByDeletingPathExtension];
```

and

```objective-c
NSString *path = [[NSProcessInfo processInfo] arguments][0];
path = [path stringByDeletingPathExtension];
```

to:

```objective-c
NSString *path = @"Blog";
```

Now run **MasterDataLoader** application.

![commandline app run](/images/blog-commandline-app-run.png)

You should get an error `Cannot create an NSPersistentStoreCoordinator with a nil model`.
This confirms our command line app runs and cannot find a compiled data model object.

### Step 3

Add a new **Run Script** build phase to your 'Blog' application, name it **Run MasterDataLoader**
and drag it under **Compile Sources** phase.

![create runscript](/images/blog-create-runscript.png)
![runscript drag](/images/blog-runscript-drag.png)

Paste this code in the **run script** you just created and save:

```bash
echo "Copying momd to MasterDataLoader folder"
cp -R ${BUILT_PRODUCTS_DIR}/${TARGET_NAME}.app/${TARGET_NAME}.momd ${BUILT_PRODUCTS_DIR}/../${CONFIGURATION}/
cd ${BUILT_PRODUCTS_DIR}/../${CONFIGURATION}/
echo "Running MasterDataLoader"
./MasterDataLoader
echo "Moving ${TARGET_NAME}.sqlite"
mv ./${TARGET_NAME}.sqlite ${BUILT_PRODUCTS_DIR}/${TARGET_NAME}.app/
echo "MasterDataLoder finished."
```

![runscript edit](/images/blog-runscript-edit.png)

Edit the scheme for 'Blog' application. Add **MasterDataLoader** in build targets.

![scheme edit](/images/blog-scheme-edit.png)

This makes sure **MasterDataLoader** is built before the 'Blog' app.

### Step 4

Open `AppDelegate.m` in the 'Blog' application and add the following code:

```objective-c
if (![[NSFileManager defaultManager] fileExistsAtPath:[storeURL path]]) {
  NSURL *preloadURL = [NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"Blog" ofType:@"sqlite"]];
  NSError* err = nil;
  if (![[NSFileManager defaultManager] copyItemAtURL:preloadURL toURL:storeURL error:&err]) {
    NSLog(@"Oops, could copy preloaded data");
  }
}
```

under the line:

```objective-c
NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"Blog.sqlite"];
```

Here we check if the SQLite database already exists. If not, we copy the one from the main bundle to the 'Documents' folder.

Our SQLite database is empty right now. Let's fetch 'Categories' and check if the table is empty.
Add the following code to `application:didFinishLaunchingWithOptions:` just before `return YES`:

```objective-c
NSFetchRequest *request = [[NSFetchRequest alloc] init];
NSEntityDescription *entity = [NSEntityDescription entityForName:@"Category"
                                        inManagedObjectContext:self.managedObjectContext];
[request setEntity:entity];
NSError *error = nil;
NSArray *cats = [self.managedObjectContext executeFetchRequest:request error:&error];
for (Category *cat in cats) {
  NSLog(@"cat: %@", cat.name);
}
```

Do not forget to import `Category.h`:

```objective-c
#import "Category.h"
```

Run the 'Blog' application. The console should print nothing.

![run](/images/blog-run.png)

### Step 5

Let's add some `Categories`. Drag the `Category.h` file from 'Blog' application to **MasterDataLoader**.

![drag category](/images/blog-drag-category.png)

Open the `main.m` file. Import `Category.h`:

```objective-c
#import "Category.h"
```

Replace the code in `main()` method:

```objective-c
// Custom code here...
// Save the managed object context
NSError *error = nil;
if (![context save:&error]) {
    NSLog(@"Error while saving %@", ([error localizedDescription] != nil) ? [error localizedDescription] : @"Unknown Error");
    exit(1);
}
```

with:

```objective-c
NSArray *categories = @[@"objective-c", @"ruby", @"python"];
for (NSString *name in categories) {
    Category *cat = [NSEntityDescription insertNewObjectForEntityForName:@"Category"
                                                  inManagedObjectContext:managedObjectContext()];
    cat.name = name;
    NSError *error = nil;
    if (![context save:&error]) {
        NSLog(@"Error while saving %@", ([error localizedDescription] != nil) ? [error localizedDescription] : @"Unknown Error");
        exit(1);
    }
}
```

### Before running

Delete 'Blog' application from the simulator.

### Run

Run the 'Blog' application.

![run](/images/blog-run.png)

The console should print three categories:

```
2013-05-07 01:00:13.911 Blog[61827:c07] cat: objective-c
2013-05-07 01:00:13.913 Blog[61827:c07] cat: ruby
2013-05-07 01:00:13.913 Blog[61827:c07] cat: python
```

### Discussion

This way you can reference any managed object from **MasterDataLoader** app and preload the data.
