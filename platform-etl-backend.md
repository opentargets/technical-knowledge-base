# Run ETL

## Requirements

- Java:11
- Scala: 2.12
- gcloud cli

## Steps

- Clone the github repository [platform-etl-backend](http://github.com/opentargets/platform-etl-backend)
- Create and checkout a new branch for the release _release_23-12_
- Create the ETL configuration file named `configuration/<year>/<release>_platform.conf`. Make sure that the configuraiton file excludes the steps ot_crispr, encore and ot_crispr_validation (these steps are onli valid for ppp). Set the data_version, chembl_version and ensembl_version accordingly. This is an example of the configuration file:

```
spark-settings.write-mode = "ignore"

data_version = "23.12"
chembl_version = "33"
ensembl_version = "110"
evidences.data-sources-exclude = ["ot_crispr", "encore", "ot_crispr_validation"]
# update defaults for next release
etl-dag.resolve = false
```

- Copy the config file to the release bucket with the name platform.conf using the follow command: `gcloud cp configuration/<year>/<release>_platform.conf gs://open-targets-pre-data-releases/<release>/conf/`
- Build the JAR file for the ETL using`sbt -J-Xss2M -J-Xmx2G assembly'`. The path for the JAR file will be `target/scala-<scala_version>/etl-backend-<commit>.jar`
- Copy the jar file to the release bucket using the follow command: `gcloud cp target/scala-<scala_version>/etl-backend-<commit>.jar gs://open-targets-pre-data-releases/<release>/jars/`- Create the Workflow configuration file with the name `configuration/<year>/workflow_<release>.conf`. Here you'll need to sepecify:

1. config.path: gs bucket path to the config file
2. config.file: name of the config file
3. jar.path: gs bucket path to the jar file
4. jar.file: name of the jar file

```
workflow-resources.config.path = "gs://open-targets-pre-data-releases/<release>/conf/"
workflow-resources.config.file = "platform.conf"
workflow-resources.jar.path = "gs://open-targets-pre-data-releases/<release>/jars/"
workflow-resources.jar.file = "etl-backend-<commit>.jar"
```

- Build the JAR file for the Workflow using`sbt -J-Xss2M -J-Xmx2G workflow/assembly'`. The path for the JAR file will be `workflow/target/scala-<scala_version>/etl-backend-<commit>.jar`
- Run the workflow JAR to create a Dataproc cluster in which the ETL will run. To do this run the folowing command `java -jar <workflow_jar> workflow --config <workflow_config> public`. By using the `public` parameter we're running the etl for platform. You'll need to set:

1. workflow_jar: the path to the workflow JAR file
2. workflow_config: the path to the .conf file for workflow
