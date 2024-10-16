+++
title = "A Better QGIS History Plugin"
date = "2024-10-14"
+++

# Processing History, Improved
I guess I've caught the QGIS plugin bug. While I've continued to make progress on CivilTools (I'll post an update here soon), I've also been working on some other projects - in particular, I just released the "Param History" plugin!

Often when I'm working in GIS, I find myself in situations where I need to run an algorithm several times, iterating the input parameters until I get results that make sense. Not a problem in itself, until I get a few steps down whatever process I'm in and realize that, actually, those results weren't quite what I wanted. By that point though I've forgotten what parameters I'd actually used! QGIS does save your processing history, of course, but it's not the most human-readable format.

Enter the Param History plugin. It creates a panel in the main window displaying your processing history, but in a clearer, easier-to-read format. Additionally, you can click on a row in the history table and the plugin will automatically re-launch the algorithm window with the parameters pre-loaded, allowing you to easily pick up where you left off. There's an option to copy the (formatted) history to your clipboard if you want to store that information somewhere, and two search options (keyword, filter by timestamp). 

The biggest weakness of the plugin is that, as of now, it can only detect changes that happen to the map canvas. In other words, it can't detect failed algorithm runs, and it can't detect if you run an algorithm and choose not to add the results to the map. The first, in particular, is annoying - I need to keep looking into whether there's a signal somewhere that I'm missing in the docs that I can connect to. But overall, it works fairly well! In particular, I'm going to find this incredibly useful for doing raster operations like reclassifying by a table - having to re-enter a long table every time I wanted to change something was tedious.

I have a laundry list of other plugins I want to create - while I do plan on continuing to focus most of my efforts on CivilTools, I'll probably keep doing what I did this week and periodically take breaks to work on other ideas. These include:

- custom menus (there's an existing plugin that does this, but it doesn't work well)
- docks that are minimized by default, and expand on hover (a la Civil3d)
- an upgraded browser panel, that allows more flexible organization, like subfolders
- an integrated basic calculator
- integrated resource browsers, like
  - the NOAA WCT (Weather & Climate Toolkit)
  - TNRIS resources
  - SSURGO
- an automated HMR51 PMP (Probable Maximum Precipitation) calculator
- implementing an anisotropic inverse distance weighted interpolation algorithm
- finding high and low points along a line
- semi-automated bankline finder

and many more. Here's to the future of my kludgy, barely functional plugins!
