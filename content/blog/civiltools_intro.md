+++
title = "Introducing CivilTools"
+++

As I mentioned in a previous blog post, civil engineering design is currently locked into essentially one of two choices for drafting software. Licenses to these software packages are extremely expensive, and there currently aren't any free alternatives that are remotely usable (no shade to FreeCAD, which is a fantastic piece of software, but not really intended for civil site design!). One of my goals for a while has been to create a FOSS alternative, and since I'm not really up for writing a full GUI package from scratch, that means writing a plugin for something existing.

So without further ado, introducing CivilTools, the civil drafting plugin for QGIS!

Well, not exactly. The plugin is currently in its infancy, doesn't really do anything yet, and isn't available from QGIS' plugin manager. If you really want to try it out, it's available on [GitHub](https://github.com/millsapjohn/qgis_civiltools), but again, it doesn't currently do anything except populate some menus and allow you to toggle a fancy crosshairs cursor. But I'm actively working on it, making some progress, and have what I believe is a viable roadmap - more on that below. But first, a note about the project philosophy, and some existing alternatives.

## Does This Really Not Exist Elsewhere?
Great question. The truth is that there are, indeed, drafting tools available for QGIS right now. First, QGIS itself has some built-in drafting tools that work reasonably well. Second, there's actually an existing QGIS plugin called QAD that, essentially, attempts to be a full-on clone of an existing CAD package. So why am I "reinventing the wheel"?

In essence, it's a philosophical difference.

## What CivilTools Is Not

QAD is a really cool project, and the developer has done an impressive job of building up the core geometry tools that you'd find in CAD. But like I mentioned, as far as I can tell the project is intended to be a full-on clone of an existing CAD package, and I don't really want to do that - I don't want to force QGIS to be something it's not. For example, the project forces global UI changes - the cursor is permanently changed to the familiar crosshairs in all contexts within a map view, there's a permanent "command line", etc. I don't want to force QGIS to be a full-time CAD package, I just want CAD tools available when I need them.

The built-in QGIS drafting tools might be thought of as the opposite extreme - rather than trying to force QGIS to be a full-time drafting program, they're essentially bolted on as something of an afterthought, to give GIS technicians the option of tweaking the odd feature when an occasional need arises. And there's nothing wrong with that! QGIS is first and foremost intended to be GIS-focused, any CAD/drafting tools are *of course* an afterthought. 

What I want to strike is a nice middle ground. I want full-featured, robust drafting tools available when I need them, that work seamlessly. I also want them out of the way when I'm using QGIS as, you know, a *GIS* tool.

## What CivilTools Is
So with all that in mind, what I'm envisioning for CivilTools is for it to integrate fully into QGIS and to feel like a natural extension, albeit one that's hopefully much more powerful than the built-in tools, rather than a wholesale redesign that's just using an existing code base for convenience.

The most obvious way this is reflected in the plugin is that, when you fire up QGIS with CivilTools installed, nothing looks immediately different. Rather than burning down the existing UI and starting over, CivilTools will act as an *available map tool*, much like the existing Pan, Zoom, Select, etc tools. It has to be toggled on, and can of course be toggled off.

I'm also going to make an effort to reuse existing UI elements as much as possible. For example, the most popular CAD package out there is known for its "command line," which is always listening for keyboard input, displays what you type in real time, and displays any available sub-options when a command is active. While it's early in the design process, I'm currently not expecting to do this at all; rather, I'm planning on only having the text you type show as a tooltip next to the cursor, and have the sub-options for a command display on the message bar at the top of the screen. I think this will be a nice middle ground that will feel familiar to users of both QGIS and CAD.

Additionally, CivilTools will be *opinionated.* In the land of software, what this means is that a developer has strong feelings about the right and wrong way to do certain things, and the software will not be particularly flexible in allowing you to do things in the "wrong" way. What this means for CivilTools is that it will be pretty inflexible in terms of what layers it allows you to use for creating geometry. The plugin won't work until you create a GeoPackage for your project from the template it provides; after that, any geometry you create will only be created on its desired default layer. The exception will be if you create a modified default GeoPackage, but even there, you will only be able to use it if you create layers through the plugin's guidelines.

My thought in doing it this way is that by ensuring certain defaults are always available, the plugin will be able to "get out of the way" and just let you draw stuff on the screen, much more so than if it relied on the user to supply the layers. For example, in QGIS, you can only use the "create points" tool if you've selected a layer whose geometry type is points, and if that layer is currently active. Makes sense, but when you're really in the weeds of drawing something, it can get annoying to constantly be clicking different layers. By ensuring that sensible default layers are always available and always editable, the plugin will be able to work much more smoothly. If the user enters the Point command, the plugin knows to set the default points layer as active; no need for extra clicks!

## What CivilTools Will Do
Okay, enough about design philosophy. What is it actually capable of?

Right now, pretty much nothing, as mentioned above. But after ruminating on this project for years, I finally think I've got enough of the basic concepts ironed out to actually start the development work.

Eventually, my plan is for it to do literally everything popular civil CAD packages can do: in addition to drawing 2d geometry, it will be able to create and edit surface (TIN) data, create and edit surface *profiles*, work with pipe networks, and even expand QGIS' print layout capabilities to more closely resemble CAD packages.

All of that is probably years of work, if it get accomplished at all (I certainly hope it does). I can't bite off all of that in one go, so the rough framework is as follows:

- Version 1.0: 
  - interface setup
  - implement "create geometry" tools
  - implement "modify geometry" tools
- Version 2.0:
  - implement grading/TIN editing tools
  - implement dimensioning tools
  - implement analysis tools
- Version 3.0:
  - implement pipe network tools
  - implement layout tools
  - implement ETL (extract, transform, load) tools
- Version 4.0:
  - implement survey-related tools

As of this writing, I'm already well on my way to getting the overall interface set up; I actually think I can have it (mostly) done fairly soon. Getting the create and modify tools implemented will take a while just because there's so many of them, but given that the programmatic functionality is already there it should be at least somewhat straightforward.

Version 2.0 is going to be a real beast. Working with topologically correct TIN data is a notoriously finicky task, and will require some pretty beastly code. However, I'm taking solace in the fact that FreeCAD has recently solved their issues with the infamous 3d instability issue, and I'll be able to look at their code as a way forward. I anticipate creating my own 3d TIN kernel from scratch, in Rust - definitely a large undertaking.

## Conclusion
Maybe I'm delusional in even attempting this project. We'll see how things go. But for now, I'm choosing to be optimistic!

Follow along on this blog for updates when I have them. I expect to upload the first version to the QGIS plugin system in a couple of months, and hopefully have Version 1.0 out a few months after that!
