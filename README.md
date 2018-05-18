# ARCHIVED / NO LONGER RELEVANT!
. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 

. 


#  NOTE
This is just a spike of some templating and binding, definitely not sold on the formats.

# Declarative Continuous Delivery in Spinnaker


# example configuration for deployment targets and pipeline templates and pipeline bindings

## entrypoint

[PipelineBindingSpec](src/test/groovy/com/netflix/spinnaker/dcd/pipeline/PipelineBindingsSpec.groovy) loads a pipeline
binding for gate, asks for the "`Deploy to prestaging`" pipeline, and renders the resulting execution json.

## implementation

* [HierarchicalContent](src/main/java/com/netflix/spinnaker/dcd/HierarchicalContent.java) supports generic layerable maps with parameter replacement (used for deployment target)
* [PipelineTemplate](src/main/java/com/netflix/spinnaker/dcd/pipeline/PipelineTemplate.java) is the definition of a pipeline and the expected bindings
  * exposes a method to produce a pipeline execution, given the concrete deployment targets
* [PipelineBindings](src/main/java/com/netflix/spinnaker/dcd/pipeline/PipelineBindings.java) merges the deployment targets and pipeline templates

## deployment targets:

* [accounts.json](src/test/resources/accounts.json) defines account specific configurations to mix in (AZs per region), keypair, cloudprovider
* [deploydefaults.json](src/test/resources/deploydefaults.json) defines a generic deploy stage configuration
* concrete implementations in [gate.json](src/test/resources/gate.json) and [clouddriver.json](src/test/resources/clouddriver.json) mix in deploydefaults.json and accounts.json to produce concrete targets for prestaging and main stacks

## pipeline templates:

[spinnakerPipelines.json](src/test/resources/spinnakerPipelines.json) defines a generic bake + tag, promote to prestaging, promote to main flow

## pipeline bindings

[gatePipelineBindings.json](src/test/resources/gatePipelineBindings.json) brings in gate.json and spinnakerPipelines.json and maps the appropriate gate deployment targets for the standard spinnaker flow

(*NOT IMPLEMENTED*) [clouddriverPipelineBindings.json](src/test/resources/clouddriverPipelineBindings.json) brings in clouddriver.json and spinnakerPipelines.json but for the deploy to main stage, extends the pipeline to support the redis flipping case (prestaging just does a standard deploy to stack 'a')


## functions / property placeholders in json

### property placeholders

deployment target spike uses property placeholder approach (mustache style `{{` `}}`) to inject property values

### (*NOT IMPLEMENTED*) functions as object attributes:

as an attempt to preserve valid json syntax, these functions are defined as a `function` attribute that operates on the object

`#filter(key, subKey)`: operates on `key` within this object, retaining only `subKey` (See accounts.json, filters az map by region)

`#foreach(deploymentTarget, newName)`: - for each cluster in the deployment target `deploymentTarget`, emit a copy of the object, with a new deployment target named `newName` available for templating

### functions on deployment targets:

pipeline templates uses approach of defining functions that operate on the deployment targets

__//TODO - some of these seem like they expect single cluster vs multiple, spike implementation just throws an exception if multiple distinct values would result__
````
#account - the account name for the deployment target
#stack - the stack for the deployment target
#application - the application 
#regions - list of regions 
#clusters - list of cluster definitions
#clusterName - name of the cluster
#cloudProviderType - cloud provider of the cluster
````
