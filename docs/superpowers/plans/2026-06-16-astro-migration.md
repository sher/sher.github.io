# Astro Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Migrate sher.github.io from compiled Octopress HTML to an Astro blog using the Cactus theme, with 4 migrated posts and GitHub Actions deployment.

**Architecture:** Scaffold astro-cactus into a temp directory, copy files into the existing repo (preserving git history), configure the theme, write 4 Markdown/MDX posts, and wire up GitHub Actions to build and deploy to GitHub Pages.

**Tech Stack:** Astro, astro-cactus theme, pnpm, GitHub Actions, GitHub Pages

---

## File Map

| Action | Path | Purpose |
|--------|------|---------|
| Create | `.github/workflows/deploy.yml` | CI/CD: build and deploy to GitHub Pages |
| Create | `src/content/post/xcode-development-using-cocoapods.md` | Migrated Octopress post |
| Create | `src/content/post/core-data-automate-master-data-preloading.md` | Migrated Octopress post |
| Create | `src/content/post/zig.mdx` | Zig reference post wrapping zig.html |
| Create | `src/content/post/fdb.mdx` | FDB reference post wrapping fdb_menu.html |
| Copy to | `public/zig.html` | Zig reference HTML preserved as-is |
| Copy to | `public/fdb_menu.html` | FDB reference HTML preserved as-is |
| Copy to | `public/images/` | Existing blog images preserved |
| Copy to | `public/favicon.png` | Existing favicon preserved |
| Modify | `src/site.config.ts` | Site title, author, description |
| Modify | `astro.config.mjs` | Disable pagefind search integration |
| Delete | `blog/` | Old compiled Octopress HTML |
| Delete | `stylesheets/`, `javascripts/`, `assets/` | Old Octopress assets |
| Delete | `index.html`, `atom.xml`, `sitemap.xml`, `robots.txt`, `fdb.html`, `zig.html` (root) | Replaced by Astro output |

> **Note:** The Cactus template uses `src/content/post/` (not `blog/`) for posts. Verify this after scaffolding in Task 1.

---

### Task 1: Scaffold Cactus template and copy into repo

**Files:**
- Create: `/tmp/astro-cactus-scaffold/` (temp, not committed)
- Modify: `/Users/sher/code/sher.github.io/` (copy scaffold files here)

- [ ] **Step 1: Scaffold the Cactus template into a temp directory**

```bash
cd /tmp && pnpm create astro@latest astro-cactus-scaffold --template chrismwilson/astro-cactus --no-git --no-install --yes
```

Expected: scaffold created at `/tmp/astro-cactus-scaffold/` with `src/`, `public/`, `astro.config.mjs`, `package.json`, `tsconfig.json`.

- [ ] **Step 2: Inspect the scaffold structure**

```bash
find /tmp/astro-cactus-scaffold -maxdepth 3 -not -path '*/.git/*' | sort
```

Check: confirm posts live in `src/content/post/`. If they are in a different directory, adjust all subsequent tasks to use the correct path.

- [ ] **Step 3: Preserve existing assets before copying**

```bash
cp -r /Users/sher/code/sher.github.io/images /tmp/blog-images-backup
cp /Users/sher/code/sher.github.io/favicon.png /tmp/blog-favicon-backup.png
cp /Users/sher/code/sher.github.io/zig.html /tmp/zig-backup.html
cp /Users/sher/code/sher.github.io/fdb_menu.html /tmp/fdb-menu-backup.html
```

- [ ] **Step 4: Remove old Octopress output from the repo**

```bash
cd /Users/sher/code/sher.github.io
rm -rf blog stylesheets javascripts assets
rm -f index.html atom.xml sitemap.xml robots.txt fdb.html fdb_menu.html zig.html
```

- [ ] **Step 5: Copy scaffold files into the repo**

```bash
cp -r /tmp/astro-cactus-scaffold/. /Users/sher/code/sher.github.io/
```

This copies all files including hidden files (like `.gitignore`) but skips `.git` since the scaffold has none.

