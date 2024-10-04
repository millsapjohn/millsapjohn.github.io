+++
title = "Using Whitebox Tools in Drainage Workflows"
date = "2024-05-17"
+++

I'm a drainage engineer by trade. Drainage engineering is a subfield of civil engineering, focused on a single question: what happens to a particular area when it rains?

Naturally it's not that simple. That question can be redefined in a number of ways, to focus on things like
- does a particular street/lot/river/area flood when it rains?
- do any pollutants end up in the runoff when it rains, and if so, where do those pollutants end up going?

and at the end of the day, it boils down to: what do we need to build to prevent negative effects to a given area?

The first step in designing that kind of infrastructure is determining, when it rains, how much water will actually be running off to a certain point? This is affected by a number of factors:
- how big of a rainstorm are we dealing with? A typical, "we get this kind of storm every spring" type of storm, or a "this only happens every hundred years" storm?
- how big of an area is draining to a particular point? As we all know, water flows downhill - how much area is uphill of our study point?
- how long does it take for runoff to collect at our study point? The longer it takes for water to collect at a point, the less likely that a storm will last for that long.
- what's covering the ground in our study area? Forests and grassy plains absorb a large amount of water, so very little actually runs off downhill; roofs, streets, and other impervious materials don't absorb much at all.
- finally, what kind of soils do we have in our study area? Certain types of soils, like sand or sandy loam, absorb water very readily; others, like clay, don't absorb much.

Determining those factors is referred to as a "drainage study", and it's a large part of a drainage engineer's job. It can be a time-consuming process - in particular, identifying how much area is uphill of our study point (referred to as the point's "watershed") can involve painstaking study of the area's topography, and determining how long it takes for all the flow to collect at the study point (referred to as the "time of concentration") involves drawing hypothetical flow paths within a watershed and applying a series of calculations to determine how long it takes for the water to travel overland. There's also the process of collecting land cover and soils data to determine how much of the theoretical rainfall is being absorbed by the land. And of course, even once you've collected all of this data, there isn't a single "one size fits all" method for estimating how they all work together!

The GIS-minded reader will probably be pricking up their ears at this point. Land cover? Soils data? Topography? Those are all types of geographic data, surely there must be a way to apply GIS analysis to this problem! And you'd be correct - this post will be describing the workflow I've developed to make the drainage study process faster and more reliable.

## The Toolset: Whitebox Tools
Whitebox Tools is a suite of geospatial tools developed by Dr. John Lindsay at the University of Guelph and distributed by Whitebox Geospatial, Inc. The suite of tools is built on a "freemium" model - the core of the suite is free and open-sourced, while a number of advanced tools are available only via subscription. The suite has tools for a large number of applications, including drainage/hydrological analysis - and all the tools I use are part of the open-sourced core. The company also maintains a free QGIS plugin for using the tools seamlessly in QGIS. A large part of my workflow uses Whitebox, but I also integrate a lot of features from QGIS itself. Here's the process.

## Step 1: Gathering Data
Regardless of the exact hydrological analysis method used, you can be certain that you'll need:
- land cover data
- topographic data
- aerial imagery (not technically required, but immensely useful)
- stream centerlines
- any existing drainage infrastructure (in particular, culverts running under roadways)
- road centerlines

Depending on the analysis method, you might also need soils data. So where do you get them?

My rule of thumb is to try and get data from the "smallest" source possible - data published by a city is preferred over data from the state, which is preferred over data from a national organization. This is simply because local data is likely to be more fine-grained and more frequently updated. My city, for example, publishes an impervious cover dataset in vector format that theoretically shows every piece of impervious cover (roofs, roads, concrete, etc) across the entire city. If such a dataset isn't available for your study area though, you can always fall back on the National Land Cover Dataset, which does indeed cover the entire United States - albeit at a lower resolution.

Stream centerlines are available from the National Hydrography Dataset if your city/state doesn't publish them; similarly, roadway centerlines should be available from your state DOT if nothing else. These two pieces of data are useful for ensuring the watershed analysis tools properly account for flow running through a creek or culvert under a road; more on that later.

My state (Texas) also publishes a great deal of LiDAR data for the entire state - ranging from very high-resolution (1ft) in large urban areas to coarser resolution (10ft) in rural areas. If it's not available in your study area, the National Geographic Service will have some elevation data in your area, although it might be at a very coarse resolution.

Imagery is available from a wide number of sources - personally I'm totally content to use Google Aerial.

Finally, as far as I'm aware, soils data is typically only available from the USDA's Natural Resource Conservation Service. They do have data for the entire United States, and work is continually ongoing to improve the accuracy and resolution of the data.

## Step 2: Watershed Preprocessing
Once you've acquired the necessary data, you'll need to prepare it. LiDAR tiles are usually pretty large, so if possible you'll want to clip them down somewhat - but err on the side of caution, as over-clipping can lead to portions of your watershed being cut off. Similarly, the NLCD (if you're using it) will be covering the entire state, so you'll want to trim it down to size.

