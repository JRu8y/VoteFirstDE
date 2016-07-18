<h2>Choropleth map</h2>

In these series of tutorials, I'm going to be producing popular journalism web-friendly charts. There are a many visuzaliation tools to help you create <a href="http://datawrapper.de/">charts</a>, and <a href="http://cartodb.com/">maps</a>. Google docs can also help you get started in data visualization. These tutorials are for journalists interested in more control over their visualizations than off the shelf tools, or a way to get started in D3. I will try to provide as much detail as possible, walking through each graphic step by step, from data preparation to final version. These are being written to help an average journalist with a chart or map accompaniment to their work. I will therefore ensure as much as possible a replicability and reusability, while making them as easy to understand as possible. 

These tutorials use <a href="http://d3js.org/">D3</a>, which is a framework that relies on scaled vector graphics to render visualizations. These tutorials will not require extensive javascript or programming knowledge as they use minimal code, and every facet that is customizable will be explained in detail. 

<a href="http://bl.ocks.org/darrenjaworski/5874133"><b>Tutorial I</b>: Pie chart, bar chart, and line chart.</a>

<a href="http://bl.ocks.org/darrenjaworski/5874214"><b>Tutorial II</b>: Choropleth map.</a>

<a href="http://bl.ocks.org/darrenjaworski/5874227"><b>Tutorial III</b>: Points on a map.</a>

<a href="http://bl.ocks.org/darrenjaworski/5874229"><b>Tutorial IV</b>: Slightly more advanced pie, bar and line charts.</a>

<a href="http://bl.ocks.org/darrenjaworski/5874233"><b>Tutorial V</b>: Slightly more advanced choropleth map.</a>

<a href="http://bl.ocks.org/darrenjaworski/5874236"><b>Tutorial VI</b>: Network graph.</a>

<hr>

<h3>TOPOJSON installation</h3>

For D3 mapping I prefer to use <a href="https://github.com/mbostock/topojson/wiki/Installation">TOPOJSON</a>. It's a JSON format that is significantly smaller than its GEOJSON equivalent. If you wish to generate your own JSON files from shapefiles or other geotagged data, you will need to install a couple of things. 

First you need to install Node.js using Homebrew. If you are already using Homebrew, skip a paragraph down. If you do not currently have Homebrew installed, open terminal on your Mac and past this code: 

```bash
ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
```

After Homebrew has been installed, type the following into terminal:

```bash
brew install node
```

With Node.js installed, you are ready to install TOPOJSON. Type the following into terminal: 

```bash
npm install -g topojson
```

I found that it takes a couple of minutes to install, be patient, it's worth it. Once it is finished installing, we are ready to generate JSON files for our choropleth and other maps. 

<b>A note on installation</b>: I and a few other users have had various issues installing topojson through the terminal. This is most like due to my unfamiliarity with bash. You will need developer command line tools to install everything. Those can be found at the <a href="https://developer.apple.com/downloads/index.action">Apple developer downloads</a> page.

<h3>Data preparation</h3>

For this tutorial we're going to be mapping some county-level census data for the state of Oklahoma. Let's get some data first and then we will collect the shapefile to combine it with. We are going to start at the US Census Bureau's <a href="http://factfinder2.census.gov/faces/nav/jsf/pages/searchresults.xhtml?refresh=t">"Fact Finder 2" website</a>.  We will be obtaining the percent of individuals living below the Federal poverty line. Here are the steps to obtain the correct CSV. 

1. Click on the button on the left which says "Geographies"
2. From the -- select a geographic type -- dropdown select county
3. From the -- select a state -- dropdown select Oklahoma
4. Click "All Counties within Oklahoma"
5. Click "ADD TO YOUR SELECTIONS" button
6. Click the close button on the top right of the Geographies box
7. Now click the button the left which says "Topics"
8. Select "People"
9. Select "Poverty"
10. Click the close button on the to right of the Topics box
11. Click the "SELECTED ECONOMIC CHARACTERISTICS" table with the 2011 ACS-5 year estimates
12. This will take you to the table viewer screen. Click "Download"
13. From the Download window, click "Data and annotations in separate files"
14. Once it's done "building" your file, click download

We now have county-level data for all 77 counties of Oklahoma, we need a shapefile to combine that data with. These files can be found in numerous places, but for this tutorial I will be using data directly from the <a href="http://www.census.gov/geo/maps-data/data/cbf/cbf_counties.html">US Census Bureau again</a>. County, and other shapefiles are contained in the TIGER products database. 

We need to generate a JSON file from the topology in the "co40_d00.shp" file using TOPOJSON <a href="https://github.com/mbostock/topojson/wiki/Command-Line-Reference">command lines</a>. You need to move the .shp and .dbf files to your user folder on your Mac. Open terminal and type the following:

```bash
topojson -o output.json --properties -- co40_d00.shp
```

At this point you will have a JSON file with the topology of the counties shapefile from the Census Bureau. I would recommend using a <a href="http://jsonprettyprint.com/">JSON formatter</a> to make the JSON easier to read and manipulate. We need to change the object designation in the JSON file. Open the file and find  "objects":{"co40_d00": ... Change co40_d00 to "counties" or another reference of the shapefiles type.

We can now combine data from the CSV. In the file named "ACS_11_5YR_DP03.csv" change the column heading "GEO.display-label" to id, and remove the words County, Oklahoma from each row using a simple find/replace search. Now we're ready to combine the column of poverty data with the JSON file. Looking through the metadata file you find that the percent of all families with income under the Federal Poverty Line is named "HC03_VC156".  Move the CSV file to your user folder.

Go back to terminal and type the following according to the TOPOJSON <a href="https://github.com/mbostock/topojson/wiki/Command-Line-Reference">command line reference</a>: 

```bash
topojson -o finaloutput.json -e ACS_11_5YR_DP03.csv --id-property=NAME -p belowfpl=+HC03_VC156 -- output.json
```

Now you have a fully ready TOPOJSON file. With the property belowfpl which was typed as a number. Your output file is named whatever you designate. I named it finaloutput.json for ease of reading. 

A word on TOPOJSON. The power of this encoding is amazing. With a little practice you'll believe me when I say that this is not difficult, nor is it time consuming. 

<h3>HTML</h3>

```html
<!DOCTYPE html>
<meta charset="utf-8">
<style>
	.county-border {
		fill: none;
		stroke: white;
	}

	.state-border {
		fill: none;
		stroke: black;
	}
	.axis text {
		font: 10px sans-serif;
	}

	.axis line, .axis path {
		fill: none;
		stroke: #000;
		shape-rendering: crispEdges;
	}
	
	#key	{
		position: absolute;
		top:90px;
		left: 50px;
	}
</style>
<body>
<script src="http://d3js.org/d3.v3.min.js"></script>
<script src="http://d3js.org/topojson.v1.min.js"></script>
<script src="choropleth.js"></script>
```

For this map a reference to topojson.v1.min.js will enable us to render the TOPOJSON files. I've already included the css for the boundaries and legend, those are fully customizable to your aesthetic and design principles. For this initial tutorial I chose to create a linear continuous gradient from a light color to a dark color. In the next tutorial on choropleths I will illustrate a threshold gradient and a simple mouseover. 

This is all the HMTL we need to render. On to the fun stuff!

<h3>choropleth.js</h3>

<b>Initial variables</b>

Declare height and width variables that will be used to size the map. These are really variables that you can adjust to work best within your page's limitations. 

I named four variables two of which, minimum and maximum, take on the properties of the dataset which was found to have a minimum value of 5 (rounded down) and 24 (rounded up). There is a way for D3 to determine the minimum and maximum extent of the data sets, but that will be left for a later tutorial.  The next two variables are the color hex codes that we choose for the lowest value and the highest value respectively. A great resource when choosing color palettes for choropleth maps is <a href="http://colorbrewer2.org">Color Brewer</a>.

```javascript
var width = 900, height = 500;

var minimum = 5, maximum = 24;

var minimumColor = "#BFD3E6", maximumColor = "#88419D";
```

<b>Color</b>

We will now encode a color scale from the variables we just declared. As you can see below, the wording is fairly close to what you'd expect. The domain is the numerical value of the data, and the range is the hex code designation for that range. 

```javascript
var color = d3.scale.linear().domain([minimum, maximum]).range([minimumColor, maximumColor]);
```

<b>Projection</b>

Next I need to turn your attention to some state plane coordinates and how to input your state's coordinates into D3 correctly. Here is the document we will be using as reference: <a href="http://www.ngs.noaa.gov/PUBS_LIB/ManualNOSNGS5.pdf">NOAA Manual, State Plane Coordinate System of 1983</a>.

On page 63 of the document, 73 of the PDF there is a reference table containing the State, central meridian and standard parallels as well as longitude and latitudes of the grid origin.  We will use this to correctly project your JSON.

As an example. For Oklahoma you will read the standard parallels for the north plane to be 35 34 and 36 46. Those are inputted as " 35 + 34 / 60, 36 + 46 /60 " into parallel.

For rotate we need grid origin which is 98 00 and 35 00. Those are inputted as " 98 + 00 / 60, -35 - 00 / 60 " into rotate. 

Scaling allows you to project the files at a custom scaling figure. 5300 is what I found worked best at this particular height and width. For your state and size of page, you'll need to adjust this scaling factor accordingly. 

```javascript
var projection = d3.geo.conicConformal()
	.parallels([35 + 34 / 60, 36 + 46 / 60])
	.rotate([98 + 00 / 60, -35 - 00 / 60])
	.scale(5300)
	.translate([width / 2, height / 2]);
```

We then define a path, and set a projection of our file using the variable we just defined. Then we are going to define the overall graphic svg and place it in the body of our HTML. 

```javascript
var path = d3.geo.path().projection(projection);

var svg = d3.select("body").append("svg").attr("width", width).attr("height", height);
```

<b>JSON and path rendering</b>

This is standard D3 JSON reading. First we define counties as a feature of the JSON. Notice particularly how I define things... " function(error, ok)..." and then later we see " feature(ok, ok.objects.county) " . If you notice a consistency in the references. Also we changed the name of the objects to "county" so that we can read them better than " co40_d00 ".

I designated each level with a comment specifying which it is. First we draw and color the counties themselves, then we provide borders, then a state border. 

The line of code that I want you will be keen on is " .style("fill", function(d) { return color(d.properties.belowfpl); }); " Within this line you can customize the variable that will be colored. I'm accessing the data by d.properties.belowfpl, again because that is what we designated as the poverty property. Whatever you name your coloration variable to be in your JSON, you will need to reference here. We call the variable color, which utilizes the scale we just set up and then color it by whatever value that specific property is for each county. 

```javascript
d3.json("finaloutput.json", function(error, ok) {
	var counties = topojson.feature(ok, ok.objects.county);

	//counties
	svg.append("g")
	.attr("class", "county")
	.selectAll("path")
	.data(counties.features)
.enter()
	.append("path")
	.attr("d", path)
	.style("fill", function(d) {
		return color(d.properties.belowfpl);
	});

	//county borders
	svg.append("path").datum(topojson.mesh(ok, ok.objects.county, function(a, b) {
		return a !== b;
	}))
	.attr("class", "county-border")
	.attr("d", path);

	//state borders
	svg.append("path").datum(topojson.mesh(ok, ok.objects.county, function(a, b) {
		return a === b;
	}))
	.attr("class", "state-border")
	.attr("d", path);

});

d3.select(self.frameElement).style("height", height + "px");
```

That will render a choropleth map at a county level colored by poverty data. We need a legend however so that we can estimate the amounts visually. 

<b>Legend</b>

As I demonstrated in a <a href="http://bl.ocks.org/darrenjaworski/5397362">separate example</a>, we will be creating a legend to append to the page for a visual reference. 

For the legend, we create a separate width (w) and height (h) variable to encode the rectangle that will be the legend. Since this is a vertical legend I encoded the var legend to to color the gradient accordingly. If you turn the legend horizontally, switch the values in order for the gradient to appear across the page. 

Lines of note are the ones starting " legend.append  ". Notice that the stop color for both is is the maximumColor and minimumColor. 

Another line of note is " var y " which contains the domain, again calling the variables we did the begin with. 

The last line that you need to draw your attention to is " key.append " at the very end. It reads " .text("% Below FPL") ". This is the text that will appear on the key itself, giving your numbers context. 

```javascript
var w = 140, h = 300;

var key = d3.select("body").append("svg").attr("id", "key").attr("width", w).attr("height", h);

var legend = key.append("defs").append("svg:linearGradient").attr("id", "gradient").attr("x1", "100%").attr("y1", "0%").attr("x2", "100%").attr("y2", "100%").attr("spreadMethod", "pad");

legend.append("stop").attr("offset", "0%").attr("stop-color", maximumColor).attr("stop-opacity", 1);

legend.append("stop").attr("offset", "100%").attr("stop-color", minimumColor).attr("stop-opacity", 1);

key.append("rect").attr("width", w - 100).attr("height", h - 100).style("fill", "url(#gradient)").attr("transform", "translate(0,10)");

var y = d3.scale.linear().range([200, 0]).domain([minimum, maximum]);

var yAxis = d3.svg.axis().scale(y).orient("right");

key.append("g").attr("class", "y axis").attr("transform", "translate(42,10)").call(yAxis).append("text").attr("transform", "rotate(-90)").attr("y", 30).attr("dy", ".71em").style("text-anchor", "end").text("% Below FPL");
```

That, ladies and gentleman, is a simple choropleth map with a continuous gradient. It's been my experience that these things usually don't render the first time I try to render them. Don't be discouraged. Exam your code for syntax errors and open the console of any browser to spot errors. There are always resources and examples to work from and reference, as well as direct contact with knowledgable people who are glad to help. This is everything you will need to render a simple choropleth map. In a later tutorial I will be walking you through more advanced techniques and added features for choropleths and maps. As usual you can find me on <a href="http://darrenjaworski.com">my site</a>, <a href="http://twitter.com/darrenjaws">twitter</a>, or <a href="http://oklahomawatch.org">Oklahoma Watch</a>.