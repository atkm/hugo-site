+++
title = "Some data engineering reading"
date = "2018-08-21"
+++

## Model Serving
- [FLIP-23](https://cwiki.apache.org/confluence/display/FLINK/FLIP-23+-+Model+Serving).
The document also discusses implementing model training as well as model serving.
Two linked documents---"Flink ML Roadmap" and "Flink-MS"---are also worth reading.
- Boris Lublinsky's book "Serving Machine Learning Models", and [talk](https://www.youtube.com/watch?v=YmrCv5onW_E).

## Distributed Systems

- [Please Stop Calling Database Systems AP or CP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html).
- Kate Matsudaira on [distributed systems](http://www.aosabook.org/en/distsys.html)
- [Distributed Systems for Fun and Profit](https://martinfowler.com/articles/microservices.html)
- Martin Kleppmann's [book](http://shop.oreilly.com/product/0636920032175.do) and [interview](https://softwareengineeringdaily.com/2017/05/02/data-intensive-applications-with-martin-kleppmann/).
- [Use of Formal Methods at Amazon Web Services](http://lamport.azurewebsites.net/tla/amazon.html)
- [A tale of two clusters: Mesos and YARN](https://www.oreilly.com/ideas/a-tale-of-two-clusters-mesos-and-yarn)
- A visual explanation of Raft: [link](http://thesecretlivesofdata.com/raft/)

## Streaming Systems

### The Dataflow Model, Apache Beam, and Cloud Dataflow
- Tyler Akidau's [article](https://www.oreilly.com/ideas/the-world-beyond-batch-streaming-101) on the Dataflow model.
- Frances Perry's [talk](https://www.youtube.com/watch?v=3UfZN59Nsk8) covers the same topics as the aforementioned article by Tyler Akidau.
- [Big Data Processing at Spotify](https://labs.spotify.com/2017/10/16/big-data-processing-at-spotify-the-road-to-scio-part-1/)
- Hands-on exercises to try out the Dataflow model created by [dataArtisans](http://training.data-artisans.com/).
- Jay Kreps' [article](https://www.oreilly.com/ideas/questioning-the-lambda-architecture) "Questioning the Lambda Architecture".
- Martin Kleppmann [compares](https://samza.apache.org/learn/documentation/latest/comparisons/introduction.html) Samza to other stream processing systems.
- [Flink Blog](https://flink.apache.org/blog/): articles that discuss how exactly-once processing, checkpointing, joins, etc. are implemented in Flink.

### Kafka and Inverted Database
- Martin Kleppmann's [talk](https://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) "Turning the Database Inside Out with Apache Samza".
- Jay Kreps' articles: [It's Okay to Store Data in Kafka](https://www.confluent.io/blog/okay-store-data-apache-kafka/),
[The Log: What Every Software Engineer Should Know About Real-time Data's Unifying Abstraction](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)
- [Tutorial](http://www.michael-noll.com/blog/2014/08/18/apache-kafka-training-deck-and-tutorial/)
- Neha Narkhede on ["event sourcing"](https://www.confluent.io/blog/event-sourcing-cqrs-stream-processing-apache-kafka-whats-connection/)
- [Spotify's Event Delivery](https://labs.spotify.com/2016/02/25/spotifys-event-delivery-the-road-to-the-cloud-part-i/)
- Boerge Svingen, Publishing with Kafka at NY Times (SWE daily, Confluent blog)

## Data Pipelines
- [Explainer articles](https://www.dremio.com/library/#explainer) on Dremio website.
    Topics include data engineering, Apache Arrow, Data Warehouses, Data Pipelines, ETL Tools.
- Quizlet on Airflow: [link](https://medium.com/tech-quizlet/going-with-the-flow-how-quizlet-uses-apache-airflow-to-execute-complex-data-processing-pipelines-1ca546f8cc68).
- Rebuilding Yelp's Data Pipeline with Justin Cunningham (Data Engineering Podcast)
-  Danny Yuan on Real-Time, Time Series Forecasting @Uber: [link](https://www.infoq.com/podcasts/streaming-real-time-uber-time-series-danny-yuan)

## Microservices
- Josh Evans, A Netflix Guide to Microservices (InfoQ talk)
- Beyond Buzzwords: A Brief History of Microservice Patterns (Kyle Brown, IBM)
- Martin Fowler on [microservices](https://martinfowler.com/articles/microservices.html)

## Others
- William Morgan on Scaling Twitter (SWE daily)
- Josh Wills' [talk](https://www.youtube.com/watch?v=qNAw7mUgx34) at MLconf.
- Scaling Uber with Matt Ranney (SWE daily)
- Cassandra with Tim Berglund (SWE daily)