First, you'll want to use the "BurnStreamsatRoads" tool. First, understand that the purpose of watershed delineation isn't to know exactly how much water is flowing at any given point, but simply to figure out where rainfall on any given point eventually ends up. The BurnStreamsatRoads tool cuts an artificial "channel" where streams cross roads because, as far as LiDAR is concerned, the road would act as a dam and block the stream from continuing. This tool helps ensure flow patterns are as accurate as possible.

Next you'll run the terrain through Whitebox's BreachDepressions tool. Imagine a small sag or depression in an open field - in the real world, during a rainstorm the depression will fill with water almost immediately and allow runoff to continue flowing downhill; however, without knowing how much water is flowing through the area, a computer program has no way of knowing when or if it will fill, and would otherwise consider it a "break" in a watershed. Using the BreachDepressions tool cuts artificial breaks into any depressions found in the LiDAR to ensure that flow has a "way out".

Now you should have a processed terrain that has gotten rid of any artificial obstacles to flow. The next step is to develop a "pointer" raster. Every cell in a pointer raster has an integer value between 1 and 8, corresponding to which of its neighboring cells it flows to (in a grid of square cells, every cell has exactly 8 neighbors). Use the Whitebox "D8Pointer" tool for this step.

Next, you develop a flow accumulation raster. In this raster, every cell has a value corresponding to how many cells are uphill of that cell. Use the Whitebox "D8FlowAccumulation" tool for this step.

The final preprocessing step is to ensure your watershed "pour points" - the points for which you want to develop a watershed - are correctly located. For example, if you're analyzing the watershed for an existing storm sewer inlet, you might have received a point from a surveyor or GIS dataset that doesn't exactly correspond with the local low point in your elevation data. Whether the LiDAR low point or the survey/GIS point is more accurate doesn't actually matter, as you just want to ensure that the watershed analysis is getting the upstream area. You can use the Whitebox "SnapPourPoints" tool for this. The tool takes a flow accumulation raster, a set of vector pour points, and a search radius, and looks for the highest accumulation value within the radius.

What search radius you use will depend on how confident you are in your initial pour points. In the above example, if you have an inlet point, it's very likely that the corresponding low in the LiDAR will be within 5 feet or so; you don't want to use a larger search radius than you actually need, as it might snap your pour point to the other side of the road, into a creek nearby, etc. Use engineering judgment, and don't be afrain to iterate multiple times.

## Step 3: Watershed Delineation
Now you should have a good preprocessed terrain and some well-sited pour points, so the final step is to run the Whitebox "Watersheds" tool. You'll use the pointer raster you developed and your snapped pour points to generate a raster of watersheds for all your pour points.

