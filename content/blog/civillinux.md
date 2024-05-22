+++
title = "Civil Consultancy on the Linux Platform"
+++

As I've mentioned elsewhere, I'm a civil engineer by trade. For all of my career thus far, I've worked for various consulting firms (i.e., I haven't been self-employed), and like everyone else I use the tools provided to me - a company-provided laptop and the licenses for whatever software they want me to use. Like many other enterprises, civil consulting is heavily tied to the Windows ecosystem; many industry-standard software packages are only available for Windows, and tools like Microsoft Office/MS Exchange are so much more convenient and ubiquitous than other alternatives that they're effectively the only options.

However, in my personal life I'm a big fan of Linux. I like the customizability, and I like that privacy is taken much more seriously on Linux than on Windows - Microsoft seems to be constantly finding new ways to harvest user data.

I have no idea if I'll ever be self-employed. It's not uncommon for established civil engineers to strike out on their own; I just don't know if I'll ever want to deal with the hassle. But a man can dream, and if I were ever to open my own shop, I would want to use Linux. In this post I'll lay out what would be necessary for that to work.

## CAD Software
The first, biggest, and most obvious obstacle to using Linux in civil consultancy is the need for good computer-aided design (CAD) software. The lack of good open-source CAD packages is frequently recognized as one of the biggest holes in the Linux ecosystem, by everyone from ordinary users to Richard Stallman himself. One of the main reasons for this is the complexity of the software; CAD is difficult to do right. In the general CAD realm, FreeCAD has recently been making great strides; however, civil engineering-oriented CAD is a niche within a niche, and requires different things than what you'd see out of a package like FreeCAD that focuses more on solid modeling. In my opinion, civil CAD is more closely related to GIS; a great deal of the work is simply drawing points, lines, and polygons. The biggest difference between civil CAD and GIS is really just that GIS is more class-oriented (focusing on editing and manipulating groups of objects at the same time), while civil CAD tends to be more focused on working with individual features/objects. 

Nevertheless, there's nothing inherently keeping a GIS software package from working well in a CAD context; in fact, packages like QGIS and ArcGIS Pro provide tools for manipulating individual objects already, they just aren't as efficient/easy to use as the highly specialized tool sets you'll find in CAD packages like MicroStation or Civil3d.

As such, one of my biggest long-term goals is to develop my CivilTools plugin for QGIS. CivilTools aims to provide an efficient CAD experience in QGIS by providing specialized tools similar to what you'd find in a dedicated CAD package. This is, of course, a huge undertaking, and I don't expect it will be usable for quite some time. However, my preliminary investigations into the QGIS API indicate that everything I'll need is already there; I just need to tie everything together.

The first priority for CivilTools will be to provide what I've previously mentioned, tools for efficiently creating and manipulating individual points, lines, and polygons. Long term, I also aim to provide tools for working with topographic data (in the form of triangulated irregular networks or TINs), and working with networks of pipes. These tools will be significantly more complex to code.

Of course, having the ability to create this data in a vacuum is useless if you can't share it with anyone, and even beyond the coding challenges, this is really the greatest obstacle to open source tools; many agencies, like state DOTs, require that work done for them results in a particular deliverable format, and there is increasing adoption of tools like Bentley's ProjectWise, which limits how you can work with data. Nevertheless, open source libraries for encoding and decoding proprietary formats like Bentley's DGN and AutoDesk's DWG continue to see progress, and provide a theoretical path forward for sharing data across consultant teams.

## H&H Modeling Software
Compared to the relatively constricted CAD landscape, the world of hydrologic and hydraulic (H&H) modeling software is relatively open and diverse. Two of the most commonly-used packages - HEC-HMS for hydrology and HEC-RAS for riverine hydrology - are freely distributed by the Army Corps of Engineers (albeit not open-sourced). Tools for storm sewer system modeling (SWMM), culvert modeling (HY-8), and quick hydraulic calculations (Hydraulic Toolbox) are freely available as well, and recognized industry-wide as reliable tools. Linux support for these tools ranges from an afterthought (HEC-RAS, HEC-HMS) to nonexistent (SWMM, HY-8), but they should be perfectly usable with the wine Windows emulator. This is another major difference with CAD; the proprietary CAD packages such as Civil3d and MicroStation are so closely tied to the Windows API and registry that they can't be successfully used in wine, and seem to even be buggy when run in a virtual machine.

Given the above, my focus when looking at H&H software isn't so much whether I have a useful tool, but in finding ways to integrate the software with the rest of my workflow. Another project (or group of projects) I want to work on is finding ways to integrate tools like HEC-RAS into QGIS directly, so I can run models - and more to the point, view their modeling results - directly within the GIS environment.

## Document Creation
We all know that MS Word is the worldwide standard for creating documents. In my experience though, it's not actually very easy to use when working on technical documents - how many times have we wanted to tear our hair out when trying to add a table to a Word document? Given that, were I to create my own firm, I would want to use a different tool - Typst.

Typst is a relatively new software package, that aims to be a modern alternative to the venerable LaTeX typesetting system. The overall idea of both packages is that document content and document styling are decoupled; you type all the content as plain text, and apply formatting rules separately. Having style settings applied explicitly, in a programming-like language, can make hunting down formatting issues much easier. Both systems also provide much more robust and flexible tools for working with tables, graphs, and other visual data organization.

Naturally, working with Typst has a learning curve. But to me I would consider the upfront effort to be worth it.

## Spreadsheets
Engineers love our spreadsheets. To their credit, Microsoft has created a magnificent tool in Excel; the ability to both view the results of a calculation, and also see the calculation itself, is unique and highly powerful.

The disadvantage of Excel, besides its proprietary status, is its reliance on VBA as its scripting language. VBA is ancient, has odd syntax that is highly dissimilar from anything else still in common use, and has limited applicability in any other domain.

The main open source alternatives to Excel are LibreOffice Calc, and Jupyter notebooks. LibreOffice has the advantage of having a very similar interface to Excel, while using Python as its scripting language. Jupyter Notebooks are more flexible, but have a higher learning curve, and may be more difficult for others to review given that they are less common in the engineering realm.

## Conclusion
Running a Linux-only civil engineering consultancy may be a pipe dream; the Windows stranglehold on the industry may be too difficult to break. But I like to think about it, and hope that my CivilTools project will provide a major missing piece to the requirements!
