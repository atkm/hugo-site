+++
title = "Stream Processing of NYC Taxi Data"
date = "2018-07-18"
draft = true
+++

## Model Serving

Topics:

- Standards: PMML (JPMML is a Java implementation of the standard), PFA (supposed to be a successor of PMML).
    Some ML systems (Spark, Tensorflow) have its own model format.
- Some models have a larger memory footprint than others.
    A KNN model, for example, has the size of the data it is trained on.
- Deployment options:
    + Option 1: Deploy as an http service;
    + Option 2: Deploy as an operator in a streaming system.
        If a very low latency is a requirement, deploying the model as an operator may be preferable, as issuing a http request for each data point is more expensive.
        In addition, when deployed as an operator, the model benefits from functions that the stream processing system provides, such as fault tolerance and resource management.
- Model update options:
    + Online learning.
    + Hot-loading a model.

Refs:

- [FLIP-23](https://cwiki.apache.org/confluence/display/FLINK/FLIP-23+-+Model+Serving).
The document also discusses implementing model training as well as model serving.
Two linked documents---"Flink ML Roadmap" and "Flink-MS"---are also worth reading.
- Boris Lublinsky's book "Serving Machine Learning Models", and [talk](https://www.youtube.com/watch?v=YmrCv5onW_E).

# The System

## Data

Pre-processing prototype in pandas; implemented in Spark.
In retrospect, it would have been more efficient to develop a prototype in a notebook environment using Spark (like the one offered by Databricks).

The AvroSerializable interface for Kafka.

### Taxi Data

### Weather Data

## Real-time Demand Pipeline

### Reverse Geocoding

Reverse geocoding is the process of converting a latitude-longitude pair to an address (e.g. 38.8977° N, 77.0365° W -> 1600 Pennsylvania Ave NW, Washington, DC 20500).
There is no reverse geocoding library with a small footprint.
Nominatim seems to be the most practical solution.
Another solution is to set up a PostGIS server.

One option is to include a Nominatim service in each predictor node.
Another is to create a stand-alone Nominatim service that predictor nodes query via http requests.

## Demand Prediction Pipeline

Confusion around exactly-once guarantee that Kafka Streaming makes => Kafka Streaming suppors exactly-once semantics.
The resulting PMML file is about 5MB.

## Trade-offs
- Reverse Geocode or Grids.
- When to join data
- Beam or Not Beam; Cloud Dataflow or Flink; the choice of streaming systems... There are so many products in this space, 
- Whether the prediction pipeline should have been implemented as a batch processor.
- Choice of the prediction model
- Using Kafka instead of a SQL database system.
- Which model serving strategy?
- The abstract class for classes that are written to Kafka topics.

## Ideas for Future Work

- Simulate more realistic ride and weather streams.
    The watermarking scheme needs improved.
    Imitate [this](https://github.com/dataArtisans/flink-training-exercises/blob/master/src/main/java/com/dataartisans/flinktraining/exercises/datastream_java/sources/TaxiRideSource.java) code.
- Use Confluent Schema Registry, which is included in Confluent Platform. The Platform also includes various connectors that are useful for our system.
- Any idea for interactions between the real-time and prediction pipelines? e.g. test the accuracy of prediction in real-time.
- Versioning of the prediction model.
- Travel time prediction pipeline
- What happens if the system needs to support regions other than NYC?
- What happens if the zip code scheme changes, e.g. 10013 expands.
- Any use for online learning?