At this point you'll want to take a step back and review what you've got pretty closely. No matter how good the tool, it can make mistakes - it's your responsibility as the engineer to make sure you're comfortable with the results. Use the QGIS "Contour" tool to generate contours from your LiDAR data, and see if there are any areas where you disagree with the Whitebox algorithm. It's not uncommon for me to make minor edits at this stage in the proceedings; if you feel it's necessary, you can do so easily by vectorizing the raster data, editing the resulting polygons, and then rasterizing the edited polygons (you'll need the raster watersheds for another step).

## Step 4: Flowpath Delineation
Next you'll want to develop flowpaths for calculating time of concentration for each watershed. This is another step that requires close attention from the engineer; the algorithm (Whitebox's "LongestFlowpath" tool) can only calculate the flowpath with the longest physical path, when what we're concerned with is the longest **travel time.** The two are often the same, but not always - be sure to check the results closely, and be sure you use your processed terrain, rather than the raw LiDAR. The "LongestFlowpath" tool requires a raster of the watersheds, so if you made any edits to the watersheds in the previous step, be sure you've rasterized the results and use those as the algorithm input.

## Step 5: Time of Concentration Calculations
NOTE: all of the below steps are for the TR-55 method of calculating time of concentration. There are other methods, but this is by far the most common, and if you need to use another method, the steps below should at least give you an idea where to start.

You could theoretically run all the calculations entirely in GIS, but given that jurisdictions differ in how those calculations should be done, I find it easiest to simply use GIS to calculate the necessary physical characteristics of a flowpath and do the rest of the calcs in a separate spreadsheet. This approach also makes it easier for someone else to review your work - very important in engineering.

First, you'll want to divide your flowpaths into segments based on flow regime.

For the purposes of determining the time it takes for a theoretical water droplet to travel along a path, there are three main "flow regimes":
- sheet flow consists of dispersed runoff traveling in a general direction without gathering into a concentrated flow. This type of flow moves the slowest and is the least stable; most studies indicate that it can only occur for a maximum of about 100 feet before the flow starts to concentrate.
- shallow concentrated flow consists of flow that isn't contained in a well-defined channel, but nevertheless has started to "gather together" through random variations in topography. A large dirt clump can be enough to cause flow to concentrate! This flow regime moves much faster than sheet flow, but slower than channelized flow.
- channelized flow occurs wherever runoff has been gathered into a well-defined channel, whether natural or manmade. This is typically the fastest form of flow.

The travel times for each type of flow are calculated differently, so it's necessary to divide a flow path into segments. The first step is to use the QGIS "Interpolate point on line" tool to generate a point 100 feet from the beginning of each flowpath. Here (and for the remaining steps in analyzing flowpaths) you'll want to use imagery to ensure that sheet flow can actually occur for the full 100 feet; if the flowpath goes into a roadside ditch after 50 feet, you clearly can't consider that sheet flow! Nevertheless, dropping a point on each line at 100' is a good start.

For the rest, use imagery and contours to see where your flow is changing regime. Drop a point at each change, making sure it lies directly on the flowpath (if you set your snapping to "edge" you can be sure that the point is directly on the line).

Next, you'll want to create a scratch line layer and draw a line for each regime change point; these lines should connect directly to the regime change point at one end. Where the other end is sited doesn't matter, as long as the line doesn't cross the flowpath.

You'll use these lines in QGIS' "Split with lines" tool. After doing so, you should have a line segment for each flow regime on each flowpath, and we can move to calculating physical characteristics.

For each segment, what really matters is 1) the segment length, and 2) the segment slope. The length is easy to calculate: open the attribute table for your flow segments layer, open the field calculator, and create a field named "Length". In the expression box, simply type

    $length

to access QGIS' built-in length property. Make sure you set the field type to "decimal number".

In order to calculate the slope, you'll need the starting and ending elevation of each segment. First, you'll need to assign Z values to your line segments, which can be accomplished with the Drape tool. Then you can get the Z value at the starting and ending vertices with the expression

    z_at($geometry, 0)

or

    z_at($geometry, -1)

The `z_at` function's second argument is the vertex index at which you want to calculate the Z value; like a lot of programming languages, the QGIS expression language uses the negative symbol for counting backward from the end; a value of -1 corresponds to the last vertex on the line.

Finally, you can calculate the slope with the following expression:

    if((abs("Start_Elev" - "End_Elev") / "Length")<0.005,0.005,(abs("Start_Elev" - "End_Elev") / "Length"))

