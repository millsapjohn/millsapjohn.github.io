+++
title = "Using the Project Setup Plugin and Macros to Expand QGIS Functionality"
date = "2024-05-09"
+++

If you couldn't already tell, I'm a big fan of QGIS. Ideologically, I love supporting open-source projects whenever possible; pragmatically, I've found that QGIS performance over a VPN (a common scenario in an era of work from home and hybrid schedules) is **vastly** superior to ArcGIS Pro.

One of the few areas that I feel ArcGIS Pro has an advantage is in managing projects. I love that when a new project is created, you have the option of creating a default geodatabase for the project, and setting it as the default save location for any new layers created in the project.

Another minor annoyance I find with QGIS is that connections (to databases, REST servers, etc) are global for a QGIS profile, rather than set per-project.

Like a lot of users, I find myself using the Python console constantly; having it open automatically when I open a project is a nice quality of life improvement.

And finally, when creating print layouts, I like to use automatic text with project variables, rather than dumb text, whenever possible.

In order to address all these issues, I've created the [Project Setup Plugin](https://github.com/millsapjohn/qgis_project_setup) and used the QGIS Project Macros feature. Here's how.

## First Step: Setting up Version Control
As evidenced by the GitHub link above, I like to use git for my coding projects whenever possible. For relatively simple code like project macros, it's not strictly necessary, but it makes it much easier to keep my workflow synced across my multiple machines.

Theoretically, QGIS macros are saved within the project itself, but getting them to reference external code is relatively simple; here's how I do it.

First, I define a global QGIS variable referencing the location of my `personal_macros.py` file containing my macro code. Again, since I use both Linux and Windows, I have separate paths for each, defined as `win_macro_path` and `linux_macro_path`.

Then, I edit the macro code in the project itself (to be completely accurate, within my template project, so that any new projects defined from that template inherit the code):

    import os
    from qgis.core import QgsExpressionContextUtils
    import sys

    curr_sys = QgsExpressionContextUtils.globalScope().variable('qgis_os_name')
    if curr_sys == 'windows':
      macro_path = QgsExpressionContextUtils.globalScope().variable('win_macro_path')
    else:
      macro_path = QgsExpressionContextUtils.globalScope().variable('linux_macro_path')

    sys.path.append(macro_path)

    from personal_macros import personal_openProject, personal_saveProject, personal_closeProject

    def openProject():
      personal_openProject()

    def saveProject():
      personal_saveProject()

    def closeProject():
      personal_saveProject()

Now, the in-project macros simply look for the `personal_macros.py` file at the location specified and run whatever code is found there.

## Macro Code
As of this writing, I haven't found a compelling use for the `saveProject` and `closeProject` macros (within my own personal workflow), so all that's in those macros is a call to `pass`. The real action is in the `openProject` macro, which does two things: launch the Python console if it's not already open, and updates the project connections.

### Launching the Python Console
Here's the code to launch the Python console:

    import console
    from qgis.utils import iface
    from qgis.PyQt.QtWidgets import QDockWidget

    pythonConsole = iface.mainWindow().findChild(QDockWidget, 'PythonConsole')
    if not pythonConsole or not pythonConsole.isVisible():
      iface.actionShowPythonDialog().trigger()

Fairly simple!

Note that, as this is defined in a project macro, the Python console is launched when a project is opened, rather than on QGIS startup. This is intentional on my part, as I wanted to keep QGIS start time as quick as possible; if you wanted to ensure the console is visible at launch, you could easily add this logic to your `startup.py` file - but that discussion is outside the scope of this post.

On to the next part, checking GeoPackage connections.

### Loading/Unloading GeoPackage Connections
The other part of my macro code ensures that each project has a defined list of GeoPackage connections associated with it, and that only those connections are shown in the project browser. This prevents you from seeing the GeoPackages you create for all of your projects, and just helps keep the interface clean.

    from qgis.core import QgsExpressionContextUtils, QgsProviderRegistry, QgsProject
    from qgis.utils import iface

    project = QgsProject.instance()
    if QgsExpressionContextUtils.projectScope(project).variable('project_gpkg_connections'):
      proj_conn = QgsExpressionContextUtils.projectScope(project).variable('project_gpkg_connections').split(';')
      proj_uri = proj_conn[1::2]
      proj_conn = proj_conn[0::2]
      md = QgsProviderRegistry.instance().providerMetadata('ogr')
      if md.connections():
        for item in md.connections():
          if item not in proj_conn:
            md.deleteConnection(item)
      for i in range(len(proj_conn) - 1):
        if proj_conn[i] not in md.connections():
          conn = md.createConnection(proj_uri[i])
          md.saveConnection(conn, proj_conn[i])
    iface.reloadConnections()

A few things to note here in the code. First, note that the code is referencing a project variable called `project_gpkg_connections`. This is a variable set by the Project Setup plugin, listing the GeoPackage connections you want associated with the project. The plugin allows you to create a blank GeoPackage, import a template GeoPackage, and/or reference an existing GeoPackage. All of those options will be referenced in the project variable; if at any point you want to add additional GeoPackages to your "persistent connections" list, there is a menu item available to do so.

Another thing worth pointing out is how the project variable is handled. You'll note the call to the `split(';')` method; the project variable is currently saved as a string, with each connection name and URI separated by a semicolon. This string is split at the semicolon, and the resulting list divided into names and URIs with list slices (`proj_conn[1::2]`, `proj_conn[0::2]`). This certainly isn't my preferred way to interact with this data, but this implementation is currently limited by a bug in QGIS. Project variables are supposed to allow any type of value, including Python lists or dictionaries, but as of this writing, attempting to save a list or dictionary to a project variable fails. For the time being I'm sticking with the hacky string manipulation approach, but I'm keeping my eye on the GitHub issue and will update my code when the issue is resolved.

Finally, note that this logic only affects the GeoPackage connections associated with the project. The reason I didn't include other types of connection, like REST servers, is that connecting to those providers is slightly less seamless, just in the fact that you have to hunt down a URI somewhere on the internet, rather than just pointing to a file in your filesystem; for that reason, I find it annoying to dig up the connection I want on the fly, and just prefer to keep all my REST server connections persistent across all projects. At some point I may look into updating the Project Setup plugin to save a list of all the connections you know you use regularly - perhaps as a TOML file or a similar configuration language - and updating the plugin/macro code to allow you to interact with that list.

## Odds and Ends: Project Setup
The Project Setup plugin allows you to set a handful of other project variables, as well:

- project file (.qgz) name and save location
- project number
- project name
- project client
- project location (i.e. physical location, not filesystem location)
- project datasources

The "project datasources" variable is intended to be a list as well, but faces the same issues and workarounds as the "project geopackage connections" variable. I use most of these variables when generating a layout.

## Try it Yourself!
Note that all of my code is open-source licensed, so you're free to use it directly or adapt it as you see fit. The code here works well for my workflow and preferences, but may not match how you prefer to work - so change it however you like!
