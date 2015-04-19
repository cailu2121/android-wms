Quick Start
===========
+ Clone the git repo.
+ **Import** this project into Android Studio.
+ Get a Google V2 Android Maps API key and add it 
   to the manifest (see TODO in Manifest)
+ Check that the demo WMS service in TileProviderFactory is still
    valid, or add your own.  (See the TODO in TileProviderFactory)
+ Run 
    **preferably on a real device, this project needs 
      google play services which can be a pain to get running
      on an emulator (though not impossible.)


Debugging hints
===============
+ The app will crash if the API key isn't set up properly.

+ A repeated message like this:

  04-19 09:14:28.426  21146-21259/com.example.wmsdemo D/skia﹕ --- SkImageDecoder::Factory returned null

indicates that your WMS url isn't returning tiles for whatever reason. Check that the service still exists and that you are setting up the URL properly in TileProviderFactory.

+ NB: Don't neglect to study up on what the fields in an WMS request mean, particularly SRS.