- [ ] **Step 6: Restore existing assets into public/**

```bash
cp -r /tmp/blog-images-backup /Users/sher/code/sher.github.io/public/images
cp /tmp/blog-favicon-backup.png /Users/sher/code/sher.github.io/public/favicon.png
cp /tmp/zig-backup.html /Users/sher/code/sher.github.io/public/zig.html
cp /tmp/fdb-menu-backup.html /Users/sher/code/sher.github.io/public/fdb_menu.html
```

- [ ] **Step 7: Install dependencies**

```bash
cd /Users/sher/code/sher.github.io && pnpm install
```

Expected: `node_modules/` created, no errors.

- [ ] **Step 8: Verify dev server starts**

```bash
cd /Users/sher/code/sher.github.io && pnpm dev
```

Expected: server starts at `http://localhost:4321`. Open it. You should see the default Cactus theme with example posts. Stop the server with Ctrl+C.

- [ ] **Step 9: Commit scaffold**

```bash
cd /Users/sher/code/sher.github.io
git add -A
git commit -m "Add Astro Cactus scaffold"
```

---

### Task 2: Configure site settings

**Files:**
- Modify: `src/site.config.ts`
- Modify: `astro.config.mjs`

- [ ] **Step 1: Update src/site.config.ts**

Open `src/site.config.ts`. Set these fields:

```typescript
export const siteConfig: SiteConfig = {
  author: "Sher",
  title: "The Cookbook",
  description: "Notes on software engineering.",
  lang: "en-US",
  ogLocale: "en_US",
  sortPostsByUpdatedDate: false,
  date: {
    locale: "en-US",
    options: {
      day: "numeric",
      month: "long",
      year: "numeric",
    },
  },
};
```

> The exact shape of `SiteConfig` depends on the scaffolded version. Match the existing field names in the file — do not add fields that are not already there.

- [ ] **Step 2: Disable pagefind search in astro.config.mjs**

Open `astro.config.mjs`. Find the `integrations` array. Remove the `pagefind()` entry (and its import) so search is off. Leave all other integrations untouched.

- [ ] **Step 3: Set site URL in astro.config.mjs**

In `astro.config.mjs`, set:

```js
export default defineConfig({
  site: "https://sher.github.io",
  // ... rest unchanged
});
```

- [ ] **Step 4: Verify dev server still starts cleanly**

```bash
cd /Users/sher/code/sher.github.io && pnpm dev
```

Open `http://localhost:4321`. Title in browser tab should say "The Cookbook". Stop with Ctrl+C.

- [ ] **Step 5: Commit**

```bash
cd /Users/sher/code/sher.github.io
git add src/site.config.ts astro.config.mjs
git commit -m "Configure Cactus theme for The Cookbook"
```

---

### Task 3: GitHub Actions deploy workflow

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Create the workflow file**

```bash
mkdir -p /Users/sher/code/sher.github.io/.github/workflows
```

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v4
        with:
          version: 9

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install

      - name: Build
        run: pnpm build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Enable GitHub Actions as the Pages source (manual)**

Go to `https://github.com/sher/sher.github.io/settings/pages`. Under **Source**, select **GitHub Actions** instead of **Deploy from a branch**. Save.

- [ ] **Step 3: Commit and push**

```bash
cd /Users/sher/code/sher.github.io
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions deploy workflow"
git push origin master
```

Expected: GitHub Actions run starts. Check `https://github.com/sher/sher.github.io/actions` — the build should pass (it will deploy the default Cactus example posts for now).

---

### Task 4: Migrate CocoaPods post

**Files:**
- Create: `src/content/post/xcode-development-using-cocoapods.md`

- [ ] **Step 1: Create the post file**

Create `src/content/post/xcode-development-using-cocoapods.md` with this content:

````markdown
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
# If you are using rbenv don't forget to rehash
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

Cocoapods makes it easier to manage libraries in your project. You can search for a library:

```bash
$ pod search TestFlightSDK
```

Xcode **workspaces** have advantages described in
[Xcode Concepts](http://developer.apple.com/library/ios/#featuredarticles/XcodeConcepts/Concept-Workspace.html "Xcode Concepts"),
discussed further in [Core Data: Automate master data preloading](/blog/core-data-automate-master-data-preloading/).
````

- [ ] **Step 2: Verify the post renders**

```bash
cd /Users/sher/code/sher.github.io && pnpm dev
```

Open `http://localhost:4321`. The CocoaPods post should appear in the post list. Click it and check that code blocks render with syntax highlighting. Stop with Ctrl+C.

- [ ] **Step 3: Commit**

```bash
cd /Users/sher/code/sher.github.io
git add src/content/post/xcode-development-using-cocoapods.md
git commit -m "Migrate CocoaPods post"
```

---

### Task 5: Migrate Core Data post

**Files:**
- Create: `src/content/post/core-data-automate-master-data-preloading.md`

- [ ] **Step 1: Create the post file**

Create `src/content/post/core-data-automate-master-data-preloading.md`:

````markdown
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
````

- [ ] **Step 2: Verify the post renders**

```bash
cd /Users/sher/code/sher.github.io && pnpm dev
```

Open `http://localhost:4321`. The Core Data post should appear. Click it and verify images load (they come from `/images/` in `public/images/`). Stop with Ctrl+C.

- [ ] **Step 3: Commit**

```bash
cd /Users/sher/code/sher.github.io
git add src/content/post/core-data-automate-master-data-preloading.md
git commit -m "Migrate Core Data post"
```

---

### Task 6: Add Zig reference post

**Files:**
- Create: `src/content/post/zig.mdx`
- `public/zig.html` already copied in Task 1

- [ ] **Step 1: Add @astrojs/mdx if not already in the scaffold**

Check `astro.config.mjs` for an `mdx()` integration. If it is missing:

```bash
cd /Users/sher/code/sher.github.io && pnpm astro add mdx --yes
```

- [ ] **Step 2: Create the Zig post**

Create `src/content/post/zig.mdx`:

```mdx
---
title: "Zig Language Reference Notes"
description: "Personal notes and reference while learning Zig."
publishDate: 2024-01-01
tags: ["zig"]
---

Personal Zig language notes and reference, rendered from the Zig autodoc output.

<iframe
  src="/zig.html"
  style="width:100%;height:80vh;border:none;"
  title="Zig reference"
/>
```

> Set `publishDate` to whatever date you first started these notes. 2024-01-01 is a placeholder.

- [ ] **Step 3: Verify the post renders**

```bash
cd /Users/sher/code/sher.github.io && pnpm dev
```

Open `http://localhost:4321`. The Zig post should appear. Click it. The iframe should load the Zig HTML. Stop with Ctrl+C.

- [ ] **Step 4: Commit**

```bash
cd /Users/sher/code/sher.github.io
git add src/content/post/zig.mdx
git commit -m "Add Zig reference post"
```

---

### Task 7: Add FDB reference post

**Files:**
- Create: `src/content/post/fdb.mdx`
- `public/fdb_menu.html` already copied in Task 1

- [ ] **Step 1: Create the FDB post**

Create `src/content/post/fdb.mdx`:

```mdx
---
title: "FDB Reference"
description: "Personal FDB reference notes."
publishDate: 2024-01-01
tags: ["fdb"]
---

FDB reference notes.

<iframe
  src="/fdb_menu.html"
  style="width:100%;height:80vh;border:none;"
  title="FDB reference"
/>
```

> Set `publishDate` to the actual date you started these notes.

- [ ] **Step 2: Verify the post renders**

```bash
cd /Users/sher/code/sher.github.io && pnpm dev
```

Open `http://localhost:4321`. The FDB post should appear. Click it and verify the iframe loads. Stop with Ctrl+C.

- [ ] **Step 3: Commit**

```bash
cd /Users/sher/code/sher.github.io
git add src/content/post/fdb.mdx
git commit -m "Add FDB reference post"
```

---

### Task 8: Remove Cactus example posts and final verification

**Files:**
- Delete: example posts inside `src/content/post/` that came with the scaffold

- [ ] **Step 1: List and remove example posts**

```bash
ls /Users/sher/code/sher.github.io/src/content/post/
```

Remove any posts that came with the Cactus scaffold (they will have names like `blog-post-1.md`, `introducing-astro.md`, etc.) — keep only the 4 posts you created:

```bash
cd /Users/sher/code/sher.github.io/src/content/post/
# Remove scaffold example posts. Adjust filenames based on what you see above.
# Keep only:
#   core-data-automate-master-data-preloading.md
#   xcode-development-using-cocoapods.md
#   zig.mdx
#   fdb.mdx
```

- [ ] **Step 2: Run a full production build**

```bash
cd /Users/sher/code/sher.github.io && pnpm build
```

Expected: exits with code 0, `dist/` directory created.

- [ ] **Step 3: Preview the production build**

```bash
cd /Users/sher/code/sher.github.io && pnpm preview
```

Open `http://localhost:4321`. Verify:
- Site title is "The Cookbook"
- 4 posts are listed
- Each post page loads without errors
- Images render in the old blog posts
- iframes load in the Zig and FDB posts
- RSS feed is available at `/rss.xml` or `/feed.xml` (check which URL the theme uses)

Stop with Ctrl+C.

- [ ] **Step 4: Commit and push**

```bash
cd /Users/sher/code/sher.github.io
git add -A
git commit -m "Remove scaffold example posts, ready for deploy"
git push origin master
```

- [ ] **Step 5: Verify deploy**

Wait ~2 minutes, then open `https://sher.github.io`. The live site should match the local preview. Check `https://github.com/sher/sher.github.io/actions` if the deploy is still running.
