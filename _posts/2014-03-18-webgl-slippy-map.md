# Prototype _COOLHV3_

At Cobalt, we handle a lot of traffic.  We host the websites of many auto dealers, and we do a great deal of advertising
on their behalf.  It's enough traffic that it's a challenge to really understand the scope of all that we do. We
recently prototyped an attempt to show a good portion of our data on a map.  Conceptually it was very simple- just
draw a circle on the map for every page viewed, every ad served, and every ad that was clicked on.  The challenge came
in the rate of these events- at peak times we see more than 1200 events/second. Displaying that many events on an
interactive web page proved to be a very interesting technical challenge.

## A Background in Geographic Projections

Most of us start out thinking of maps as being a special case of the Cartesian plane we learned about in our first
Geometry classes.  Somebody decided to call the _y_ axis 'latitude', the _x_ axis 'longitude', and the units 'degrees',
but other than those strange differences, they seem the same.  Sure we know that a map is flat and the world is not,
but it's easy to ignore that and presume it's not that big of a difference.

### Latitude and Longitude

So if latitude isn't _y_ and longitude isn't _x_, what are they?  Imagine a transparent sphere- put a laser pointer in
the center, pointing outwards.  Orient the laser pointer so that it's perpendicular to the line between the poles of
the sphere.  Spin the laser pointer on that perpendicular plane- that's the equator.  Rotate it to be 10º above
perpendicular.  Now spin it horizontally- the line it makes is ten degrees latitude.  Keep moving it up- when it's
parallel with the sphere's axis, it's at 90º.  Now start at a pole and swing it along the axis to the other pole and
back- a full circle of 360º. There's longitude, though we give it a range of -180º to 180º.  The axis between
the poles gave us a natural reference for 0º latitude, but where is 0º longitude?  At the Greenwich Naval Observatory,
where all the old men who came up with this scheme worked.

### Mercator and How He Messed You Up

Our expectations for maps are set mostly by the most common map 'projection', called the 'Mercator Projection'.
In the Mercator projection, North is always towards the top, South is always down, West is to the left and East is to
the right; a projection that matches our perceptions of a flat world and what we see when we look at a compass.  But
again the world is not actually flat and those expectations can come back to bite us. It's the most common of a type
of projection called 'cylindrical projections'.

Imagine taking a light and placing it in the center of a translucent globe.  Then take a piece of paper and wrap it
around the globe so that the edges meet along the International Date Line through the Pacific.  Now trace the contours
that are projected from the globe onto the paper- that's a cylindrical projection.

Alternately, take a globe made of rubber, slice out a patch around the poles (cylindrical projections aren't defined
at the poles), cut down the date line, and then stretch it out until it's rectangular.  You can imagine that there won't
be any stretching at all in the middle (the equator), but the edges at the top and bottom will have to stretch a _lot_
in order to reach the edges.

With Mercator, we get something that looks Cartesian, but it's giving us a lot of distortion up towards the poles.
How much distortion?  Well here's Greenland plotted twice at the same scale.  The one on the left is a Mercator
projection.  The one on the right is a _transverse_ Mercator projection- wrapping the paper with the seam along the
equator rather than along the date line so that the distortion is around the equator instead of the poles.

![Mercator vs Transverse Mercator](/diagrams/mercator-distortion.png)

