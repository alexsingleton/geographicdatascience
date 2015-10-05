---
author: Alex Singleton
layout: post
avatar: alexsingleton
title: Building an open datastore for every local authority in the UK
comments: false
categories:
- CDRC
---

Using open data can be challenging, and especially for new users or those with limited analytical resources. It can be difficult to find what data are available, the formats can be confusing (or in some cases not that open!), and an advanced level of software skill may be required for their analysis. However, we don't think it has to be like this!

As part of the [ESRC Consumer Data Research Centre](https://www.cdrc.ac.uk/), we have spent the summer working on a new project that creates an open datastore for each local authority in the UK. 

**These are available here: https://data.cdrc.ac.uk/lad**

![datastore](/public/images/lad.png)

Each local authority also have a separate URL for their own datastore; so, Liverpool could be found here for example: https://data.cdrc.ac.uk/lad/liverpool

We do not believe in simple replication of data sources available elsewhere, and we have **added value** to each open data deposit by reengineering these into new formats that are optimized for simple analysis, and which we hope are going to limit barriers to entry.

As of 5/10/15 we have created **8,738k** separate data items for a very wide variety of topics.

Not every local authority have resources to create their own datastore, and for those which do, we hope that what we have created will be complementary. 

## Technical bits...

For those technically inclined, the software platform is based upon highly customised [CKAN](http://ckan.org/) / [Wordpress](https://wordpress.org/) installations and additionally uses the [datashine](http://datashine.org.uk/) platform to create an interactive map visualisation for some of our data deposits. 

![maps](/public/images/map_LAD.png)

The datapacks were all scripted, and created using a combination of [R](https://www.r-project.org/), [Python](https://www.python.org/) and [PostGIS](http://postgis.net/).

Thanks in particular go to the hard work of Data Scientists [Wen Li](http://www.geog.ucl.ac.uk/about-the-department/people/research-staff/research-staff/wen-li/), [Hai Nguyen](http://geographicdatascience.com/people/#hainguyen) and [Michail Pavlis](http://geographicdatascience.com/people/#michailpavlis) who have spent much of summer working on this project.




