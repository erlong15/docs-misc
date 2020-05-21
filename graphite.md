# Graphite. Carbon.

## What is whisper? How is stores data on HDD?

- Whisper - is a fixed-size database, similar in design and purpose to RRD (round-robin-database)
- Data points in Whisper are stored on-disk as big-endian double-precision floats.
  - Each metric is stored in its own file
  - Each file has a fixed size, which is regulated by retention policies 
  - Each data point is stored with its timestamp
  - Archives overlap time periods
  - All time-slots within an archive take up space whether or not a value is stored

## How retention policies work for whisper? Where do you set them?

- The configuration file **storage-schemas.conf** details retention rates for storing metrics. It matches metric paths to patterns, and tells whisper what frequency and history of datapoints to store.
- [storage-schemas.conf](https://graphite.readthedocs.io/en/latest/config-carbon.html#storage-schemas-conf)

```bash
[apache_busyWorkers]
pattern = ^servers\.www.*\.workers\.busyWorkers$
retentions = 15s:7d,1m:21d,15m:5y
```

## What is aggregation function for retention? What is X files factor?

- This file defines how to aggregate data to lower-precision retentions. 
- **xFilesFactor** should be a floating point number between 0 and 1, and specifies what fraction of the previous retention level’s slots must have non-null values in order to aggregate to a non-null value. The default is 0.5.
- **aggregationMethod** specifies the function used to aggregate values for the next retention level. Legal methods are average, sum, min, max, and last. The default is average.
- [storeage-aggregation.conf](https://graphite.readthedocs.io/en/latest/config-carbon.html#storage-aggregation-conf)

```bash
[all_min]
pattern = \.min$
xFilesFactor = 0.1
aggregationMethod = min
```

## How to change retention \ aggregation function \ x files factor for already existing whisper files?

- [whisper-resize](https://github.com/graphite-project/whisper#whisper-resizepy)

## How do you fetch data from carbon?

- [metric API](https://graphite-api.readthedocs.io/en/latest/api.html#the-metrics-api)
- [render API](https://graphite-api.readthedocs.io/en/latest/api.html#the-render-api-render)

## What is the metric series? Metric series list?

- [terminology](https://graphite.readthedocs.io/en/latest/terminology.html#term-series)
- series
  - A named set of datapoints. A series is identified by a unique name, which is composed of elements separated by periods (.) which are used to display the collection of series into a hierarchical tree. A series storing system load average on a server called apache02 in datacenter metro_east might be named as:
    - metro_east.servers.apache02.system.load_average
- series list
  - A series name or wildcard which matches one or more series. Series lists are received by functions as a list of matching series. From a user perspective, a series list is merely the name of a metric. For example, each of these would be considered a single series list:
    - metro_east.servers.apache02.system.load_average.1_min,
    - metro_east.servers.apache0{1,2,3}.system.load_average.1_min
    - metro_east.servers.apache01.system.load_average.*

## Which functions you can apply when retrieve the data?

- [functions](https://graphite.readthedocs.io/en/latest/functions.html#module-graphite.render.functions)

## Which port Carbon uses? Graphite?

- carbon
  - **[cache]**
    - LINE_RECEIVER_PORT (2003)
    - PICKLE_RECEIVER_PORT (2004)
  - **[relay]**
    - LINE_RECEIVER_PORT (2013)
    - PICKLE_RECEIVER_PORT (2014)
  - **[aggregate]**
    - LINE_RECEIVER_PORT (2023)
    - PICKLE_RECEIVER_PORT (2024)
- graphite
  - port (8080)