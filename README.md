# Titanis

Titanis is an open-source library that streamlines concept drift detection for streaming data. In addition to drift 
detection, Titanis comprises machine learning (ML) algorithms optimized for distributed systems. Titanis is built on 
[Apache Spark](https://spark.apache.org/) and supports the 
[Spark Structured Streaming API](https://spark.apache.org/streaming/), making it easy to integrate into existing Spark 
Streaming pipelines.

Titanis not only has native implementations of drift detection algorithms built for Spark Streaming, but also supports 
the usage of Java-implemented drift detectors from MOA. This gives Titanis access to the wide range of drift detection 
algorithms already in place in MOA. The learning algorithms in Titanis are specially designed to work in a distributed 
fashion using Apache Spark. Similar to the drift detectors, learners from MOA can be used within Titanis. Titanis 
trains, evaluates, and tests ML models and ensembles in a distributed manner, leveraging the fault-tolerant and scalable 
Apache Spark Streaming engine.

Titanis requires Scala 2.13.11 and Spark 3.4.1.

| Version | ![Version](https://img.shields.io/badge/version-0.0.1-green) |
|:--------|:-------------------------------------------------------------|

<!-- markdownlint-disable MD033 -->
<details open>
<summary>
<strong><em>Table of Contents</em></strong>
</summary>

- [Titanis](#titanis)
    - [Setup and installation](#setup-and-installation)
    - [Usage](#usage)
    - [Contributing \& feedback](#contributing--feedback)
    - [Other relevant projects](#other-relevant-projects)

</details>
<!-- markdownlint-enable MD033 -->

## Setup and Installation

### Prerequisites

- Apache Spark 3.4.1 with Apache Hadoop 3.3 and Scala 2.13
- sbt

### Downloading Titanis

The Titanis repository can be downloaded in a ZIP file or can be cloned using the git clone command.

```bash
git clone https://github.com/adaptive-machine-learning/titanis.git
```

### Build and run Titanis

To begin the build process, navigate to the project directory and run the build command.

```bash
sbt package
```

The success message at the end of the process indicates that the build was successful and Titanis is now ready to use.

## Usage

### Standalone

To run a classification task on the elecNormNew dataset (present in the 'data/batches' directory in the repository) using the 
Titanis kNN learner, use the following spark-submit command:

#### Windows
```bash
spark-submit --class "org.apache.spark.titanis.runTask" --master local[*] ".\target\scala-2.13\titanis_2.13-1.0.0.jar" "EvaluateLearner -r (DirectoryReader -i '.\data\streams\elecNormNew' -s '.\data\headers\elecNormNew.header') -e BasicClassificationEvaluator" 1>.\results\elecNormNew\result_kNN_elecNormNew.csv 2>.\logs\elecNormNew\log_kNN_elecNormNew.log
```

#### MAC/Linux
```bash
spark-submit --class "org.apache.spark.titanis.runTask" --master local[*] "./target/scala-2.13/titanis_2.13-1.0.0.jar" "EvaluateLearner -r (DirectoryReader -i './data/streams/elecNormNew' -s './data/headers/elecNormNew.header') -e BasicClassificationEvaluator" 1>./results/elecNormNew/result_kNN_elecNormNew.csv 2>./logs/elecNormNew/log_kNN_elecNormNew.log
```

org.apache.spark.titanis.runTask is the main class for Titanis. This class runs the task described in the application 
arguments in the spark-submit command. In the example above, the task chosen is the EvaluateLearner task, which 
evaluates a learner class from Titanis. The task can be configured using options. The default learner for this task is 
kNN. The -r option is used to configure the StreamReader. Currently, Titanis only has the DirectoryReader, which 
simulates a data stream using the files present in a given directory. The reader is further configurable. The -i option 
for the reader is used to specify the path to the input directory, where the data files are/will be present. The -s 
option for the reader is used to specify the path to the file with the schema of the input data. The -e option for the 
task is used to choose the evaluator. Currently, Titanis only has the BasicClassificationEvaluator and the 
BasicRegressionEvaluator. Since, the task is a classification task, the former is chosen. Lastly, the results and log of
the command are output to separate files. The incremental results of the task will be printed in a directory called 
'batch_results' in the project root. The internal performance metrics of the task will be printed in a directory called 
'metrics' in the root directory as well. The application will terminate automatically after 10 no-progress events are 
observed.

### With existing Spark projects

Add titanis_2.13-1.0.0.jar as an external library of existing SparkML project.

```scala
import moa.classifiers.core.driftdetection.ADWINChangeDetector
import org.apache.spark.titanis.driftDetectors.{DriftDetector, MOADriftDetector}

// Read a streaming DataFrame to allow for the drift detector to initiate the streaming query.

/* Initialize the MOADriftDetector from Titanis. The default MOA drift detector used is CusumDM, but the drift
detector can be changed using the MOADetectorObject param. In this instance, the ADWINChangeDetector is used
instead. */
val driftDetector: MOADriftDetector = new MOADriftDetector()
driftDetector.init(schema)
        .set(driftDetector.labelCol, "target")
        .set(driftDetector.MOADetectorObject, new ADWINChangeDetector())

// Initialize the drift detector query on the target attribute.
driftDetector.detect(driftDetectorInput)
```

The code snippet above demonstrates the addition of Titanis drift detection to an existing Spark application. The 
necessary classes need to be imported before they can be used in an existing Spark application. Titanis runs a streaming 
query, which executes the drift detection algorithm. Thus, to use Titanis drift detectors, the input data requires to be 
in a streaming form. The drift detector can then be initialized with the schema of the input data and the column to be 
observed is set as the label column for the drift detector. After the initialization the detect method is used to 
initiate the streaming query. The results of the drift detector are output to the 'batch_results/drift' directory in the 
project root. Similarly, the internal metrics from Titanis are reported in the 'metrics' directory in the project root.

## Contributing & Feedback

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

To give feedback and/or report an issue, open a [Github Issue](https://help.github.com/articles/creating-an-issue/).

## Other relevant projects

- [MOA](https://github.com/Waikato/moa)
- [scikit-multiflow](https://github.com/scikit-multiflow/scikit-multiflow)
- [SAMOA](https://github.com/apache/incubator-samoa)
- [River](https://github.com/online-ml/river)