Note that in the QGIS expression language, double quotes indicate a field name, so here I'm using the calculated fields rather than the actual geometry properties. I do this for two reasons. First, I'm again looking at this with an eye to being reviewable; it's much easier for another engineer to verify my calcs if all the fields are explicitly calculated. Second, if for some reason I want to edit the fields - say that I got a bad elevation at a particular vertex for some reason - I can manually edit the field and have the calculation use my corrected values.

Additionally, note the overall format of the QGIS "if" expression: `if(condition,value if true,value if false)`. This is fairly familiar if you've used Excel before. In this calculation I'm setting a minimum value of 0.005 ft/ft for the slope; this is fairly common for many jurisdictions, but you'll want to check the manual for your project area.

The last thing to note is the `abs` function. As far as I'm aware, the Whitebox LongestFlowpath tool will always return lines that begin on the uphill end, but I still use the absolute value just in case.

Now you can use the physical properties of the flowpaths to calculate time of concentration for each watershed. The last GIS-related task is to evaluate land cover/soils data.

## Step 6: Land Cover/Soils Data
This section will be somewhat general, because there an enormous number of methods for dealing with land cover as it pertains to drainage analysis. Two of the more common methods are the Rational Method and the SCS method.

The Rational Method ignores soils entirely and assigns a coefficient between 0 and 1 for different cover types. Because it ignores soils, it's only appropriate for small watersheds, but it's frequently used because of its simplicity.

The SCS method is considered more accurate than the Rational method, but is more complex. It does take into account different soil types - using the "Hydrologic Group" property calculated by the NRCS in its soil surveys. The SCS method assigns a "Curve Number" between 0 and 100 based on the soil and land cover.

In addition to the many different methods for calculating how land cover impacts runoff, additional complexity is introduced by the fact that different jurisdictions will use different coefficients/curve numbers for the same general land cover type, and the fact that land cover data is often unique to a specific jurisdiction. For this reason I can't be too specific in this section, but I will make some general notes.

Once the appropriate method has been selected for your analysis, the most important step will be to create a table mapping between whatever values you have your land cover data in, and the corresponding coefficient, Curve Number, or other parameter. If your land cover values are not in a numeric format, you'll want to create that mapping as well - "Single Family Residential" corresponds to land use code 001, etc. If there isn't clear correspondence between the coefficient values in the code you're using and the land use data you have, you'll want to get sign-off from your superior before moving forward. For example, if you're using the NLCD data, I've yet to come across a drainage criteria manual that clearly mapped those values to a coefficient. You'll need to have the necessary conversations to get agreement that "Developed, medium intensity" in the NLCD corresponds to "Single Family, Large Lot" in the criteria manual.

Additionally, if your method of analysis uses soil hydrologic group, use a three-digit code for soils (A=100, B=200, etc) and a 2-digit code for land cover. This way you can simply add the two values together to get a coefficient/curve number.

For example, in the SCS method, land cover types have different curve numbers depending on what soil is beneath them - using the above methodology, you can simply sum the values of your soils and land cover rasters, and know that a value of 101 means land cover type 001 over soil group A.

With this in mind, I strongly recommend using raster data for this portion of the analysis. I personally find it much easier and more intuitive to use the Raster Calculator for these operations than to deal with numerous overlay/dissolve steps, which is what you'd have to do if you used vector data for the analysis.

If use the raster method, the general process would be

- assign a numeric code to all vector features
- rasterize the vector features, using the numeric code field as the burn-in value
- if using soils data, sum the land cover and soils rasters
- using your mapping table, reclassify the raster
- using your watershed raster map, run zonal statistics on your reclassified raster

The mean value that you'll get in the zonal statistics report will be the coefficient/curve number/etc of the watershed.

## Conclusion
That's my process! Hopefully someone, somewhere, will find it helpful. As with all processes or workflows, there's always room for improvement - one area in particular I'm thinking about recently is trying to integrate Jupyter Notebooks into the workflow. The advantage of this would be being able to combine the automated calculation of GIS with the reviewability of spreadsheets. If I come up with a good workflow for Jupyter, I'll make a new post about it.
