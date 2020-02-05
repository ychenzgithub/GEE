## References
* [GEE homepage](https://earthengine.google.com)
     * [Code editor](https://code.earthengine.google.com): a web-based IDE for the Earth Engine JavaScript API  
     * [Explorer](https://explorer.earthengine.google.com): a simple web interface to the Earth Engine API  
     * [Documentation](https://developers.google.com/earth-engine/): documentation for developers      
     * [Tutorials](https://developers.google.com/earth-engine/tutorials): official and community tutorials  
     * [Reference](https://developers.google.com/earth-engine/api_docs): api documents
* GEE Data 
     * [List of GEE data](https://developers.google.com/earth-engine/datasets/)  
     * [List of GEE vector data](https://developers.google.com/earth-engine/vector_datasets)     
* [GEE API on github](https://github.com/google/earthengine-api): JavaScript and Python wrapper functions for the Earth Engine API
* [GEE reference paper](https://www.sciencedirect.com/science/article/pii/S0034425717302900):
_Gorelick et al (RSE, 2017), Google Earth Engine: Planetary-scale geospatial analysis for everyone_
      
## Applications and projects
* [GEE time lapse](https://earthengine.google.com/timelapse/)
* [Global Forest Change](https://earthenginepartners.appspot.com/science-2013-global-forest)
* [Global surface water explorer](http://global-surface-water.appspot.com/)
* [Global surface water changes](http://aqua-monitor.appspot.com/)
* [EarthEnv](http://www.earthenv.org)
* [Global croplands](https://croplands.org)
* [Global Height Above the Nearest Drainage](http://global-hand.appspot.com/)
* [GFED4s Explorer](https://globalfires.earthengine.app/view/gfedv4s)
* [FIRECAM](https://globalfires.earthengine.app/view/firecam)

## Useful tutorials
* [Add a legend to map](https://mygeoblog.com/2016/12/09/add-a-legend-to-to-your-gee-map/)

## Other GEE information
* [Medium.com](https://medium.com/google-earth): A collection of google earth and google earth engine articles
* [Open Geo Blog](https://mygeoblog.com/category/google-earth-engine/): Tutorials, Code snippets and examples
* [StackExchange](https://gis.stackexchange.com/tags/google-earth-engine/): Technical questions about GEE
* [GEE google group](https://groups.google.com/forum/#!forum/google-earth-engine-developers): Discussion group for GEE developers

## My GEE
* [Tutorial](https://code.earthengine.google.com/?accept_repo=users/ychen17/LabTutorial): An entry-level tutorial I created 

## Installation and initialization

### [Web based IDE](https://code.earthengine.google.com/)
Directly run GEE javascript online.

### [Python api on local machine](https://developers.google.com/earth-engine/python_install-conda.html)
* Installation
```
Conda install google-api-python-client
Conda install earthengine-api
earthengine authenticate
```
* Initialization
```
import ee
ee.Initialize()
```
### [Python api on Colab notebook](https://developers.google.com/earth-engine/python_install-colab.html)
```
import ee
ee.Authenticate()
ee.Initialize()
```
