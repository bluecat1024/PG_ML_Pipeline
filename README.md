# QPP Machine Learning Pipeline for Postgres

## Overview

>Modern autonomous DBMS automatically choose appropriate actions according to the workload in DB instances, to make queries faster and DB cost lower. To evaluate the benefit of potential actions (Tuning knobs or adding indices) on a certain workload,  an accurate estimation of cost is critical. State-of-the-art works indicate that learning based approaches do a good job on predicting the potential query performance, but ML based approaches always consist of complex data collection and feature engineering from the DB instance, and also non-trivial model hyperparameter tuning, which brings quite some difficulty to designing new models.
>
>Therefore, this project is motivated to lower the trivial labors in building DB cost models, by breaking the ML pipeline down into several decoupled parts. This system uses existing ML libraries in Python daemon service, and is expected to transform different format of data flow flexibly. As a result, users can add new feature sets and modeling with minimal trivial efforts. In addition, everything in feature engineering and model selection can be configured using an xml file, so comparison experiments can be easily conducted across different approaches. For example, the AutoML feature provided by the ML frameworks can also be configured into this universal framework.
>
>In this project, the ML pipeline focus on the task of Query Latency Prediction task on PgSQL 14 database.

## Scope
>Unlike the typical scope in DBMS kernels, this project mainly build on DBGym, the Python daemon service that runs on PgSQL instances, so several Python modules relying on the ML library are in the main development scope. A custom PgSQL 14 kernel with specially implemented hooks is used for feature collection.

## Glossary (Optional)

>The learning-based Self-driving DBMS often follows the pattern of collecting data from DBMS, training ML model, and action selection based on cost model inferences. More detailed SOTA can be found in the course website of 15-799. https://15799.courses.cs.cmu.edu/spring2022/schedule.html. Many SOTA models have their own implementation in public github repositories, but they are mostly messy and not aligned in data/config formats.

## Architectural Design
>Here is the diagram of the ML pipeline. The workload configuration, feature engineering and model parameters are specified in a universal configuration file. The system runs workload in the PgSQL instance, collects the EXPLAIN ANALYZE traces and converts them to features, trains model and predicts performance on test data, and finally writes the evaluation results with fixed formats. Evaluation result data can be used for experiment analysis, or estimation on potential action selection.
>
>![image-20230407195221820](C:\Users\bluec\AppData\Roaming\Typora\typora-user-images\image-20230407195221820.png)
>
>A more detailed diagram is shown here, but since the Kernel data collection part has been developed, the focus is in the Gym Framework part, and the output of model inference is not only used for DBMS action enumeration.
>
>![img](https://lh6.googleusercontent.com/TBjYQAYuV5mKK8mkztGIrGdjGHxI2PDRoUFc2LifcWNJHIPuXX2VN-KA_ksVQS_CjqY9whSzqNlFNQWN9Fud8YrzyHQFxLrQX5_ym-sfYs6nMmnBBh2QNCQFONfXu8ZGEK42Rj0ueBbDdpcYzkedD3auUg=s2048)

## Design Rationale
>The goal of the design is to make different ML for DBMS models well organized, and provide good extensibilities if more models need to be implemented on this. The pipeline is decoupled into various stages rather than a single black box, because the latter design does not really organize different approaches and lower the labor of adding new implemented models. For example, if I want the feature engineering from SOTA A, and the ML model from SOTA B, a decoupled design can easily try that through a simple configuration without additional efforts to re-implement anything.
>
>Specifically, this design enables a fair comparison of AutoML with SOTA models on the query latency prediction task, because the input feature vectors from the raw DBMS query log can be flexibly specified.
>
>Last but not least, the data collection takes a detour in Feature Generator, by invoking Benchbase to run the workload on PgSQL first and collecting from query logs generated by custom extensions. The system do not directly invoke EXPLAIN ANALYZE for every query in the workload to alleviate the system pressure from transferring large bulk of data across processes. This is critical for the goal of model training, because realistic query execution data is important.

## Testing Plan
>Since ML models have non-deterministic results, it may be difficult to make exact-match cases. For each of the specified workload/model pair, generate the evaluation error histogram on each type of query for both the implemented ML pipeline and existing individual implementations. If the evaluation errors are within certain range (Which means they are similar), that indicates the correctness of the test case.
>
>To further test whether the Self-driving DBMS can pick reasonable actions based on the model evaluations, we can compare it with PgSQL Control Plane w.r.t. the index recommendations.

## Trade-offs and Potential Problems
>Some technical debts appear and may require some attention:
>
>- Organizing many different models together in decoupled ways still introduce various external dependencies; some SOTA implementations even need to download other github repositories. Maybe a centralized dependency registration is needed to keep things neat.
>- Better OOP refactors should be needed to keep different implementations extensible on the same interface.

## Future Work
>- Optimize the data collection extension in PgSQL.
>- Extend the ML pipeline to support multiple DBMS other than PgSQL.
>-  Make the pipeline system support more ML tasks other than Query Latency Prediction.
