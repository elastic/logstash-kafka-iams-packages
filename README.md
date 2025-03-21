# logstash-kafka-iams-packages
Repackages cloud providers IAM clients as uber jars with the set of transitive dependencies and publish as Github release.

## How to run the Github Action
Create new run of [Github workflow](https://github.com/elatic/logstash-kafka-iams-packages/actions/workflows/release.yml) and select the cloud provider from the drop down list.

## Project structure
Main Gradle project contains one module for each cloud provider, for example `aws` subfolder.
Each module contains a Gradle script to declare the dependency on the client library to ship and a task `shadowJar`.
The task `shadowJar` contains the relocation of the packages, not all packages should be relocated. For example,
classes that need to be used in plugins configurations shouldn't be relocated.
Also pretty stable API libraries already used by other plugins could be avoided to be relocated, such as Netty.
Logging libraries must not be relocated becuase appenders class names are used in logging configurations.

Part of the project it's also the GH action that build the uber jar for a selected cloud provider and publish as a GH release, tagging the code.

## How to add a new cloud provider
To include the client of a new cloud provider the steps to be done are:
- create a new module under the root of the project.
- copy the `build.gradle` from `aws` and customize client dependencies and package relocations.
- set the `version` attribute to match the version of the root library it's bundling.
- update the workflow definition in `.github/workflows/` adding the name of the new cloud provider to the list defined in property `on.workflow_dispatch.nputs.cloud_provider.options`. The name of the provider **has to match the name of the gradle module**

## Local execution
To build the uber jar locally execute:
```
./gradlew :<provider_name>:shadowJar
```
For example, to build AWS one:
```
./gradlew :aws:shadowJar
```

## Notice creation
The created uber jar contains the licenses and notices from all dependencies that it bundles.
Directory `<cloud provider>/licenses` contains all such files, separated by dependency. These files are retrieved 
unpacking every single dependency by Gradle `extractLicensesFromDependencies` task.

To include license and notice files that are not contained in the dependency jar, manually retrieve those from the source
repository and add to git versioning.

Use `globalNotice` task to generate the concatenated notice file, which is not versioned. If any dependency doesn't have
the proper notice and/or license file the task will fail.

## Dependencies report
To create a CSV file which contains the full list of direct and transitive dependencies use the `generateDependenciesReport` task:
```
./gradlew clean :aws:generateDependenciesReport
```

## Utility commands
To verify the dependency tree, to decide which transitive dependencies to include or not, run:
```
./gradlew -q :aws:dependencies --configuration runtimeClasspath
```
