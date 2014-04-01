# Android Web Map Service

## Implemented Using a Google Maps Android API v2, TileLayer

<p>Android Weekly, 2/2013<br>
Sam Halperin<br>
email: <a href="mailto:sam@samhalperin.com">sam@samhalperin.com</a>
</p>

<center>
![WMS Image](http://auto2.cdn.azavea.com/blogs/labs/wp-content/uploads/2013/01/306x600xptm-phone-306x600.png.pagespeed.ic.9fKei2Oev9.png)
<br><br>
<a href="http://www.phillytreemap.org">PhillyTreeMap</a> Android App (Released Spring 2013) <br> showing WMS technique described in the article.</i>

</center>
<h3>Intro</h3>

<p>This brief article documents some early exploration into using WMS with the new Google Maps V2 API.  It is intended as a reference to help someone trying to get WMS tiles (IE from GeoServer) onto an Android map.</p>

<p>
WMS is used to serve map tiles over HTTP by back end frameworks like GeoServer.  Some set of geo-referenced data, typically shape files or data stored in a PostGIS database, are returned as raster map tiles.  In the past, this data has been consumed by web applications using a client library such as Leaflet or OpenLayers.  With Google’s v2 mapping API for android, it is now relatively straightforward to build Android apps that combine WMS tiles with Google’s base maps and other data such as vector shapes and map markers.
</p>

<p>
For basic <i>getting started</i> info for v2 Maps, see the <a href="https://developers.google.com/maps/documentation/android/?hl=fr-FR">Google Developer's site for the v2 API</a>.  This article assumes a working v2 setup with the sample code running without error. After downloading the google play SDK and setting up the library make sure you can view the TileOverlayDemo. ($ANDROID_SDK_ROOT/extras/google/google_play_services/samples/maps)</p>

<h3>Extending the UrlTileProvider class.</h3>
<p><i>Please refer to the sample code at the end of this post.</i></p>
<P>The v2 API provides the UrlTileProvider class,  a partial implementation of the TileProvider class which allows developers to pull in map tiles by composing a URL string.  
</p>
<p>
    The API to UrlTileProvider is its getTileUrl method.  To request a WMS tile,
    we override this method to compose the right URL, and the Android mapping SDK does the rest for us. It seems simple enough, but the problem is that the signature of getTileUrl which is <b span style="font-family:monospace">getTileUrl(int x, int y, int zoom)</b> provides tile indexes (x and y) and a zoom level, but WMS requires that we provide a bounding
    box (xmin, ymin, xmax, ymax) in the request URL.  The x, y, zoom parameters provide us enough information to figure this out, but we have to do a little bit of math.
<p>


<h3>Calculating the map bounds</h3>
<P>We know the bounds of the entire map which is square. (roughly -20037508m to 20037508m in both directions using Web Mercator.  See the graphic below or the <a href="http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/">map tiler site</a> for exact values.)    We don't use Latitude/Longitude though, because it is unprojected, and this will cause map distortions.
</p>

<p>From the <a href="https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/TileOverlay">google api docs for TileOverlay</a>:<br>
<i> Note that the world is projected using the Mercator projection (see <a href="http://en.wikipedia.org/wiki/Mercator_projection">Wikipedia</a>) with the left (west) side of the map corresponding to -180 degrees of longitude and the right (east) side of the map corresponding to 180 degrees of longitude. To make the map square, the top (north) side of the map corresponds to 85.0511 degrees of latitude and the bottom (south) side of the map corresponds to -85.0511 degrees of latitude. Areas outside this latitude range are not rendered.</i>
</p>

<h3>Dividing by the number of tiles for a given zoom level.</h3>
<p>
The number of tiles in either x or z at any zoom level is  n = 2^z.   With this, and the bounds of the map, we can figure out the size of the tile.  Using this information combined with the maps origin (see graphic) and the x, y, zoom data for a given tile, we can find out its bounding box.
</p>

<p>Again from the <a href="https://developers.google.com/maps/documentation/android/reference/com/google/android/gms/maps/model/TileOverlay">google api docs for TileOverlay</a>:<br>

<i>At each zoom level, the map is divided into tiles and only the tiles that overlap the screen are downloaded and rendered. Each tile is square and the map is divided into tiles as follows:</p>

<ul>
<li>At zoom level 0, one tile represents the entire world. The coordinates of that tile are (x, y) = (0, 0).</li>
<li>At zoom level 1, the world is divided into 4 tiles arranged in a 2 x 2 grid.</li>
<li>...</li>
<li>At zoom level N, the world is divided into 4N tiles arranged in a 2^N x 2^N grid."</li>
</ul>
</i>
</p>


![Map Bounds](http://www.samhalperin.com/image?key=ahNzfmhhbHBlcmluLWJsb2ctaHJkcg0LEgVJbWFnZRjivAsM)
<p></p>

<p>
<h3>Summary</h3>
<ul>
    <li style="margin-bottom:12px;">zoom  level: z = [0..21]<br><i>(See GoogleMap.get[Min|Max]ZoomLevel)</i></li>
    <li style="margin-bottom:12px;">map size: S = 20037508.34789244 * 2 <br><i>This constant comes from converting the lat/long values above to EPSG:900913, Web Mercator. Again, see <a href="http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/">this page on maptiler</a> for a fantastic visual explanation.</i></li>
    <li style="margin-bottom:12px;">tile size = S / Math.pow(2, z)<br><i> So @ zoom level 0, S is the full map, at zoom level 1 there are 2x2 tiles, and at zoom 3 there are 8x8 tiles and so forth.</i></li>
    <li style="margin-bottom:12px">tile origin = (-20037508.34789244, 20037508.34789244)</li>
    <li style="margin-bottom:12px;">minX of the tiles bbox (for tile index x,y) = origin.x + x * S<br><i>Where x is the tile index in the east-west direction passed to the getTileUrl function discussed above. </i></li>
    <li style="margin-bottom:12px;">maxX of the tiles bbox (for tile index x,y) = origin.x + (x+1) * S<Br><i>x+1 because we are looking for the right edge of the tile.</i></li>
    <li style="margin-bottom:12px;">minY of the tiles bbox (for tile index x,y) = origin.y + y * S</li>
    <li style="margin-bottom:12px;" >maxY of the tiles bbox (for tile index x,y)= origin.y + (y+1) * S</li>
</ul>

<h3>Demo Code</h3>

<p>
In addition to the following code snippets, I’ve put <a href="https://github.com/shalperin/android-wms-demo">a sample project on github.</a>
</p>
<p>
Here is a WMSTileProvider class, which inherits from UrlTileProvider.  It supports the above bounding box calculation.</p>

<pre><code class="language-java">
import com.google.android.gms.maps.model.UrlTileProvider;
 
public abstract class WMSTileProvider extends UrlTileProvider {
  
    // Web Mercator n/w corner of the map.
    private static final double[] TILE_ORIGIN = {-20037508.34789244, 20037508.34789244};
    //array indexes for that data
    private static final int ORIG_X = 0; 
    private static final int ORIG_Y = 1; // "
    
    // Size of square world map in meters, using WebMerc projection.
    private static final double MAP_SIZE = 20037508.34789244 * 2;
    
    // array indexes for array to hold bounding boxes.
    protected static final int MINX = 0;
    protected static final int MAXX = 1;
    protected static final int MINY = 2;
    protected static final int MAXY = 3;
    
    
    // Construct with tile size in pixels, normally 256, see parent class.
    public WMSTileProvider(int x, int y) {
        super(x, y);
    }
        
    
    // Return a web Mercator bounding box given tile x/y indexes and a zoom
    // level.
    protected double[] getBoundingBox(int x, int y, int zoom) {
        double tileSize = MAP_SIZE / Math.pow(2, zoom);
        double minx = TILE_ORIGIN[ORIG_X] + x * tileSize;
        double maxx = TILE_ORIGIN[ORIG_X] + (x+1) * tileSize;
        double miny = TILE_ORIGIN[ORIG_Y] - (y+1) * tileSize;
        double maxy = TILE_ORIGIN[ORIG_Y] - y * tileSize;
  
        double[] bbox = new double[4];
        bbox[MINX] = minx;
        bbox[MINY] = miny;
        bbox[MAXX] = maxx;
        bbox[MAXY] = maxy;
        
        return bbox;
    }
    
}
</code></pre>

You might use this class in a <i>factory</i> class as follows:

<pre><code class="language-java">
import java.net.MalformedURLException;
import java.net.URL;
import java.util.Locale;
import android.util.Log;
import com.google.android.gms.maps.model.TileProvider;
 
public class TileProviderFactory {
  
    private static final String GEOSERVER_FORMAT =
            "http://yourApp.org/geoserver/wms" +
            "?service=WMS" +
            "&version=1.1.1" +              
            "&request=GetMap" +
            "&layers=yourLayer" +
            "&bbox=%f,%f,%f,%f" +
            "&width=256" +
            "&height=256" +
            "&srs=EPSG:900913" +
            "&format=image/png" +               
            "&transparent=true";    
    
    // return a geoserver wms tile layer
    private static TileProvider getTileProvider() {
        TileProvider tileProvider = new WMSTileProvider(256,256) {
            
            @Override
            public synchronized URL getTileUrl(int x, int y, int zoom) {
                double[] bbox = getBoundingBox(x, y, zoom);
                String s = String.format(Locale.US, GEOSERVER_FORMAT, bbox[MINX], 
                        bbox[MINY], bbox[MAXX], bbox[MAXY]);
                URL url = null;
                try {
                    url = new URL(s);
                } catch (MalformedURLException e) {
                    throw new AssertionError(e);
                }
                return url;
            }
        };
        return tileProvider;
    }
    
}
</code></pre>


So your Activity code (again see the sample Google maps code referenced above) would have the following calls to add the overlay:

<pre><code class="language-java">
public class MapDisplay extends android.support.v4.app.FragmentActivity {
 
//...
 
  private void setUpMap() {
        TileProvider tileProvider = TileProviderFactory.getTileProvider();
        mMap.addTileOverlay(new TileOverlayOptions().tileProvider(tileProvider));    
    }
}
</code></pre>


<h3>Conclusion</h3>

<p>
    This article presented a demonstration of a simple WMS client using Google's Android v2 Maps API.  It covered the math involved with converting from tile index/zoom level to Web Mercator bounding box, and showed how to compose a URL using these values and an instance of Google's UrlTileProvider class. 
</p>


### This article was covered in the following Google Maps Garage episode:
<p><a href="https://www.youtube.com/watch?feature=player_embedded&v=U6ZbHAXPnhg"><img src="http://www.samhalperin.com/img/projects/wms-android/garage2.png"></a></p>
<iframe width="560" height="315" src="//www.youtube.com/embed/U6ZbHAXPnhg" frameborder="0" allowfullscreen></iframe></div>