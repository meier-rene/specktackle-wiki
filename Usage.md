# Example
Once the SpeckTackle library and its CSS are properly referenced in the head of the target
HTML website, different charts can be created. SpeckTackle provides pre-defined chart types 
for IR, MS, NMR, 2D NMR, and general time series data. The layout and mouse behaviour of 
each chart type is defined by de facto standards and concern the x- and y-axes (placement/direction), 
zoom behaviour (box/range-zoom), color schemes, and representation of data points (impulse/line/point). 

Expected options such as a chart title, x- and y-labels, an interactive legend, chart margins, and 
signal labels can be set on chart creation in a cascading fashion before a data set is assigned 
to the chart. 

SpeckTackle accepts input in JSON format either directly or through Ajax. Similar to the pre-defined 
chart types, data handlers are implemented to reduce library set up to a minimum. A data handler 
is used to describe the structure of input data and to deal with data load and removal events 
after it is is associated (bound) to the chart. 

As a general rule, all interaction between a chart and raw data is mediated through a data handler, 
which keeps track of added data series and their properties. Multiple data series (overlays) 
are supported with the ability to highlight an individual data series via its legend key.

For more complex examples than the code snippet listed below, please see the **index.html** website in
the [home directory](https://bitbucket.org/sbeisken/specktackle/src) of the project, which contains 
multiple examples for different chart types, data load/removal events, and annotations.

```
#!html+django

<body>
    <div id="wrap">
        <div id="stgraph" class="stgraph"></div>
    </div>
    
    <script type="text/javascript">
        var chart = st.chart          // new chart
                .ms()                 // of type MS
                .xlabel("m/z")        // x-axis label
                .ylabel("Abundance"); // y-axis label
        chart.render("#stgraph");     // render chart to id 'stgraph'
        
        var handle = st.data          // new handler
            .set()                    // of type set
            .ylimits([0, 1000])       // y-domain limits
            .x("peaks.mz")            // x-accessor
            .y("peaks.intensity");    // y-accessor
            
        // bind the data handler to the chart
        chart.load(handle);
        // load the spectrum and annotations for Uridine
        handle.add("15814.json");
    </script>
</body>
```

## Data
The two default data structures currently supported are 'sets' and 'arrays'. Default data handlers
simplify data load and process data for visualisation, e.g. bin the data or assign annotations to
individual data points. Data needs to be in JSON format and can be provided either via URLs pointing
to a data source or directly as array.

A 'set' is defined as having either explicit x- and y-values in its JSON input in pairs ...

```
#!json

{
   "spectrumId":"MS",
   "mzStart":80.8,
   "mzStop":263.2,
   "peaks":[
      {
         "mz":80.8,
         "intensity":17.0
      },
      {
         "mz":85.2,
         "intensity":3.0
      },
      {
         "mz":92.1,
         "intensity":454.0
      }
   ]
}
```
```
#!js+cheetah

var handle = st.data          // new handler
    .set()                    // of type set
    .ylimits([0, 1000])       // y-domain limits
    .x("peaks.mz")            // x-accessor
    .y("peaks.intensity");    // y-accessor
```

... or as two separate arrays with x- and y-values.

```
#!json

{
    "spectrumId": "PRIDE",
    "mzRangeStart": 72.0838,
    "mzRangeStop": 1295.5687,
    "mzArray": [
        157.13047,
        235.1217,
        334.190579,
        387.286451
    ],
    "intenArray": [
        3611,
        8168,
        8026,
        9767
    ]
}
```
```
#!js+cheetah

var set = st.data.set() 
    .xlimits(["mzRangeStart", "mzRangeStop"])
    .x("mzArray")             
    .y("intenArray")      
    .title("spectrumId");
```

An 'array' is defined by a single array of y-values, complemented by defined x-value start 
and stop values.

```
#!json

{
    "id":"NMR",
    "xLabel": "ppm",
    "yLabel": "Intensity",
    "xMax": 10.8032,
    "yMax": 4.187424E8,
    "xMin": -1.21169,
    "yMin": -2.8624496E7,
    "data": [
        -97423.0,
        -105938.0,
        -118276.0,
        -124206.0,
        -118488.0
    ]
}
```
```
#!js+cheetah

var array = st.data.array()     // data type (array)
    .xlimits(["xMin", "xMax"])
    .ylimits(["yMin", "yMax"])
    .y("data")                  // one dimensional array
```

## Annotations
Annotations and tooltips for data point selection events are supported through the concept of 
annotation types. Implemented annotation types encompass:

* textual annotations that are drawn onto the chart 
* textual/structural tooltip annotations. 

Whereas textual annotations are simply drawn besides their target data points, 
tooltip annotations are specified as key-value pairs: in the first case, the key-value 
pairs are character strings that are displayed as list in the form '<key>:<value>'. 
In the second case the value of each pair is treated as URL to a MDL Molfile, which 
contains the molecular structure to be displayed, and resolved accordingly.

The following two code listings demonstrate the concept. The structure of the annotation 
JSON file needs to be defined in the data handler before data can be loaded.

```
#!html+django

<body>
    <div id="wrap">
        <div id="stgraph" class="stgraph"></div>
    </div>
    
    <script type="text/javascript">
        var chart = st.chart          // new chart
                .ms()                 // of type MS
                .xlabel("m/z")        // x-axis label
                .ylabel("Abundance"); // y-axis label
        chart.render("#stgraph");     // render chart to id 'stgraph'
        
        var handle = st.data          // new handler
            .set()                    // of type set
            .ylimits([0, 1000])       // y-domain limits
            .x("peaks.mz")            // x-accessor
            .y("peaks.intensity");    // y-accessor
            
        // annotation structure
        handle.annotationColumn(st.annotation.ANNOTATION, "Ions");
        handle.annotationColumn(st.annotation.TOOLTIP, "Instrument");
        handle.annotationColumn(st.annotation.TOOLTIP, "CE");
        handle.annotationColumn(st.annotation.TOOLTIP, "Ion mode");
        handle.annotationColumn(st.annotation.TOOLTIP_MOL, "Fragment");
        
        // bind the data handler to the chart
        chart.load(handle);
        // load the spectrum and annotations for Uridine
        handle.add("15814.json", "15814_anno.json");
    </script>
</body>
```

The annotation JSON file contains two required columns by default: 

* the first column defines the group. Multiple groups are permitted and are listed for 
selection on data load. 
* the second column defines the lookup value in the x-domain. Subsequent columns 
must match the structure described to the data handler. In the case of a one dimensional array
where no x-values are provided, the lookup value in the x-domain becomes a lookup index, i.e.
the position of the data point in the array.

```
#!json

[
    ["Fragments", 113.0, "***", "LC-ESI-QQ", "10 V", "positive", "frag_113.mol"],
    ["Fragments", 245.2, "[M+H]", "LC-ESI-QQ", "10 V", "positive", "frag_245.mol"],
    ["Fragments", 92.1, "***", "LC-ESI-QQ", "10 V", "positive", "frag_92.mol"]
]
```
