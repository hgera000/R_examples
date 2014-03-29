# GIS/Maps in R
I am by no means an expert in GIS analysis, but this GIST should be a useful reference for the basic commands needed to import, transform and plot different types of GIS data in R. Some basic R knowledge is assumed, but the comments try and explain what is going on in each step.

## Shapefiles example

Shapefiles (extention `.shp`) are a common format for GIS data. The data used for this example are freely available via the [FSP Portal](http://www.fspmaps.com/dataportal/content/pakistan-financial-service-locations "Get the Data") and contain latitude/longitude values and urban/rural tags for UBL Omni financial agent locations in Pakistan.

#### 1. Import required packages
Note: to intall these packages type `install.packages("package_name_in_quotes")`

```
library(data.table) #Not a GIS package, but highly recommended for anyone using R.
library(ggplot2) #General graphing/plotting package 
library(ggmap) #Additional package to access google maps 
library(maptools) #For loading shapefiles
```

#### 2. Load the data into R

```
setwd('/home/username/path_to_data_folder')
agent = readShapeSpatial('UBL_FinancialServiceLocations.shp') #Read the shapefile
```

#### 3. Convert the shapefile to a data.table

`agent` above is a 'SpatialPointsDataFrame' object, which is not the easiest things to work with. It needs to be converted into a data frame (or data.table) in order to use ggplot. 

```
agent_dt<-data.table(slot(agent,'data'),key="OBJECTID")
setnames(agent_dt,old=c("LONGITUDE","LATITUDE","LandUse"),new=c("lon","lat","landuse")) #re-name key variables
```
`agent_dt` is now a data.table object. The first few rows look like this:

<TABLE border=1>
<TR> <TH>  </TH> <TH> OBJECTID </TH> <TH> OUTLET_ID </TH> <TH> AGENT_ID </TH> <TH> lon </TH> <TH> lat </TH> <TH> Name </TH> <TH> landuse </TH> </TR>
  <TR> <TD align="right"> 1 </TD> <TD align="right">   1 </TD> <TD align="right"> 16758 </TD> <TD align="right">  28 </TD> <TD align="right"> 67.08 </TD> <TD align="right"> 24.95 </TD> <TD> UBL </TD> <TD> Urban </TD> </TR>
  <TR> <TD align="right"> 2 </TD> <TD align="right">   2 </TD> <TD align="right"> 17390 </TD> <TD align="right">  68 </TD> <TD align="right"> 67.09 </TD> <TD align="right"> 24.92 </TD> <TD> UBL </TD> <TD> Urban </TD> </TR>
  <TR> <TD align="right"> 3 </TD> <TD align="right">   3 </TD> <TD align="right"> 20425 </TD> <TD align="right">  98 </TD> <TD align="right"> 67.11 </TD> <TD align="right"> 24.90 </TD> <TD> UBL </TD> <TD> Urban </TD> </TR>
  <TR> <TD align="right"> 4 </TD> <TD align="right">   4 </TD> <TD align="right"> 17370 </TD> <TD align="right"> 116 </TD> <TD align="right"> 67.05 </TD> <TD align="right"> 24.96 </TD> <TD> UBL </TD> <TD> Urban </TD> </TD> </TR>
  <TR> <TD align="right"> 5 </TD> <TD align="right">   5 </TD> <TD align="right"> 17258 </TD> <TD align="right"> 126 </TD> <TD align="right"> 67.02 </TD> <TD align="right"> 24.87 </TD> <TD> UBL </TD> <TD> Urban </TD> </TR>
   </TABLE>

#### 4. Plotting with ggplot

* A simple points plot is useful just to give an idea of what's going on. The option `alpha=0.2` will make points transperant and is very useful when plotting lots of data on top of one another. 

```
g = ggplot()+geom_point(data=agent_dt,aes(lon,lat),colour="lightsalmon",alpha=0.2)
print(g)
```
The output looks like [this.](https://raw.githubusercontent.com/hgera000/GIS_examples/master/R_plotting/Rplot01.png) Not that useful yet.

* The code below adds a google-map background

```
google = ggmap(get_map("pakistan",zoom=6,maptype="roadmap"))
g = google + geom_point(data=agent_dt,aes(lon,lat),colour="lightsalmon",alpha=0.2)
```
and now the map makes a lot more [sense.](https://raw.githubusercontent.com/hgera000/GIS_examples/master/R_plotting/Rplot02.png) 

* Finally, [ggplot2](http://docs.ggplot2.org/current/) is an extremely flexible package for making nice graphs (and in and of itself is a reason to learn R). The code below, for example, plots agents in Rural versus Urban locations in a different colour, and then presents the two maps side by side. 

```
pdf('Rplot03.pdf',height=800,width=1200) #save as pdf
g = google + geom_point(data=agent_dt,aes(lon,lat,colour=landuse),alpha=0.2)+
    facet_wrap(~landuse,nrow=1) + 
    theme(legend.position="none")
print(g)
dev.off() #close pdf
```

The map will be higher definition if saved as pdf, as shown above, but should look something like [this.](https://raw.githubusercontent.com/hgera000/GIS_examples/master/R_plotting/Rplot03.png)

* Ignoring the importing of packages, and renaming of a few variables, it took only 3 or 4 lines of code to make quite a nice looking map. 