At a global scale, Mercator is a horrible projection- we can't correctly compare the sizes of countries (Greenland is
actually the size of Algeria) and we never navigate at that scale[^great-circle-charts].  But there are
many projections- conical, cylindrical, gnomonic, isometric. In the end, a projection is just a mathematical
[function](http://en.wikipedia.org/wiki/Function_(mathematics)) that takes an input of geographic coordinates and
returns a set of cartesian coordinates suitable for a two-dimensional representation.

[^great-circle-charts]: In a previous life I was a [Quartermaster](http://en.wikipedia.org/wiki/Quartermaster) in the
U.S. Coast Guard.  During my tenure there I got to plan several long cruises- the first was from Sitka, AK to
Honolulu, HI.  While we are all familiar with the adage that the 'closest distance between two points is a straight line',
that doesn't make much sense in navigation- with a round planet, the closest distance between two points leads
_through the earth_. When navigating across the Earth, the closest route will be a
[great circle route](http://en.wikipedia.org/wiki/Great_circle_route)- an arc of a circle which passes through the center
of the earth.  In order to determine this course, we plotted it on a small scale chart (covering a wide area) that
used a [Gnomonic projection](http://en.wikipedia.org/wiki/Gnomonic_projection).  We then broke that line into segments,
got the latitude and longitude of the endpoints of each segment, and then overlaid them on a Mercator projection to
determine the course to steer for that segment of the voyage.  Just a tangential illustration of various uses of
different projections.

### But Why is Mercator Everywhere?

Because there is no perfect mapping from a 3-dimensional sphere to a flat screen or paper.  A Mercator projection works
the way that we _think_, though.  North is always up.  East is always to the right.  If you draw a line between one
point on the map and another and then measure the angle between, that's the measurement that you need to hold on your
compass to walk right there.  In short, for navigation between places that aren't too far apart and aren't too close to
the poles, it works exactly as you would expect. It's the most intuitive, and as it turns out, there aren't many people
living near the poles to complain.

The Mercator projection also divides the world nicely into a grid.  It's become the _de facto_ map projection over the
last few years largely because Mapquest used this to their advantage.  Take the world's geographic data at the finest
grain you have available, project it, and evenly divide it into 256px by 256px tiles.  That's your closest zoom level.
For the next zoom level, take 2 tiles by 2 tiles and make that one 256px by 256px tile.  Keep building a zoom level
'pyramid' like that until you can see the whole planet on a page.

In doing this, we can easily find the right tile for any desired point at any desired level of detail, and for any given
tile we can easily determine the points within.  Mercator enables the 'Slippy Map'.

## Slippy Maps

These days when we think about maps on the Internet we think about Google Maps. Before Google Maps came Mapquest. Now
we have a huge variety of online maps (Bing, OpenStreetMap, Apple Maps), libraries to work with them (OpenLayers, Modest
Maps, Google Maps API, Leaflet.js), and cloud mapping infrastructure (CloudMade, OpenStreetMaps, Google Maps).  But all
the big players share a few things in common- they are all based on the Spherical Mercator[^spherical_mercator]
projection and they are all based on a uniform grid overlaid upon that projection.  In other words, (absent licensing
restrictions), you can use ModestMaps with Google Maps tiles.  Or Leaflet with OpenStreetMaps.  The URLs (and the URL
patterns) are similar; there is a difference between OpenStreetMaps and Google Maps in that the y-axis[^y_axis] is
counts up from the bottom in one and from the top in the other, but generally they are all built on the same model.

[^spherical_mercator]: The Earth is not, as many of us have been taught, a globe or a sphere.  It's wider at the equator
than it is at the poles, making it an _oblong spheroid_.  In fact it's not actually even an oblong sphereoid- it's
not regular in that description either.  As we describe the Earth more and more accurately, we use different
[datums](http://en.wikipedia.org/wiki/Datum_(geodesy)) to better describe the unique shape of the earth.  The _spherical_
in Spherical Mercator means that we simplify everything by presuming that the Earth is a sphere.  My high school physics
teacher would be proud.

[^y_axis]: I talked a lot above about latitude __not__ being the _y_ axis- why am I using _y_ axis here?  It's a subtle
point that took me a while to really understand- projection is taking a location on a globe and placing it on a
cartesian plane. If we are talking about a point on the globe it's latitude.  If we are talking about a spot on a _map_,
a two-dimensional representation, it's the _y_ axis as we suspected all along.

## Leaflet.js

For this project we chose [Leaflet.js](http://leafletjs.com) as our mapping library.  It works well with a number of
different tile sources, it's pretty lightweight, mobile-friendly, and very extensible.  Other alternatives were [Modest
Maps](http://modestmaps.com/), [OpenLayers](http://openlayers.org/), and [PolyMaps](http://polymaps.org/).  OpenLayers
is large and complex, doing a lot that we don't need.  Development on Modest Maps and PolyMaps seems to be slow or
stalled, but Leaflet.js seems to have a pretty vibrant ecosystem and good documentation.

Integration with Leaflet takes the form of a couple Leaflet modules that can be added to a Leaflet map as a layer.  One
adds a 2D Canvas overlay, the other adds a WebGL overlay.  These modules are simple- they simply work with the
containing map to ensure correct alignment as Leaflet moves the underlying DOM elements on pan actions. It also exposes
information about the current visible bounds of the map, the zoom level, and a function to project latitude and
longitude to screen coordinates.

We coordinate with the map by way of event callbacks on the Layer object.

```javascript
   L.webglLayer({animationLoop: true})
        .on('initialize', function(e) {
            for (var mgr in drawing_managers) {
                drawing_managers[mgr].initialize(e.ctx());
            }
        })
        .on('draw', function(e) {
            for (var mgr in drawing_managers) {
                drawing_managers[mgr].bounds(leaflet.getPixelBounds())
                    .zoom(leaflet.getZoom())
                    .draw(e.ctx());
            }
        })
        .addTo(leaflet);
```

The `initialize` event gives an opportunity to set up resources before entering the draw loop.  The `draw` event will
be called any time the map is panned or zoomed.  If the attribute `animationLoop` on the constructor argument is truthy,
a `requestAnimationFrame` loop will be set up and the `draw` callback will be called on every tick.

Both the `initialize` and `draw` callbacks will be called with a single object with the following attributes:

| Attribute | Value Description                                                                                      |
| --------- | ------------------------------------------------------------------------------------------------------ |
| canvas    | The canvas DOM element                                                                                 |
| ctx       | The canvas drawing context (as returned by `getContext('2d')` or `getContext('webgl', args)`           |
| latlon2xy | Function that takes a latitude and a longitude and returns an array of `[x, y]` in screen coordinates. |
| xy2latlon | Function that takes an _x_ and a _y_ in screen coordinates and reverse-projects them to `[lat, lon]`       |
| zoom      | Function that returns the current zoom level of the map.                                               |

## Drawing a Map With TileMill

Now we have the ability to render a map using tiles hosted somewhere else, but the real gravy is to be able to build
our own maps, with our own colors and with the data that we select.  To do that, the easiest tool we can use is
[TileMill](https://www.mapbox.com/tilemill/).  TileMill Allows us to pull in geographic information, style it in a
syntax very similar to CSS (CartoCSS) and export the resulting map in a number of formats.

One of the most difficult parts of building a map is finding the right data.  The best easy source of data
is the [Natural Earth](http://naturalearthdata.com) project- a compiled list of both physical and cultural features that
are available under a public domain license.  The data is available as a mix of shapefiles and GeoTIFFs.  A shapefile
is a form of spatial database for vector data.  This is the format in which you will find country borders, city centers,
and coastlines.  A GeoTIFF is a TIFF file with embedded projection, datum information[^datum], and extents that allow
it's pixels to be spatially positioned.  This is the format in which you will find heightmaps for topography and
bathymetry, ground coloring, or continuously varying data (a heatmap or vegetation coverage map would likely be available
as a GeoTIFF).

[^datum]: When making a map, this can be really big pitfall.  Data can be found in all sorts of different geographic
reference systems, and they aren't directly compatible- they have to be translated back and forth.  When you find a
shapefile, make sure you make note of the datum- TileMill can fix it for you, but you have to tell it what to expect.
Data from Natural Earth is all in WGS84 which is something of a web mapping default and it's what TileMill expects,
but it can make for a nasty surprise.  For example another big source of data is census data, known as TIGER/Line.  That
data uses a datum called NAD83. These datum are also known by an 'EPSG' entry number- WGS84 is EPSG:4326 and NAD83 is
EPSG:4269.

## Using Our Tile Set

After pulling data into TileMill and styling it as we want it, we need to deploy it so that we can use it from Leaflet.
Leaflet manages figuring out which tiles to request from the server, but how do we get the tiles onto the server? In
theory we could just generate all the tile images we need and copy them over into an image directory, but the sheer
number of tiles we need to generate becomes a performance issue- 21,824 tiles are in our initial tile set that covering
only zoom ranges 3 through 7.

A better solution would be to deploy a dedicated tile server such as [TileStache](http://tilestache.org) which renders
tiles on the fly and then caches them.  That's more infrastructure though, and we want to keep this simple.

We find an answer in one of TileMill's export formats- mbtiles.  It's a simple format that takes each of the tile images and
inserts it into a blob column in a sqlite database.  The tile will be requested by a URL of the format
{x}/{y}/{z}.{image format}, where {x} is the _x_ coordinate of the tile, {y} is the _y_ coordinate, and {z} is the zoom
level.  With mbtiles, _x_, _y_, and _z_ are columns- the request then maps to the sql:

```sql
select tile_data from tiles where tile_row = ? and tile_column = ? and zoom_level = ?
```

We can add a simple controller to our Spring web application to take a request for tile coordinates and find it in the
sqlite database.  Since sqlite databases are just regular files, we created a directory to hold different styles and maps-
drop a mbtiles file into that directory and you have a new map style available in Leaflet.

Here's our map, being served up with TileMill and our controller:

![Basic Basemap](/diagrams/basemap.png)

# Drawing Dots

Now we have a custom map and a canvas to draw on.  It's time to draw some dots.  First, a diversion- why not use SVG?

The answer lies in two rendering methodologies- [retained mode rendering](http://en.wikipedia.org/wiki/Retained_mode)
and [immediate mode](http://en.wikipedia.org/wiki/Immediate_mode).  When you use retained mode, you build a model or
tree of models that represent the structure of your drawing. You then iterate over that tree of models and convert it
to a visual representation.  As programmers we describe the model and let it draw itself.

This is very powerful and very flexible- we can work at a higher level of abstraction and build static representations
that take care of themselves.  Instead of calling 'fill a circle with `#0000ff` at `(300, 200)` with a radius of `50`'
on each draw call, we can specify 'there is a circle.  It's center is at `(300, 200)`.  It has a radius of `50`.
Everything else is a default', and every time the page is drawn, that circle will be there until we remove it.

The cost that we pay for that power is overhead- we have to maintain the model and keep it in sync.  In the case of SVG,
it's also a relatively heavy-weight model as each graphical element is a node in the DOM, and the DOM is integrated with
the DOM of the page that contains it.  As a result, every time we add or remove an element it takes the computer time
and effort, and it causes us to have to redraw other elements that may have been affected by our change.

Contrast this with the 'immediate mode' of rendering- the 'fill a circle with blue' above.  In this case the drawing is
essentially a bitmap- when the blue circle is drawn, the pixels contained within the circle are changed to blue and all
state is discarded.  It takes more management, but updates have a lower overhead.

For this visualization, we are trying to visualize all of our display add impressions (currently up to 450
events/second), page views (around 250/second), and ad clicks (a few tens per second).  Modifying the DOM that often is
going to be impractical.  We also need to leave time to parse, process, and filter all ~800 or so events every second.

We also need to watch our memory churn- Javascript is a single-threaded, garbage-collected language.  When we run up
against a memory threshold, everything stops until memory is reclaimed.  When this happens, the user sees 'jank'- a
stuttering and unpleasant rendering where everything seems jerky.

Even just an immediate-mode canvas may not be enough.  But this isn't a problem that's unique only to trying to put dots
on graphs- visualizing lots of rapidly-changing points of data has been a large focus of software development for
decades- in games and scientific visualization.

## A Background in WebGL

OpenGL has been a standard technology for games and visualization for many years.  Specialized hardware has been
designed and built in concert with the API to accelerate processing and rendering of data.  A graphics card is designed
to be massively parallel- my current laptop has 40 cores, and some cards have more than 350 cores.  OpenGL has been
evolved to process massive amounts of data quickly and to process that data into images at lightning speed.  When mobile
devices such as the iPhone started getting prevalent, Khronos (the OpenGL working group) developed a stripped-down
version of OpenGL for mobile devices called OpenGL ES.  They then revised desktop OpenGL to better match the rendering
pipeline as implemented on video cards (relying on program fragments running on the cards).  After that redesign came
OpenGL ES 2.0 to bring the mobile devices in line with new desktop practices.  Then that got ported to the web.

### What is WebGL?

WebGL is basically OpenGL ES 2.0 with a Javascript API.  The advantage to us is that it allows us to offload a lot of
the data visualization to the video card, allowing it to be processed in a massively parallel fashion rather than
relying on the single-threaded behavior of the rest of our Javascript runtime.  To do this we need to learn a new way of
processing graphics.

### `gl.drawArrays()`

In both of the examples above, we dealt with the circles we are drawing individually.  With SVG we add a `<circle>` node
to the DOM and style it to place it.  With Canvas we fill an arc. With WebGL we put together an array of data that we
want to render, ship it _en masse_ to the video card and convert them to an image there.

First, we need to set up a _rendering pipeline_[^webgl_lernings].  To do this, we have to declare a buffer to hold
our data and  'attach' it to the current context. We then need to supply data to that buffer.  We need to compile a
_vertex shader_  and a _fragment shader_, we need to bind both together into a _program_ that will actually run on the
video card's render cores, and then we need to attach it to the current context.  We then need to set the context for
the next run of the program using _uniform variables_.  We specify how to map _attribute variables_ to vertexes, and
finally with  the state prepared we can call `gl.drawArrays()` and the whole pipeline is distributed and set in motion.

[^webgl_lernings]: This is obviously a comically simplified version of WebGL with large portions that we aren't using
for this example missing (including features that were used in the prior prototype). The
[WebGL Programming Guide](http://www.amazon.com/WebGL-Programming-Guide-Interactive-Graphics/dp/0321902920) is a good
place to start, as is the
[OpenGL ES 2.0 Programming Guide](http://www.amazon.com/OpenGL-ES-2-0-Programming-Guide/dp/0321502795).

### Vertex Shaders and Fragment Shaders

The program running on the video card is made up of two components- a _vertex shader_ and a _fragment shader_.
What we are doing in the shaders is very analogous to the geographic projections we discussed earlier.  We take
coordinates in our program's coordinate system and we translate and project them into the OpenGL coordinate system.
We do this in two steps. First, we use a vertex shader to take a point in the user coordinate
system and we return a location in the OpenGL coordinate system. Second, the system correlates the vertex with the other
vertexes in the primitive it belongs to (the other two vertexes for a triangle, the other one for a line, or none for
a point), figures out what pixels in the image may[^frag-vs-pixel] change based on the primitive. It then calls
the fragment shader for each pixel which assigns the color or discards it.

[^frag-vs-pixel]: It's called a fragment shader and not a pixel shader because the output of a fragment shader might
not actually change a pixel.  As a simple example, OpenGL will take a look at the z-axis value of the pixel- if it
would be occluded by another fragment in the buffer the pixel will be discarded.

### Uniforms, Varyings, and Attributes

We coordinate with and between these shaders using three types of variables- uniforms, varyings, and attributes.

A uniform is a value that is the same for every vertex and every fragment- it is essentially a global variable for the
shader program.  We use these to coordinate between the hosting program and the shader program.

A varying is used for coordinating between a vertex shader and a fragment shader.  For primitives with several vertexes
the value read by the fragment shader will be interpolated between it's associated vertices.  For example if a fragment
is at the midpoint of a line one vertex of which sets a varying named `vColor` to `vec3(1.0, 0.0, 0.0)` [^glsl-types] and
the other vertex sets `vColor` to `vec3(0.0, 1.0, 0.0)`, the fragment will read `vColor` as `vec3(0.5, 0.5, 0.0)`.

The final type of variable is an attribute- attributes are per-vertex values that are supplied as the input to the
OpenGL pipeline.

[^glsl-types]: GLSL doesn't do conversion of types for you- and that includes the conversion of an int to a float as we
have come to expect from the C family of languages.  If you specify a `vec3(1, 0.0, 0.0)`, GLSL will refuse to compile
because of the mixed types. While this makes sense, it goes contrary to a behavior we may expect from other programming
languages.  It has been the source of an inordinate amount of cursing as I have learned to use GLSL effectively.

## Plotting Points on a Map

OK, now that we have some background on maps and some background on WebGL, let's put it all together and put some markers
on a map.

### Requirements (differing colors, size, shrink, opacity, fade)

The starting point of this map was a prior POC. That iteration used a technique called 'ping-ponging'. It used
two off-screen buffers and a two-dimensional canvas.  As events came in, they would be rendered immediately to the 2d
canvas. On each animation frame, the previous frame (stored in a framebuffer) was copied to a second framebuffer with a
GLSL program that blurred it.  The contents of the 2D canvas would then be written on top of the blurred frame.  The
frame was then copied to the screen.  The 2D canvas and the first framebuffer were then cleared, and the second
framebuffer became the source for the next iteration.

That approach had some fatal limitations. First, that the map didn't zoom.  It was set to a view of the United States
using an [Albers Equal Area](http://en.wikipedia.org/wiki/Albers_projection) projection that preserved the relative
sizes and proportions of regions, but it didn't allow for focusing on a region.  Second, there wasn't any ability to
vary the representations of different plotted traffic.  Since all of the markers were 'degraded' together as an image,
it wasn't possible to make more important events like ad clicks more prominent than less important events like ad
impressions so they were swallowed up in the noise.  Additionally, treating the marker overlay as an image didn't work
well if we added interaction- if the map were panned, blank areas would be moved onto the screen.  If it were zoomed the
markers would no longer match up with their geographic locations.

The solution was to move away from the hybrid 2D Canvas/WebGL image processing model and move to OpenGL vertex arrays.
Each marker type is given it's own buffer of locations and the shader program takes arguments through uniform variables
to vary the color, size, whether it shrinks over time, it's inital opacity, and whether it fades to transparent.

### Projecting Events

We have spent an awful lot of time working up to this, but here's where we tie it all together.  We know that we can't
treat each event as a momentary instance and forget about it- since the marker may move around the screen, may change
in size based on the map zoom, and may fade, we need to keep some information about the event for as long as it's on
the screen.  We want to keep the barest minimum of data we can, though!  We can distill the required data down to three
pieces- the location of the event, the time of the event, and the type of event.  We can define all of the other
properties as functions of those three attributes.  How that plays out is a little subtle, however.

The screen location of a marker may change at any time- any time a user touches the screen the map we are overlaying
changes it's bounds, which changes the portion of the earth that we are displaying.  It doesn't make any sense to
loop through in the slow single-threaded Javascript context and blocking the UI thread to project them from their
geographic coordinates to screen coordinates.  This is particularly true since many of the points may not even actually
be plotted- if the map is zoomed to view Denver, the user will never see events in Wichita.

The solution to this lies in what a geographic projection is and why we spent so long in going over it earlier; it's
just a function that takes geographic coordinates and maps them to a cartesian plane.  A vertex shader similarly takes
coordinates in a programmer-defined space and projects them into a screen space- defined in OpenGL as a range between
-1.0 and 1.0 along each axis.  Anything that falls outside of that range is quickly and efficiently discarded.  So we
can simply define our programmer-defined space to be a geographic space of spherical coordinates, and our projection
function to be the Spherical Mercator projection interpolated within map bounds at the time of render.

Here is the Spherical Mercator projection, with scaling for zoom levels,  as implemented in the vertex shader:

```glsl
float scale_factor(float zoom) {
    float result = 256.0 * pow(2.0, zoom);
    return result;
}

vec2 transform(vec2 point, float zoom) {
    vec2 result = vec2((((0.5 / M_PI) * point.x) + 0.5), ((-0.5 / M_PI) * point.y + 0.5)) * scale_factor(zoom);
    return result;
}

vec2 project(vec2 point) {
    vec2 result = vec2(point.x * DEG_TO_RAD, log(tan((M_PI / 4.0) + (point.y * DEG_TO_RAD ) / 2.0)));
    return result;
}

void main() {
    vec2 transformed = transform(project(a_coords.xy), u_zoom);
    //...
}
```

With this in place, we simply specify our x-values as degrees longitude and our y-values as degrees
latitude[^xy-ordering].

[^xy-ordering]: One source of constant frustration is that geographic coordinates are specified in lat-lon order,
meaning that north-south comes first, east-west second.  Cartesian coordinates are specified in x, y.  As latitude
corresponds to y and longitude corresponds to x, there is a lot of opportunity for error.

This gets us much of the way there, but we can't ignore z-values.  Since we are drawing all of these points at once,
if we don't assign a z-value they will all be the same, which turns into a situation called
'[z-fighting](http://en.wikipedia.org/wiki/Z-fighting)' as a marker will be drawn under another marker one frame and
over it the next. This becomes a really distracting flicker when it's animated.

We expect that the most recent markers should be on top of older markers, and that as new markers come in they cover
those that were there before.  In order to accomplish this we pass in our event time in the place of a normal z-value.
If we pass in the current time and establish a time window, we can map event times to our z-value.  In our case each
marker fades out in a few seconds, so we can assert that we won't even display an event older than a minute.  We can
then map the time range of 'now - one minute' to 'now' to the z-values -1.0 to 1.0.  Now we have it so that the most
recent events are always at the top of the stack- exactly what we expect.

### Making Circles

That makes displaying markers work pretty well.  But we are displaying points, and by default points in OpenGL are
square. This looks horrible.  We need another trick to turn them into circles.

In a fragment shader we can discard any fragments that we don't want.  We can use this to turn our squares into circles.

```glsl
    float distance = distance(gl_PointCoord, vec2(0.5, 0.5));
    if (distance > 0.5) {
        discard;
    }
```

In the fragment shader, `gl_PointCoord` is a vec2 normalized to the range `[0.0, 1.0]`, so the center is at `[0.5, 0.5]`
no matter what the size of the point is in screen coordinates. We take a look at the distance from the center to the
current location, and if it's more than 0.5 (the distance from the center to an edge vertically or horizontally)
we discard it.

### Fuzzing Borders

That's a _lot_ better, but the borders on the circles are full of jaggies- by just discarding pixels we have a sharp
edge.  Instead of just discarding, let's ramp the alpha to the closer you get to the edge.

```glsl
    // This value 'fuzzes' the edges a bit to make the circles look better
    float falloff_modifier = clamp(pow(1.8 - distance * 2.0, 2.0), 0.0, 1.0);
    gl_FragColor = vec4(u_color.rgb, falloff_modifier);
```

The actual code takes into account an original alpha (`u_color.a`[^swizzleing]) and a potential fade, but this code
gives us nice, smooth circles.

[^swizzleing]: One clever bit about the GLSL language is that you can refer to elements of a vector type by letter codes.
Generally a vec4 represents either a location (`xyzw`) or a color (`rgba`).  You can refer to the first element by
`color.r`, the third element by `color.b` and a vec2 containing both by `color.rb`.  You can even change order-
`color.argb` is perfectly valid and will return a vec4 with the elements placed in that order, and `color.aaaa` will
return a vec4 with four copies of the fourth value.  You can even swap the sets (though you can't mix them): `color.x`
will give you the red value.  But don't do that.

# The 'Finished' Product

Now we have a map that we can pan and zoom- in our office we have it running on a 55" television with an infrared touch
overlay.  We can now style the different types of events individually, changing color, size, fading and shrinking, and
we can keep up with our traffic rate- at times exceeding 1500 events per second and still maintain a 60 frame/second
draw rate.  We actually still have a lot of breathing room, too- according to Chrome's development tools, rendering a
frame takes consistently takes less than 2ms on my MacBook Pro.  You can see the video in full motion
[here](https://vimeo.com/89335767).

![Map Demo](/diagrams/map_demo.png)
