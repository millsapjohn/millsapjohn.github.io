+++
title = "Setting Up a Dev Environment for QGIS Plugins: WSL Edition"
date = "2024-10-08"
+++

A quick update to my previous post, on getting LSP completions working with QGIS modules - specifically as it relates to working in Linux/WSL.

I've discovered from trial and error that PyLSP interprets the `$PYTHONPATH` variable differently depending on the platform. As mentioned in my previous post, when running on Windows it expects a series of paths separated by semicolons, as that's the way Windows stores its environment variables. On Linux/WSL, however, it expects a series of paths **prepended by a colon.** For example:

    :/mnt/c/path:/mnt/c/otherpath

A simple change to make in the "qpath.txt" file I use for storing the QGIS paths. Once this change is made, PyLSP works just fine again.
