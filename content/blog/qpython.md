+++
title = "Setting Up a Dev Environment for QGIS Plugins"
+++

A large part of my development work right now is spent on QGIS plugins. This is slightly complicated for a relatively inexperienced programmer like me because, like any good Python dev, I want to use virtual environments for anything complex, and the QGIS Python libraries aren't available from a standard repository; instead, they come bundled with the QGIS install as standard .py files. I've spent a decent amount of time figuring out how to get my venv set up correctly so I can avoid conflicting dependencies, while still having autocomplete/suggestions available in my text editor. Here's what I've worked out.

## Development Tools
I'm currently using helix as my text editor - it's dead simple to get language servers running. Additionally, because I switch between Windows and Linux machines, I use Nushell as my shell. That becomes pretty important later on. I use PyLSP as my language server, a variety of tools (such as black and pyflakes) for formatting/linting, and finally, I'm currently experimenting with Poetry for packaging.

## Setting up the Virtual Environment
The first step is setting up the project. Because I have a template repository I use to kick off my projects, with things like a license file and a setup.cfg for my formatting/linting tools, I usually create the bones of the project first, then run `poetry init` to set up the pyproject.toml file. Python's `venv` module, which Poetry uses under the hood, helpfully sets up activation scripts in a variety of languages, including nu. The next step involves editing that file.

## Editing activate.nu
The pre-generated `activate.nu` script that `venv` creates is, unfortunately, somewhat broken - in an ideal world you want to have `activate` and `deactivate` available as commands for your virtual environment, but it's not fully working right now. Thankfully, the bones of the script - related to setting your Python interpreter and environment variables - work just fine. So after running `poetry init` and following the prompts, you can navigate to the newly created environment - in my case, I keep it in a subdirectory in my main project repo - and edit the script located at `Scripts/activate.nu`. 

The last line of the script is where the issue lies; it looks like this:

  export alias deactivate = overlay hide deactivate

If you try to use this script, the nu interpreter will complain that the `overlay` keyword can't be used in a pipeline; so I simply comment it out. But what does that mean?

## Overlays in Nushell
In Nushell, overlays are pretty directly analogous to Python virtual environments; they're collections of code and environment variables that are only active for as long as the overlay is active. The `activate.nu` script is intended to set up an overlay as a wrapper around the Python `venv`. Overlays are activated by the command `overlay use nu_file.nu` and deactivated with `overlay hide overlay_name`. The last line in the file, discussed above, is trying to create a command alias that will tell Nushell to hide the overlay; unfortunately, as the error message will tell you, the Nushell interpreter currently doesn't let you use the keyword `overlay` in an alias. This is an annoyance - and something I plan on looking into at some point - but for now, I just comment out that line and use the overlay the "normal" way. I also make some additional edits:

## Adding Environment Variables to activate.nu
PyLSP can resolve any Python modules that are currently in the `PYTHONPATH` environment variable - so a logical step would be to just add the location of QGIS' special packages to `PYTHONPATH`. However, this causes issues because there are a LOT of packages in that install; when I originally tried this method, I ran into conflicts almost immediately. The method I've come up with is to set those environment variables within `activate.nu`.

Nushell files allow you to set environment variables with the `load-env` keyword. So somewhere in `activate.nu`, I define a variable with the list of paths I need. Once again, I've found that this requires some special attention, as it seems PyLSP only recognizes these paths when they're stored the way Windows natively stores them: as a single string, separated by semicolons. For simplicity's sake and to avoid retyping, I saved the string in my .config directory as a plain text file. Then I add this to `activate.nu`:

  load-env {PYTHONPATH: (open ($nu.default-config-dir | path join "qpath.txt"))}

Now, activate the environment, from the root of my repo, I just run 

  overlay use .venv/Scripts/activate.nu

and voila! Not only is the Python virtual environment working, but I have my QGIS modules available for PyLSP.

## QGIS Python Paths
For reference, the paths you need are, on Windows, all located in the Program Files directory, under the version name of your QGIS install - in my case, at the time of writing, "C:\\Program Files\\QGIS 3.36.0". Within that directory, you need to add:

- apps\\qgis\\Python
- apps\\qgis\\plugins
- apps\\qt5\\plugins
- apps\\Python39\\Lib\\site-packages

Note that the version number for the last one will no doubt change as time goes on.

## Limitations
As mentioned before, the annoying thing right now is that the script doesn't allow you to use the familiar `activate` and `deactivate` commands. I plan on looking into this at some point and seeing if I can work out a solution.

## Conclusion
This procedure is handy, not only for making sure your LSP completions work, but also for making sure you're working with whatever packages QGIS has installed. I couldn't find this info anywhere else on the web, so I wanted to write it down - not just for other users, but also for myself when I inevitably forget somewhere down the line!
