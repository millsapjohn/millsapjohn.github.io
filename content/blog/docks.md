+++
title = "Flyout Docks, and CivilTools Updates"
date = "2025-06-02"
+++

# A New Plugin Has Entered the Chat
Because I'm incapable of not being Like That sometimes, I've spent some time over the last week or so developing one of the items on my plugin laundry list: flyout docks. Inspired by a popular CAD package, this plugin creates vertical buttons on the sides and bottom of the main QGIS window, one for each dock widget. Clicking the dock widget's button toggles its view state, and any other widgets that were visible in that dock area are hidden when a widget is toggled visible (in other words, if Widget A is visible and the Widget B button is clicked, Widget A is no longer visible and Widget B is now visible).

This is functionality I've wanted for a while, and working on it has given me deeper familiarity with the incredibly deep Qt API, which can only be a good thing.

# But CivilTools Wasn't Neglected
Never fear, hypothetical reader who's deeply invested in CivilTools for some reason! I've put a good amount of work into it, as well. Since my last update, I've:

- updated the crosshairs to only be visible in the map view
- worked out the selection behavior, including window select, crossing select, and the deselect versions of both as well
- refactored the MapTool code for easier extension

The last isn't sexy, but it was necessary. I've created a base MapTool from which all the others will inherit, allowing me to save a lot of repetition in the code.

The second item, working out selection behavior, was a BEAR. The ability to select single features with a click, or multiple features with either a window or a fence, or deselect in any of those three ways by holding down SHIFT...it was a lot to work through. As your resident Not Very Good Programmer, conceptualizing the logic flow for what a mouse click does or doesn't mean in any of those contexts stretched my brain a little bit. I'm honestly pretty proud that I was able to work it out, and it works pretty well!

I've also been working on snaps, although I might have ended up putting the cart before the horse a little bit. Initially my idea was to grab all the "static" snaps (i.e., vertices, midpoints, things that don't change depending on the cursor location). However, doing this in the main thread causes significant slowdown. I could put it in the background using a QgsTask (and I'm going to explore this option), but it may be worthwhile to just leave the "snap grabbing" to happen once a command is active. This will require some testing.

I've also been exploring some QGIS housekeeping, such as using Jupyter notebooks with QGIS. As with everything QGIS, the setup seems to be a little involved; but if I can figure it out it will be worthwhile.

# CivilTools Code Count
Currently sitting at a little under 1500 LOC.
