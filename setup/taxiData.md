---
layout: page
title: Using the Taxi Data Streams
permalink: /setup/taxiData.html
nav-title: Using the Taxi Data
nav-parent_id: setup
nav-pos: 20
---

The [New York City Taxi & Limousine Commission](http://www.nyc.gov/html/tlc/html/home/home.shtml) provides a public [data set](https://uofi.app.box.com/NYCtaxidata) about taxi rides in New York City from 2009 to 2015. We use a modified subset of this data to generate streams of events about taxi rides.

### 1. Download the taxi data files

You can download the taxi data files by running the following commands

~~~~
wget http://training.ververica.com/trainingData/nycTaxiRides.gz
wget http://training.ververica.com/trainingData/nycTaxiFares.gz
~~~~

It doesn't matter if you use wget or something else to fetch these files, but however you get the data, **do not decompress or rename the `.gz` files**.

### 2. Schema of Taxi Ride Events

Our taxi data set contains information about individual taxi rides in New York City.
Each ride is represented by two events: a trip start, and a trip end event.
Each event consists of eleven fields:

~~~
rideId         : Long      // a unique id for each ride
taxiId         : Long      // a unique id for each taxi
driverId       : Long      // a unique id for each driver
isStart        : Boolean   // TRUE for ride start events, FALSE for ride end events
startTime      : DateTime  // the start time of a ride
endTime        : DateTime  // the end time of a ride,
                           //   "1970-01-01 00:00:00" for start events
startLon       : Float     // the longitude of the ride start location
startLat       : Float     // the latitude of the ride start location
endLon         : Float     // the longitude of the ride end location
endLat         : Float     // the latitude of the ride end location
passengerCnt   : Short     // number of passengers on the ride
~~~

**Note:** The data set contains records with invalid or missing coordinate information (longitude and latitude are `0.0`).

There is also a related data set containing taxi ride fare data, with these fields:

~~~
rideId         : Long      // a unique id for each ride
taxiId         : Long      // a unique id for each taxi
driverId       : Long      // a unique id for each driver
startTime      : DateTime  // the start time of a ride
paymentType    : String    // CSH or CRD
tip            : Float     // tip for this ride
tolls          : Float     // tolls for this ride
totalFare      : Float     // total fare collected
~~~

### 3. Generating Taxi Ride Data Streams in a Flink program

**Note: The exercises already provide code for working with these taxi ride data streams.**

We provide a Flink source function (`TaxiRideSource`) that reads a `.gz` file with taxi ride records and emits a stream of `TaxiRide` events. The source operates in [event-time]({{ site.docs }}/dev/event_time.html). There's an analogous source function (`TaxiFareSource`) for `TaxiFare` events.

In order to generate these streams as realistically as possible, events are emitted proportional to their timestamp. Two events that occurred ten minutes after each other in reality are also served ten minutes after each other. A speed-up factor can be specified to "fast-forward" the stream, i.e., given a speed-up factor of 60, events that happened within one minute are served in one second. Moreover, one can specify a maximum serving delay which causes each event to be randomly delayed within the specified bound. This yields an out-of-order stream as is common in many real-world applications.

For these exercises, a speed-up factor of 600 or more (i.e., 10 minutes of event time for every second of processing), and a maximum delay of 60 (seconds) will work well.

All exercises should be implemented using event-time characteristics. Event-time decouples the program semantics from serving speed and guarantees consistent results even in case of historic data or data which is delivered out-of-order.

#### Checkpointing

Some of the exercises will use `CheckpointedTaxiRideSource` and/or `CheckpointedTaxiFareSource` instead. Unlike `TaxiRideSource` and `TaxiFareSource`, these variants are able to checkpoint their state.

#### Table Sources

Note also that there are `TaxiRideTableSource` and `TaxiFareTableSource` table sources available for use with the Table and SQL APIs.

### How to use these sources

#### Java

{% java %}
// get an ExecutionEnvironment
StreamExecutionEnvironment env =
  StreamExecutionEnvironment.getExecutionEnvironment();
// configure event-time processing
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

// get the taxi ride data stream
DataStream<TaxiRide> rides = env.addSource(
  new TaxiRideSource("/path/to/nycTaxiRides.gz", maxDelay, servingSpeed));
{% endjava %}

#### Scala

{% scala %}
// get an ExecutionEnvironment
val env = StreamExecutionEnvironment.getExecutionEnvironment
// configure event-time processing
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

// get the taxi ride data stream
val rides = env.addSource(
  new TaxiRideSource("/path/to/nycTaxiRides.gz", maxDelay, servingSpeed))
{% endscala %}

There is also a `TaxiFareSource` that works in an analogous fashion, using the nycTaxiFares.gz file. This source creates a stream of `TaxiFare` events.

#### Java

{% java %}
// get the taxi fare data stream
DataStream<TaxiFare> fares = env.addSource(
  new TaxiFareSource("/path/to/nycTaxiFares.gz", maxDelay, servingSpeed));
{% endjava %}

#### Scala

{% scala %}
// get the taxi fare data stream
val fares = env.addSource(
  new TaxiFareSource("/path/to/nycTaxiFares.gz", maxDelay, servingSpeed))
{% endscala %}

{% next %}
