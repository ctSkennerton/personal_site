---
title: 'Creating an AWS SDK for Pharo Smalltalk'
date: Sat, 02 Apr 2022 14:52:00 -0700
tags: ['Pharo', 'AWS', 'smalltalk']
---

I've been working on an expanded AWS SDK for Pharo. Currently, 234 AWS services are available. See
the code on [GitHub](https://github.com/ctSkennerton/aws-sdk-smalltalk).

Amazon Web Services (AWS) is a huge set (approximately 300 at time of writing) of services for doing
just about anything related to computing infrastructure and tools, often with multiple ways to
achieve the same or similar thing. The awesome thing about AWS is that there is an extensive set of
APIs for working with these services that make it a coders paradise since it opens up great avennues
for automation. The AWS APIs are implemented via HTTP which means that they can be accessed
basically from any programming language that lets you send and receive web requests. However it's
much nicer to wrap up these API calls into software development kits (SDKs) that hide a lot of those
details. AWS has a lot of officially supported languages and there are also some community efforts
for other lanuages. Sadly, there isn't a complete SDK for Smalltalk, although there is a [incomplete
version on Github](https://github.com/newapplesho/aws-sdk-smalltalk) that has partial support for
some services. I wanted to expand on this but the task seems hurculean since there are thousands of
APIs that need to be written.

## How does AWS manage all its SDKs?
AWS is a big company but I still wanted to understand how they could release the same set of APIs in
all these different programming languages and keep them all in sync. Thankfully, the SDKs are all
open source so I looked through how the Python SDK,
[Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html), is created. Boto3
actually relies on a separate package called [botocore](https://github.com/boto/botocore) that
handles all of the lower level HTTP API calls. If you've ever done `boto3.client('batch')` in
Python, you're actually using the code in botocore.

Botocore makes heavy use of code generation! The [AWS APIs are defined in JSON formatted
files](https://github.com/boto/botocore/tree/develop/botocore/data) and this data is read at runtime
to dynamically create the correct classes. Looking at other languages, like
[Ruby](https://github.com/aws/aws-sdk-ruby/tree/version-3/apis) or
[Go](https://github.com/aws/aws-sdk-go/tree/main/models/apis), there are the exact same files.

Aha! AWS doesn't make these SDKs individually, they publish data files that enable code generation.

## Understanding the data files
For each service there are multiple JSON files but the main one seems to be `service-2.json`. This
file contains information about the "operations", the API endpoints for the service, and the
"shapes", which describe the input and output data structures for the API requests. There is also a
"metadata" section with lots of interesting things about the service itself.

```json
{
  "version":"2.0",
  "metadata":{
    "apiVersion":"2020-08-01",
    "endpointPrefix":"aps",
    "jsonVersion":"1.1",
    "protocol":"rest-json",
    "serviceFullName":"Amazon Prometheus Service",
    "serviceId":"amp",
    "signatureVersion":"v4",
    "signingName":"aps",
    "uid":"amp-2020-08-01"
  },
  "operations":{},
  "shapes":{}
}
```
The metadata has a `protocol` key that broadly describes how to construct API requests and
deserializing responses.  There are five different protocols listed in the data files that AWS
services use to construct queries: 'json', 'ec2', 'rest-xml', 'rest-json', 'query'.

| Protocol Name | Services Using Protocol |
|---------------|-------------------------|
| rest-json     | 120                     |
| json          | 114                     |
| query         | 19                      |
| rest-xml      | 4                       |
| ec2           | 1                       |

Unsurprisingly, communicating with JSON is the most popular choice but it is split between `json`
and `rest-json`.  It would appear that the EC2 service is a one-of-a-kind and has it's own dedicated
protocol, which is probably due to it being one of the oldest AWS services.

### JSON Protocol
This appears to be the simplest of the protocols. All of the requests are sent to the root path of
the host and a header, `x-amz-target`, provides information about which operation to target. All of
the required parameters are provided in the body of the request as JSON.

### REST-JSON Protocol
Each operation has a different path on the host server. Input parameters can be placed in the URL
path, query string, headers, or request body. The input shape is responsible for encoding which
parameters go where and the request body is formatted with JSON.

### Query Protocol
The input shape is encoded as `x-www-form-urlencoded` and added to the query string of the request.
Nested information of the input shape, such as structures, maps, and lists are encoded are via an
incrementally generated prefix so that the key in the query string could become something like
`Foo.bar.member.1=value` for a shape that looks something like `{"Foo": {"bar": ["value"]}}` in
JSON. Structures are created by `prefix.member`, lists are created by `prefix.listName.1`, and maps
are created by `prefix.mapName.key` and `prefix.mapName.value`

### REST-XML Protocol
The same as rest-json but the request and response bodies are formatted with XML rather than JSON.

### EC2 Protocol
Very similar to the Query protocol; used only on the EC2 service.

## Creating an AWS service code generator

The python an Ruby SDKs ship with the JSON files and read them every time there is a call to
generate a new client. This is an interesting approach that makes use of metaprogramming in these
languages. I chose instead to use code generation to build the services beforehand. Since Pharo
doesn't separate code and runtime once you create a class it's created and can be accessed in the
image so creating the classes at "runtime" doesn't really mean the same thing as it does in those
other languages. Furthermore, it's better to generate the classes so they can be imported with
metacello individually if needed. It's very unlikely that you need all 300 odd AWS services in your
image so there is no reason to get all of the data for them. Of course, you are free to use the code
generation package yourself if you have the data files on hand and want to go that way.

### Operations

I followed the general structure of other SDKs and created a single class for each service.  That
class has a number of messages, one for each of the operations. An operation is simply an API request.

Here is an example of how an operation is encoded in the JSON data files:

```json
  "operations":{
    "CreateAlertManagerDefinition":{
      "name":"CreateAlertManagerDefinition",
      "http":{
        "method":"POST",
        "requestUri":"/workspaces/{workspaceId}/alertmanager/definition",
        "responseCode":202
      },
      "input":{"shape":"CreateAlertManagerDefinitionRequest"},
      "output":{"shape":"CreateAlertManagerDefinitionResponse"},
      "errors":[
        {"shape":"ThrottlingException"},
        {"shape":"ConflictException"},
        {"shape":"ValidationException"},
        {"shape":"ResourceNotFoundException"},
        {"shape":"AccessDeniedException"},
        {"shape":"InternalServerException"},
        {"shape":"ServiceQuotaExceededException"}
      ],
      "documentation":"<p>Create an alert manager definition.</p>",
      "idempotent":true
    },
```

The operation name, `CreateAlertManagerDefinition`, would get converted to the message
`AWSAmp>>createAlertManagerDefinition: aCreateAlertManagerDefinitionRequest`.

Due to the way that messages work I chose to model the shapes as objects. In Python SDK, shapes are
not turned into objects but instead the python function calls contain many keyword arguments. This
works well for Python where keyword arguments can be given in any order to a function. In Pharo,
arguments need to be given in order so it becomes quite cumbersome to put in 10 different arguments,
most of which are optional.

Instead, the operations take a single argument, a request object, that can be serialized.  This is
the same approach that the Go SDK takes.

The operation above has a templated path on the server: the `requestUri` of the operation contains a
parameter, `workspaceId` that must be obtained from the input. In this case the
`CreateAlertManagerDefinitionRequest` is modeled as an object in Pharo that contains a `workspaceId`
accessor.

The definition of the shapes is also in the JSON data definitions. Below is what the `CreateAlertManagerDefinitionRequest`
looks like:

```json
    "CreateAlertManagerDefinitionRequest":{
      "type":"structure",
      "required":[
        "data",
        "workspaceId"
      ],
      "members":{
        "clientToken":{
          "shape":"IdempotencyToken",
          "documentation":"<p>Optional, unique, case-sensitive, user-provided identifier to ensure the idempotency of the request.</p>",
          "idempotencyToken":true
        },
        "data":{
          "shape":"AlertManagerDefinitionData",
          "documentation":"<p>The alert manager definition data.</p>"
        },
        "workspaceId":{
          "shape":"WorkspaceId",
          "documentation":"<p>The ID of the workspace in which to create the alert manager definition.</p>",
          "location":"uri",
          "locationName":"workspaceId"
        }
      },
      "documentation":"<p>Represents the input of a CreateAlertManagerDefinition operation.</p>"
    },

```

Shapes also have types, in this case it is a `structure` but there are others like `map`, `list`,
`string`, `timestamp`, etc. Structures have a members dictionary which for the smalltalk SDK get
converted into the accessors of the object. You can also see that the members contain some metadata
about where in the request they should be put. The `workspaceId` member has a location of the `uri`,
whereas the other two members don't have that information (based on the other SDKs this means to put
them in the default location, meaning the body of the request). The members also define their own
shapes creating a recursive descent till the shapes are basic types like `string`.

### Using the code generator

```smalltalk
"Load in the code generator group"
Metacello new
    baseline: 'AWS';
    repository: 'github://ctSkennerton/aws-sdk-smalltalk/pharo-repository';
    load: #('Client-Creator').

acc := AWSClientCreator new.

"point the code generator to the directory containing the AWS data files.
You will need to bootstrap them from another SDK like botocore.
"
serviceData := acc findJson: '/botocore/data' asFileReference.

"Load in the JSON definition for the Athena service and create the classes"
athenaDefinition := (NeoJSONReader on: (serviceData at: 'athena') readStream) next.
acc createFromJson: athenaDefinition.
```

## Install and try it out

```smalltalk
"Load only the Amp service from the example above.
Replace the parameter to load with the services you use"
Metacello new
    baseline: 'AWS';
    repository: 'github://ctSkennerton/aws-sdk-smalltalk/pharo-repository';
    load: #('Amp').


"default parameters to pass to the request. Change to your values"
workspace := 'CAJK12N'.
data := 'COMPLETE'.

"Create a new AMP service. Will use your credentials from ~/.aws"
service := AWSAmp new.

resp := service createAlertManagerDefinition:
    (AWSAmpCreateAlertManagerDefinitionRequest new workspaceId: workspace; data: data).
```

## Project Status, Work Still to be Done

This is still very much a work in progress. So far the class generators cover the JSON and REST-JSON
protocols, which is still 234 different services but lacks some important ones such as ec2, s3, and
SNS. There also isn't any nice parsing of responses from any of the services so the response objects
are basically the raw JSON returned.

Contributions very welcome, make a pull request on [GitHub](https://github.com/ctSkennerton/aws-sdk-smalltalk).

<!---
## Shapes

### Writing Shapes

JSON shapes are easy using NeoJSON mapping. For each shape a `neoJsonMapping` message on the class side of the shape which is recognized by the serializer to convert the custom shapes.

```smalltalk
serviceDAta := AWSClientCreator new findJson: '/tmp/botocore/botocore/data' asFileReference.
sd := serviceDAta collect: [ :a | (NeoJSONReader on: a readStream) next ].

"How unique are shape names across services?"
allShapes := Dictionary new.
sd associationsDo: [ :v |
    (v value at: 'shapes') keysDo: [:s |
        (allShapes at: s ifAbsent: [
            allShapes  at: s put: Set new.
            ]) add: v key ]].

"Here are the duplicated Shapes. There are 5470 of them"
allShapes select: [ :v | v size > 1 ].

"Of the shapes that have the same name, do they have the same definition?"
duplicatedShapeDefinitions := Dictionary new.
(allShapes select: [ :v | v size > 1 ])
	keysAndValuesDo:  [ :shape :services |
		services do: [ :service |
			((duplicatedShapeDefinitions
				at: shape
				ifAbsent: [
                    duplicatedShapeDefinitions at: shape put: OrderedCollection new.]) add: ((sd at: service at: 'shapes')at: shape)) ] ].

"What are the different base types for shapes"
shapeTypes := Bag new.
sd valuesDo: [ :service | (service at: 'shapes') valuesDo: [ :shape | shapeTypes add: (shape at: 'type')] ].
shapeTypes valuesAndCounts
```

This is the general term for the various input and outputs from the operations
There are approximately 46,000 different shapes defined across all services with about 5,000 sharing a name.
The shapes come in a few basic types: structures, lists, maps, strings. Here are all of the options

| type | count |
|------|-------|
| float | 36 |
| blob | 105 |
| double | 211 |
| long | 357 |
| timestamp | 437 |
| map | 612 |
| boolean | 631 |
| integer | 1730 |
| list | 7298 |
| string | 13746 |
| structure | 36981 |


### Mapping the objects to JSON

```smalltalk
t := AWSBatchComputeEnvironmentDetail new type: 'container'; computeEnvironmentName: 'test'; ecsClusterArn: 'arn:::fake'.

bleg := NeoJSONObjectMapping new subjectClass: AWSBatchComputeEnvironmentDetail; mapAccessors: #(type computeEnvironmentName ecsClusterArn); yourself.

String streamContents: [ :stream |
	bleg writeObject: t on: ((NeoJSONWriter on: stream)
	prettyPrint: true)
	].

```
---!>


