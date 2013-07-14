**Sample code for [WMS On Android](http://www.azavea.com/blogs/labs/2013/01/wms-on-android/) originally published
in Android Weekly, 2013.**

+ [src/com/example/wmsdemo/MainActivity.java] (https://github.com/shalperin/android-wms-demo/blob/master/src/com/example/wmsdemo/MainActivity.java) : 
The demo application's activity.
+ [src/com/example/wmsdemo/TileProviderFactory.java] (https://github.com/shalperin/android-wms-demo/blob/master/src/com/example/wmsdemo/TileProviderFactory.java):
Initialize our WMSTileProvider with all of the static configuration that it needs to operate.
+ [src/com/example/wmsdemo/WMSTileProvider.java] (https://github.com/shalperin/android-wms-demo/blob/master/src/com/example/wmsdemo/WMSTileProvider.java) : 
The class that does all of the WMS work.

So essentially, the steps to get WMS working in your application are:
+ Create your own TileProviderFactory on the pattern of the one above.  The important part is that it has a static method
that returns a WMSTileProvider, and has a custom WMS request format in a  String variable.
+ Call the factory in your activity to get an instance of a WMSTileProvider
+ In your activity, call map.addTileOverlay with newly minted WMSTileProvider instance.

The main argument for using a WMS service that is not behind a tile cache (Which uses the TMS query format that more closely 
aligns with Googles overlay feature) is that you can request features that are constructed on the fly.  Examples of this are 
layers with selective styling using CQL queries.  It is relatively easy to add CQL parameters to the WMS query string above.

Example WMS Query String from the demo:

  final String OSGEO_WMS =
    		  "http://vmap0.tiles.osgeo.org/wms/vmap0" +
	    		"?service=WMS" +
	    		"&version=1.1.1" +  			
	    		"&request=GetMap" +
	    		"&layers=basic" +
	    		"&bbox=%f,%f,%f,%f" +
	    		"&width=256" +
	    		"&height=256" +
	    		"&srs=EPSG:900913" +
	    		"&format=image/png" +				
	    		"&transparent=true";
