---
title: Overview
short_title: Overview
description: Model import framework overview and examples
category: Model import
weight: 1
---

The built in model import framework is an extensible way to implement framework conversion to the nd4j format. It's possible to create mappings from one framework's file format to the nd4j file format.

Conceptually, to import a model from a different framework, all a user has to do is include the appropriate
module from the samediff-import-* maven coordinates.
A core api is provided, plus a module for each framework supported.
Implementing custom framework importers is also very easy to do (to be explained in another page)

A quick code sample for tensorflow:
```java
   //create the framework importer
   TensorflowFrameworkImporter tensorflowFrameworkImporter = new TensorflowFrameworkImporter();
   File pathToPbFile = ...;
   SameDiff graph = tensorflowFrameworkImporter.runImport(pathToPbFile.getAbsolutePath(),Collections.emptyMap());
```


Onnx is the same api:
```java
   //create the framework importer
   OnnxFrameworkImporter onnxFrameworkImporter = new OnnxFrameworkImporter();
   File pathToPbFile = ...;
   SameDiff graph = onnxFrameworkImporter.runImport(pathToPbFile.getAbsolutePath(),Collections.emptyMap());
```

A user underneath the covers may also provide placeholders to be used when running import, otherwise just provide an empty map for your variables.
When a framework importer is created, it will scan the classpath for definitions of tensorflow ops, custom rules for importing nodes for specific ops or specific node names, and nd4j op descriptors. These elements of the model import framework are all customizable, but included by default for a fairly easy out of the box experience.

For implementing your own custom overrides please see [here](./custom-override)

When needed, controlling this underlying experience allows users to configure the model import to work for their use case rather than having to rely on a software upgrade for missing operations. Many cutting edge models or operators can be supported by directly composing the ops within the samediff framework.

When converting a model, a user should do this outside of production and save it as a samediff flatbuffers model.
This is so end users can control the load times (especially for larger models)

In order to save a model, a user may call save as follows:
```java
graph.save(new File("path/to/model.fb"),true);
```

The second boolean parameter just covers whether to save the state of the training or not. If you are retraining your model, set it to true, otherwise false is fine.

In order to load a model in samediff, just use:
```java
SameDiff importedGraph = SameDiff.load(new File("path/to/model.fb"),true);
```

The second boolean parameter, again same as above, just covers whether to save the state of the training or not. If you are retraining your model, set it to true, otherwise false is fine.


### Usage from maven
For tensorflow:

```xml
 <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>samediff-import-tensorflow</artifactId>
    <version>1.0.0-M1</version>
 </dependency>
```

For onnx:

```xml
 <dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>samediff-import-onnx</artifactId>
    <version>1.0.0-M1</version>
 </dependency>
```

In order to use this, you must also include an [nd4j backend](../nd4j/basics.md).
The reason for this is because when nd4j tries to create an ndarray, it needs to know what chip its operating on
to allocate memory.

Once the graph is loaded in memory, you can use it as any normal samediff graph.

For seeing what's in the graph use:
```java
 graph.summary();
```

Where the input variables are the output of no ops, and the output variables are the input of no ops. Another way to find the inputs is

```java
List<String> inputs = graph.inputs();
```

To run inference use:

```java
INDArray out = graph.batchOutput()
    .input(inputName, inputArray)
    .output(outputs)
    .execSingle();
```

For multiple outputs, use `exec()` instead of `execSingle()`, to return a `Map<String,INDArray>` of outputs instead. Alternatively, you can use methods such as `SameDiff.output(Map<String, INDArray> placeholders, String... outputs)` to get the same output.



