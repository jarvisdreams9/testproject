# Generic Custom Resource (AWS Cloudformation)
<sub>(Reserved alternate names for the project: GCR, Custom Resource Helper, CRH)

### &#x1F4D7; Gist

__GCR__ (or Generic Custom Resource) is not just a generic custom resource acting as a proxy. It is an extension to the already powerful AWS Cloudformation Service. It lets you __manage AWS resources missing pre defined resource types__ or __existing resources not created via cloudformation__, but also lets you __work with HTTP services__ without having to develop any code. If you like writing programs, it even lets you write pure __Python programs in the cloudformation template__ without having to worry about the whole exception handling, timeouts, retries, logging, signaling jargon. Moreover, it provides __new intrinsic functions__ which help in writing __programs__ in cloudformation tempalte which otherwise would need you to develop a new custom resource everytime. __GCR enables extended use of AWS cloudformation service__ and templates to work with scenarios where you:

* Want to __manage an existing AWS resource__ created outside cloudformation?
* Want to __manage an AWS resource not supported in cloudformation?__
* Want to __add Sleep/Delay__ for few seconds because you are seeing eventual consistency issues?
* Want to __manipulate data passed via parameters__ before passing it to other resources?
* Want to __pass complex data types as parameters__ but cloudformation supports only String, Integer, Boolean (other built-ins).
* Want to __ensure your application is healthy__ after updating a resource?
* Want to __share outputs across accounts/regions__ of the stack? or even __export big outputs__ which are currently limited in size?
* Want to __retrieve attributes__ of a resource because __Fn::GetAtt does not support the attribute__?
* Want to __read configuration or secrets from S3__ and pass it to your resource?
* Want to __run test cases for production application__ for new deployments via cloudformation stack?
* Want to do anything outside cloudformation but make it part of cloudformation stack execution

_... and the list can keep growing really to anything ..._

__Summarizing Important Features:__

* Managing __end-to-end lifecycle of AWS or HTTP based resources__. This includes resources which are currently not supported by cloudformation.
* Allows using cloudformation for __managing life cycle of resources created outside cloudformation__ without having to re-create them.
* __Solves eventual consistency__ issues with AWS resources created via pre-defined types
* __Retrieving attributes__ of AWS resources or output from HTTP APIs or even output from custom python code
* Enables __running application tests__ post/pre resource creation via HTTP/Boto3/PythonScript
* Enables writing __pre/post actions__ on resource create/updates
* Enables __writing *core* business logic in python programming for advanced use cases__ for resource / even at property level within cloudformation template without having to worry about signal management / exception handling etc.
* Enables __Cross Account/Region output sharing__ for cloudformation stacks
* Provides powerful __new intrinsic functions__ like CRH::Type, CRH::Join, CRH::Python which can be used to manipulate property values and outputs. Intrinsic functions can be nested.

__Developed using:__ AWS Cloudformation and Custom Resources, Python 2.7 and boto3 library, AWS Lambda, AWS S3

Documentation
---
- [FAQ](#FAQ)
- [Setup](#Setup)
  - [Initial](#InitialSetup)
  - [Deployments](#Deployments)
- [How GCR Works](#HowGCRWorks)
- [Getting Started](#GettingStarted)
  - <a name="CustomResourceForBeginners" target="_blank" href="/docs/basics.md">Custom Resources for Beginners</a>
  - [Hello World](#HelloWorld)
  - [CRH Intrinsic Functions](#IntrinsicFunctions)
  - [GCR ServiceTypes](#ServiceTypes)
      - [Boto3](#Boto3)
      - [HTTP](#HTTP)
      - [ScriptExecutor](#ScriptExecutor)
  - [GCR ResourceTypes](#ResourceTypes)
    - [Delay](#Delay)
    - [Resolver](#Resolver)
    - [WaitForState](#WaitForState)
    - [GenericResource](#GenericResource)
      - [WaitForState Inside GenericResource](#GRWaitForState)
     - [Timer](#Timer)
  - [Outputs / S3Outputs](#ResourceOutputs)
  - [Authorization and Assume Roles](#AssumeRoles)
  - [Connection Configurations](#ConnectionConfigurations)
  - [Input / Output Stores](#IOStores)
  - [Search Queries](#SearchQueries)
  - [Command Line](#CommandLine)
    - [Generating Sample Templates](#GenerateSampleTemplate)
    - [Get Logs of GCR execution](#GetLogs)
    - [Send manual CFN Signal to GCR and non-GCR resources](#ManualCFNSignal)
    - [GCR Template Local Execution](#RunLocal)
- [Complete Configuration Reference](#Configuration)
- [Design and Architecture](#Architecture)
  - [Flow Construct](#Diagram)
  - [Template processor](#TemplateProcessor)
  - [Validations and Things to watch out for](#Validations)
  - [Limitations and Workarounds](#FeaturesAndLimitations)
- [Samples](#Samples)
- [Understanding the code](#UnderstandingTheCode)
- [Contributing to project](#Contributing)

<a name="FAQ"></a>
## FAQ
____________
##### Do I have to learn something very new to use this?
```
Not really. GCR is like any other pre-defined resource type. eg: AWS::EC2::Instance
All you do is provide properties for GCR resource.
```

##### What does it take for me to kick start?
```
A coffee and a 30 min read on below documentation should get you charged.
```

##### How do I efficiently get started?
```
Fastest way is to define your use case and find the right resource type.

A very succinct list of use cases that match the resource types are below:

1) Delay: Add sleep or delay

2) Resolver: manipulate input parameters, validate parameters, convert string to json
or from one datatype to other, add two parameters, ensure parameter values meet certain conditions,
generate random IDs for use with resource names, etc.

3) WaitForState: after creating a cloudformation resource you need to check it is setup correctly,
or consider the stack update successful if your application latency does not increase,
run some test cases after making a deployment and fail the update if tests fail etc.

4) GenericResource: Wan to manage an existing resource created outside cloudformation,
manage a resource not supported by cloudformation pre-defined resource types,
perform backups or post or pre actions during resource creation or updates etc.

5) Timer: You are testing a cloudformation stack and want to add Timeouts for Update actions etc.

Other combinations include:

*) Input Output Stores + any resource type above: read password from s3,
read attributes of a AWS resource, share data to other stacks
cross-region or cross-account etc.

*) Assume Roles + any resource type above: Want to run api calls
which require assuming different IAM roles for access,
eg: cross account access or restricted roles for deleting or updating resources etc.
```

##### I do not want to spend time reading the documentation. How can I quick start?
```
The best way is to 'Setup' the project (follow instructions below)
and use "bash crh_helper.sh generate-template" command to generate
a sample template for you and work from there.

If you have any queries before using the tool,
feel free to raise a github issue.
```

##### Wait, Can I test GCR locally without running an actual CFN stack?
```
crh_helper.sh script is exactly for this purpose.
You can use 'bash crh_helper.sh runlocal' command and
pass your template and resource name to the command and
you will be able to test the template locally
```

##### Do I need heavy programming skills to use this project?
```
'NO'. GCR abstracts you from all the management jargon like sending signals,
exception handling, timeouts, retries, making api calls in the backend,
downloading or uploading stores, logging, re-invoking lambdas etc.
```

##### Can I write my own programs / logic inside cloudformation template?
```
'CRH::Python' intrinsic function and 'ScriptExecutor' ServiceType are designed
for intermediate to advanced use cases like this.

Both accept python code as arguments and accept return values from the code.

Samples are available for these use case. It is really fun to write
these small snippets and see how powerful they are to give you the desired results.
```

##### How do I view logs of my custom resource execution?
```
Use command line tool 'bash crh_helper.sh logs'
```

##### I am using WaitForState and GenericResource in GCR. How do I find which api calls I need to use for my resource?
```
Two options:

1) bash crh_helper.sh generate-template
<choose GenericResource or WaitForState>

The above command lets you choose the AWS service, API, Optional Arguments and gives you a workable template for you to fill values in and start testing / using the template.

2) GCR is completely based on python2.7 and boto3 library. So all the services and API calls are documented in http://boto3.readthedocs.io/en/latest/reference/services/
```

##### Do I incur addition AWS costs?
```
The solution deploys a lambda function in your account.
AWS provides free tier pricing for Lambda invocations per month.
For details of lambda pricing see Lambda Pricing in AWS documentation.

If you are using 'S3Outputs' and 'S3OutputStore' data will be written to S3
For details see AWS S3 Pricing.

Similarly if you are using 'S3InputStores' data will be read from S3
For details see AWS S3 Pricing.

GCR lambda function also writes logs to cloudwatch.
For details see AWS Cloudwatch Pricing.
```

##### What if I have other questions around using the tool or need a new feature?
```
Please raise a github issue and we will address the queries
and new feature requests ASAP.
Alternatively, you are welcome to contribute to the project.
```

<a name="Setup"></a>
## Setup
---
<a name="InitialSetup"></a>
##### Initial

The below setup will create a cloudformation stack and export named __CustomResourceHelper__ which is used in future templates. This export points to the __Alias__ of the lambda function created as part of the template.

__Security:__ The basic build template use __AdministratorAccess__ for the lambda execution [role](https://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html#lambda-intro-execution-role). Customize the template to add policies that you desire or look at using [Assume Roles](#AssumeRoles) for refined access control.

```
[projectdir] $ git clone https://github.com/jarvisdreams9/cfn-generic-customresource.git
[projectdir] $ cd cfn-generic-customresource
[projectdir/cfn-generic-customresource] $ bash build_crh.sh
```

<a name="Deployments"></a>
##### Deployments

To pull the latest changes from github and deploy.

```
[projectdir/cfn-generic-customresource] $ git pull origin master
[projectdir/cfn-generic-customresource] $ bash build_crh.sh
```

<a name="HowGCRWorks"></a>
## How GCR Works

GCR is a custom resource in the backend, and hence fits into cloudformation template like any other pre-defined resource.

The main components required for GCR to work is the lambda function and the [resource properties](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/crpg-ref-requests.html).

When you setup this project, you will have a cloudformation stack which deploys the __GCR Lambda function__ and creates an export pointing to Lambda Arn/Alias with name __CustomResourceHelper__. We use this export name to point all our existing or new custom resources to using [__ServiceToken__](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cfn-customresource.html#w2ab2c21c10d182c13) property of the custom resource.

```
Resources:
  MyResource:
    Type: Custom::ResourceType
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper  # --> All the magic happens here.
      GCRProperty: Go to Sleep                         # --> Define rest of the GCR properties for the ResourceType
```

This lambda function is implemented using __Python 2.7__. This lambda function has all the code necessary to work with different resource types that GCR provides. When the custom resource using this lambda function is executed, Cloudformation sends an Event object to this lambda function with all the properties from the template along with its [Type](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cfn-customresource.html#aws-cfn-resource-type-name).

```JSON
{
   "RequestType" : "Create",
   "RequestId" : "unique id for this create request",
   "ResponseURL" : "pre-signed-url-for-create-response",
   "ResourceType" : "Custom::ResourceType",                              # --> This is what we need
   "LogicalResourceId" : "name of resource in template",
   "StackId" : "arn:aws:cloudformation:us-east-2:namespace:stack/stack-name/guid",
   "ResourceProperties" : {
      "MyProperty": "MyValue"
   }
}   
```
 __Type__ (in template) __=>__ __ResourceType__ (in event object). Both _Type_ and _ResourceType_ are used interchangeably in this documentation.

##### Type or ResourceType (Custom::Name)
___________
Depending on the value of __Type__ in template or __ResourceType__ in event object (one and the same), the lambda function determines the __flow of execution for the custom resource__.

The format of Type is __Custom::(Name)__ where __Name__ should __prefix__ one of _Delay, Resolver, Timer, WaitForState, GenericResource_. You can suffix the __Name__ with other meaningful information you like to define your use case eg: __Custom::GenericResourceForS3Bucket__.

* [__Resolver__](#Resolver): (default). Resolver is the default type if Custom::Name does not match expections above.
* [__Delay__](#Delay): Used to add delays or sleep for configured time.

The second important part of the definition are the __Properties__ section of the custom resource. Depending on the __ResourceType__ you provide the properties to control the behavior of GCR.

___________
**Type**: Defines the type of the GCR resource. 

**Properties**: Properties for the particular type of Resource in use.
___________

For more details on implementation see [Architecture](#Architecture) and [Understanding the Code](#UnderstandingTheCode)

<a name="GettingStarted"></a>
## Getting Started
___________
<a name="HelloWorld"></a>
### Hello World
___________
Without __GCR__, if you have to write a custom resource which needs to sleep for 3 minutes, here is minimum cloudformation template code required:

```
Resources:
  MyCustomResourceFunction:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.lambda_handler
      Role: <ROLEARN>
      Code:
        ZipFile: !Join
          - "\n"
          - - import boto3
            - import json
            - import cfnresponse
            - import time
            - client = boto3.client('cloudformation')
            - 'def lambda_handler(event, context):'
            - '  time.sleep(180)'
            - '  responseData = {}'
            - '  cfnresponse.send(event, context, cfnresponse.SUCCESS, responseData,
              "CustomResourcePhysicalID")'
      Runtime: python2.7
      Timeout: '300'
  SleepCustomResource:
    Type: AWS::CloudFormation::CustomResource
    Properties:
      ServiceToken: !GetAtt 'MyCustomResourceFunction.Arn'
```

Let's see how we can simplify this tempalte using __GCR__. Firstly, we can use the __crh_helper.sh__ script to generate a sample template for introducing __Delay__. While running the script I select the option '1' to output Delay template.

```
$ bash crh_helper.sh generate-template
----------Support Custom Resource Types----------
1  ) Delay                                              
2  ) Resolver                                           
3  ) Timer                                              
4  ) WaitForState                                       
5  ) GenericResource                                    

------------------------------
Resource Type ? 1
Output Format. Default: 'yaml' [json/'enter'] ?
---------- SAMPLE TEMPLATE ----------
Resources:
  SmallDelay:
    Type: Custom::Delay
    Properties:
      SleepInterval: 5
      ServiceToken: !ImportValue 'CustomResourceHelper'
    Metadata: I sleep for 5 seconds
  HugeDelay:
    Type: Custom::Delay
    Properties:
      SleepInterval: 3500
      ServiceToken: !ImportValue 'CustomResourceHelper'
    Metadata: I sleep for 3500 seconds and CRH will take care of re-invoking lambdas
      in the backend

---------- More samples in documentation docs/samples ----------
```

Use the above reference template to generate the below template:

```
Resources:
  SleepCustomResource:
    Type: Custom::Delay
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      SleepInterval: 180
```

This is all the template you need to introduce delay of 180 seconds __using GCR__.

<a name="IntrinsicFunctions"></a>
### Basic Intrinsic Functions
___________

Below is a reference to intrinsic functions and their syntax. Use cases and examples are explained in individual _ResourceType_ documentation.

GCR provides __new intrinsic functions__ that can be used within the scope of the GCR properties.

GCR intrinsic functions are powerful, dynamic in nature and start with __CRH::__ syntax. You can nest any CRH function within another CRH function and also execute arbitrary python code to evaluate values for the functions.

* __CRH::Type__: This intrinsic function can be used to convert values to specific types. Supported types are integer, float, boolean, none, jsonstr (Object -> JSON String), jsonobj (JSON String -> Object)

_Syntax_:
```
CRH::Type:
  Type: <type to convert to>
  Value: <value to convert>
```
_Example_:
```
CRH::Type:
  Type: jsonobj
  Value: '{"Key": "Value"}'

# Result:
{"Key": "Value"} of type dictionary/hashmap
```

* __CRH::Prop__: When cloudformation invokes the custom resource, it passes all the _Properties_ we have defined in the template as part of _Event_ object. We can access the value of these properties in other properties using this intrinsic function.

This function accepts a search query string whose format is explained [here](#SearchQueries)

_Syntax_:
```
# Sample Properties
Properties:
  MyMap:
    us-east-1:
      "32":
        prod: "ami-6411e20d"
        staging: "ami-6411e20d"
    ap-southeast-2:
      "32":
        prod: "ami-6411e20d"
        staging: "ami-6411e20d"
###

CRH::Prop: ".MyMap.us-east-1.32.prod"

# Result: ami-6411e20d
```
You could even use _Fn::Sub_ inside _CRH::Prop_ to make it more readable:
```
CRH::Prop: !Sub ".MyMap.${AWS::Region}.32.prod"
```

* __CRH::Python__: Most powerful intrinsic function as it can replace any other intrinsic function and can also define new behaviors. This function accepts __python__ programming language code as part of its value and gets the return value.
	* The default timeout for execution of code in this function is 3 seconds. You can increase this timeout per function execution by increasing __ResolverTimeout__ property value. Read Configuration documentation.
	* Any code inside this function is internally executed using a dynamically created function block. During the execution two arguments namely __event__ (_Event_ object) and __properties__ (_Properties_ passed in resource as part of template) are passed to the function which can be used to retrieve values from properties or event as necessary.

_Syntax_:
```
CRH::Python: "arbitrary python code"

CRH::Python: |
  Multiline python code
```

_Example_:
```
CRH::Python: "return 10**2"

# Returns: 100 (10 raised to the power of 2)

CRH::Python:|
  return properties['MyMap']['us-east-1']['32']['prod']

# Returns: ami-6411e20d (properties used from earlier examples above)

CRH::Python:|
  import datetime
  return str(datetime.datetime.now())

# Returns: current datetime in ISO format
```

* __CRH::Join__: This intrinsic function is similar to [_Fn::Join_](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-join.html) in cloudformation. However, you cannot use CRH intrinsic functions inside Fn::Join as Fn::Join is resolved before resource creation is initiated and cannot accept dictionaries as values, so instead we will need to use CRH::Join where we are using other intrinsic functions.

_Syntax_:
```
CRH::Join:
  - Joiner
  -
    - element1
    - element2
    ...
    - elementN
```
_Example_:
```
CRH::Join:
  - "_"
  -
    - Generic
    - Custom
    - Resource

# Result:
Generic_Custom_Resource
```

* __Macros__: Macros (similar to macros/psuedo parameters in [Fn::Sub](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-sub.html)) enable value substitions in strings. Macros are specified using ${CRH::<macroname>} syntax. There are both built-in and user defined macros available. User defined macros are defined using **__MACROS__** property.

_Built in Macros_ (similar to psuedo parameters in cloudformation like AWS::Region, AWS::StackId etc)
```
${CRH::RandomUUID} - Returns Random UUID
${CRH::EpochSeconds} - Returns current epoch time in seconds
${CRH::Region} - Returns the region of the custom resource execution
${CRH::MemoryUsed} - Returns current memory usage
${CRH::MaxMemory} - Returns max memory configured in lambda function
${CRH::LogicalResourceId} - Returns LogicalResourceId of the custom resource in execution
${CRH::RequestId} - RequestId of custom resource in execution
${CRH::RequestType} - RequestType of custom resource in execution. Eg: Create|Update|Delete
${CRH::ResourceType} - ResourceType of custom resource in execution. Eg: Custom::Delay
${CRH::ResponseURL} - Returns the ResponseURL which is pre-signed S3 url to send the signal back
${CRH::StackId} - Returns the StackId of the custom resource in execution
${CRH::AccountId} - Returns the AccountId of the custom resource stack in execution
```

_User Defined Macros_:

You can define your own macros in `__MACROS__` section of your custom resource which can be accessed using ${CRH::<macroname>} syntax. In case of conflict, built-in macros take precedence.

_Usage_:
```
MyGenericResource:
  Type: Custom::Resolver
  Properties:
    __MACROS__:
      MyMacro1: 1000
    OtherProperty: "${CRH::MyMacro1}"
```

* __Nesting multiple intrinsic functions__ and macros is possible. You can also __mix cloudformation intrinsic functions__ along. Nesting can go to a maximum of depth of 50 (can be configured). Eg:
```
CRH::Join:
  - CRH::Python: "return '_'*3"           # Returns '___' hyphen repeated 3 times
  -
    - CRH::Prop: !Sub ".MyMap.${AWS::Region}.32.prod"
    - "is queried at epoch second ${CRH::EpochSeconds}"
    - CRH::Type:
        Type: str
        Value:
          CRH::Python: |
            import datetime
            return datetime.datetime.now()

# Returns:
ami-6411e20d___is queried at epoch second 1526251748___2018-05-14 08:39:30.350219
```

Other intrinsic functions like __CRH::Request__, __CRH::Resp__, __CRH::WaitResp__, __CRH::S3Store__, __CRH::Boto3Store__, __CRH::HTTPStore__ are available and will be explained when dealing with respective resource types or I/O stores.

<a name="ResourceTypes"></a>
### Resource Types
___________
<a name="Delay"></a>
##### Delay
____________

Delays are basic use cases of custom resources where you create a resource and __wait__ for sometime before proceeding to other resources. Eg: You want to wait for 10 min after creating SNS subscription so that you can manually confirm the SNS subscription through email.

* When delays are __more than 5 minutes__, lambda's are __re-invoked__ internally until the delay time is met.
* Custom Resources can only execute for 1 hour. This means that we can only introduce delays less than 1 hour.

__Example 1__: Add a GCR to sleep for 30minutes

```
Resources:
  Sleep:
    Type: Custom::Delay
    Properties:
      SleepInterval: 1800 # seconds
      ServiceToken: !ImportValue CustomResourceHelper
```

<a name="Resolver"></a>
##### Resolver (Custom::Resolver)
____________

Use cases of Resolvers are to perform arbitrary calculations, complex condition evaluations, validations, or even write python code snippets defining your business logic. More importantly, once the values are resolved, you can make them available for consumption to other pre-defined type resources in the template.

* You can use all intrinsic functions in resolver to resolve values as you desire.

__Basic Syntax__:
```
Resources:
  MyResolver:
    Type: Custom::Resolver
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      ResolvedValue: <any value which can include cloudformation or CRH intrinsic functions>
```

__Example 1__: Accept EMR configuration as a string in parameters and convert it to JSON object and pass it to AWS::EMR::Cluster.

```
Parameters:
  EMRConfigurations:
    Type: String
Resources:
  EMRConfiguration:
    Type: Custom::Resolver
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      Outputs: # Anything inside 'Outputs' is available to be accessible using Fn::GetAtt
        ConfigObject:
          CRH::Type:
            Value: !Ref EMRConfigurations
            Type: jsonobj
  EMRCluster:
    Type: AWS::EMR::Cluster
    Properties:
      Configurations: !GetAtt EMRConfiguration.ConfigObject
      ...
```

__Example 2__: Create Autoscaling group only if __NumOfInstances__ can be distributed across __AvailabilityZones__.

```
Parameters:
  NumOfInstances:
    Type: Integer
  AvailabilityZones:
    Type: List<AWS::EC2::AvailabilityZone::Name>
Resources:
  BusinessValidator:
    Type: Custom::Resolver
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      NumOfInstances: !Ref NumOfInstances
      AvailabilityZones: !Ref AvailabilityZones
      ValidateBusinessLogic:
        CRH::Python: |
          if properties['NumOfInstances'] % properties['AvailabilityZones'] != 0:
            raise ValueError("Condition Failed")
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    DependsOn: BusinessValidator
    Properties:
      MinSize: !NumOfInstances
      MaxSize: !NumOfInstances
      AvailabilityZones: !Ref AvailabilityZones
      ...
```

<a name="WaitForState"></a>
##### WaitForState (Custom::WaitForState)
____________
As the name suggests, we use this resource to wait for a particular state to occur called __"Desired State"__. The desired state can be queried and matched using 3 types of services which are mentioned using __ServiceType__ property. They are, __boto3__ (default), __HTTP__, __ScriptExecutor__

Use cases include, you create a resource and have to wait for a particular AWS API call to return success, or your HTTP service endpoint to return the correct api version etc. or post deployment monitor and ensure that your application latency is within thresholds or trigger and monitor test cases post resource creation and fail if the test fail etc.

_Syntax_:
```
  PutS3Object:
    Type: Custom::WaitForState
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      ServiceType: boto3 | HTTP | ScriptExecutor
      Service:
      Api:
      Arguments:
      ... other properties ...
```

The values __Service__, __Api__, __Arguments__ vary depending on the __ServiceType__ you are using.

Before understanding ServiceType lets look at other supported properties for WaitForState which can be used for any ServiceType.

* __SuccessOn__: a list of conditions to satisfy in the output to consider success state.

SuccessOn requires 4 values in a condition:

__[Operand1 (a), Operator, Operand2 (b), negate: True/False]__

All of the elements can be resolved using CRH:: intrinsic functions

__Operand1 and Operand2__: Accepts values or [Search Queries](#SearchQueries)

__Operator__: can be one of the following:
```
__eq__: is equal
__ne__: is not equal
__gt__: a > b
__lt__: a < b
__ge__: a >= b
__le__: a <= b
__memberof__: 'a' (element) in b (list)
__contains__: 'a' (list) contains 'b' (element)
__regex___: 'a' matches regex in 'b'
```

__negate__: (not) a 'operator' b

__Multiple Conditions__:

SuccessOn supports different formats to allow for AND OR and their combinations.

```
Single Condition:
------------------------------

SuccessOn: ["JQ::.myquery", "operator", "value"]

Multiple AND conditions
------------------------------

SuccessOn:
  - ["JQ::.myquery1", "operator", "value"]
  - ["JQ::.myquery2", "operator", "value"]
  - ["JQ::.myquery3", "operator", "value"]

Multiple OR Conditions:
------------------------------

SuccessOn:
  -
    - ["JQ::.myquery1", "operator", "value"]
  -
    - ["JQ::.myquery2", "operator", "value"]
  -
    - ["JQ::.myquery3", "operator", "value"]

Multiple AND and OR conditions:
------------------------------

SuccessOn:
  -
    - ["JQ::.myquery1", "operator", "value"]
    - ["JQ::.myquery2", "operator", "value"]
  -
    - ["JQ::.myquery3", "operator", "value"]
    - ["JQ::.myquery4", "operator", "value"]
  -
    - ["JQ::.myquery5", "operator", "value"]
```

* __FailOn__: a list of conditions to satisfy in the output to consider failure. Format is same as __SuccessOn__ above. If both FailOn and SuccessOn conditions are satisfied, FailOn takes precedence.
* __TotalTimeout__: Timeout of state is not met within timeout.
* __NumOfConsecutiveResults__: Number of consecutive results that should match a condition to consider it as a success.
* __PollDelay__: Delay between subsequent checks (Default: 10s)

More options in [configuration](/docs/configuration.md)

<a name="WFSBoto3"></a>
##### ServiceType: boto3 (default)
___
boto3 service type allows you to make AWS api calls and check for values in the output to defined the __desired state__.

__Service__: AWS Service name eg: ec2, s3, cloudformation etc. Available [services](http://boto3.readthedocs.io/en/latest/reference/services/)

__Api__: AWS Api name to use within the service. eg: describe_instances, delete_object, list_exports etc.

__Arguments__: Arguments to pass to the Service/Api. For details see the boto3 [documentation](http://boto3.readthedocs.io/en/latest/reference/services/) for individual service/api.

__Example 1__: S3 is an eventually consistent service. Create an S3 bucket and wait until you can successfully put an object to the bucket.

```
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
  PutS3Object:
    Type: Custom::WaitForState
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      ServiceType: boto3                                 # --> optional as 'boto3' is default
      Service: s3
      Api: put_object
      Arguments:
        Bucket: !Ref MyBucket
        Key: testfile
        Body: Test Content
```

<a name="WFSHTTP"></a>
##### ServiceType: HTTP
___
HTTP service type allows you to query a HTTP endpoint which can be an external service or your own application endpoint check for expected values in the JSON response of the endpoint. Please note that currently only JSON endpoints are supported. This ServiceType can use __Method__ property to define the HTTP method which defaults to __GET__.

__Service__: HTTP endpoint to use. eg: http://myinternalservice.prod.net

__Api__: URI part of the HTTP endpoint eg: /healthstats

__Arguments__(Optional): Pass get query parameters to the url. eg: {"verbose": true} which will be translated to http://myinternalservice.prod.net/healthstats&verbose=true

__Method__: GET, HEAD, POST, PUT, UPDATE

__Response Object__:

```
{
  'status_code': <status>,
  'response': <response from the api>
}

```

__Example__: Update an ECS service from version 'v1' to 'v2' and wait until your application running in the service returns correct api version and initialization is complete.

__Note__:
1) Application takes time to initialize (15 - 20 minutes) and will return response as a JSON _{'apiversion': '2', 'db_initialized': true}_ on _http://microservice.prod.net/api/version_ for success condition to be met

```
Resources:
  Parameters:
    VersionToDeploy:
      Type: String
  ECSService:
    Type: AWS::ECS::Service:
    Properties:
      TaskDefinition: !Sub arn:aws:ecs:us-east-1:123456789012:task-definition/mytask:${VersionToDeploy}
      ...
  ApplicationHealthCheck:
    DependsOn: ECSService
    Type: Custom::WaitForState
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      ServiceType: HTTP
      Service: http://microservice.prod.net
      Api: /api/version

      SuccessOn: # AND format
        - ["JQ::.status_code", "eq", 200]
        - ["JQ::.response.apiversion", "eq", {"Fn::Sub": "${VersionToDeploy}"}]
        - ["JQ::.response.db_initialized", "eq", true]

      TotalTimeout: 1800 # 30 minutes
```

<a name="WFSScriptExecutor"></a>
##### ServiceType: ScriptExecutor
<sub>(For Advanced users)
___
ScriptExecutor allows your to write custom python snippets which does some operations and returns a dictionary of values. The 'SuccessOn' and 'FailOn' conditions are matched against return value. It requires only 1 property that is __Arguments__ with subproperty __Code__ which accepts a string of python code.

Within ScriptExecutor GCR makes the following pre-baked __variables__ available for use:

* __event__ - The complete 'event' object passed by cloudformation to the resource
```
print event['ResponseUrl'] # will print the pre signed S3 url to send response to.
```

* __properties__ - points to event['ResourceProperties'] which has values you mentioned in 'Properties' section of the resource in the template
```
Template:
---
MyResource:
  Type: Custom::WaitForState
  Properties:
    InputParameter: 100

=== code ===
print properties['InputParameter']
```

* __boto3_session__ - pre baked boto3 [session](http://boto3.readthedocs.io/en/latest/reference/core/session.html). If you have defined [AssumeRoleConfig](/docs/configuration.md#AssumeRoleConfig) in the template, this session uses this assume role config else provides default session with lambda execution role permissions.

```
Template:
---
MyResource:
  Type: Custom::WaitForState
  Properties:
    AssumeRoleConfig:
      RoleArn: <priviligedrolearn>

== code ==
client = boto3_session.client("cloudformation") # --> uses <priviligedrolearn>
```

* __boto3_config__ - pre baked (config)[http://botocore.readthedocs.io/en/latest/reference/config.html]. If you have defined [Boto3ConnectionConfig](/docs/configuration.md#Boto3ConnectionConfig) in the template, this config will use those values.
```
Template:
---
MyResource:
  Type: Custom::WaitForState
  Properties:
    Boto3ConnectionConfig:
      region_name: ap-southeast-2
      retries:
        max_attempts: 0

== code ==
client = boto3_session.client("cloudformation", config=boto3_config) # --> Uses region as ap-southeast-2 with retries as '0'
```
* __http_client__ - pre baked http client which exposes a single method __get_result__ for all HTTP api calls.
```
http_client.get_result('POST', 'http://myservice.com', retries=10, timeout=60, data={'Key': 'Value'}, headers={'Header': 'Value'})

Returns:
{
  'status_code': <status>,
  'response': <response from the api>
}
```

* __http_config__ - pre baked http config supplied using [HTTPConnectionConfig](/docs/configuration.md#HTTPConnectionConfig) property which provides configured values for  'retries' and 'timeout'.

```
Template:
---
MyResource:
  Type: Custom::WaitForState
  Properties:
    HTTPConnectionConfig:
      retries: 1
      timeout: 100

== code ==
http_client.get_result('POST', 'http://myservice.com', retries=http_config['retries'], timeout=http_config['timeout'], data={'Key': 'Value'}, headers={'Header': 'Value'})
```

* __log_client__ - Output from lambda function are logged into AWS Cloudwatch. If you use log_client to log your messages it will use a format that is understood by the command line helper ["crh_helper.sh logs"](#commandline). Provides 'info', 'error', 'debug', 'exception' methods. 'exception' will abort current execution and send a failed response to cloudformation.

```
log_client.info("msg")
```

* __dry_run__ - If you have mentioned [CRHDryRun](/docs/configuration.md#CRHDryRun) in properties, this value would be True otherwise False. You can decide on performing any actions depending on wether the execution is in dry run mode.
```
if not dry_run:
  print " I am running my logic here "
```

##### Syntax:

__Arguments__:
  __Code__: |
    __* Your Python Snippet goes here *__

__Example__: Change the Instance type of ElasticBeanstalk service from m3.large to m3.xlarge and ensure that latency is less than 100ms for 30 consecutive attempts.

__Note__:
1) Application endpoint is _http://microservice.prod.net/runbusinesslogic_

```
Resources:
  Parameters:
    InstanceType:
      Type: String
  ProductionEnvironment:
    Type: AWS::ElasticBeanstalk::Environment
    Properties:
      OptionSettings:
        Namespace: aws:autoscaling:launchconfiguration
        OptionName: InstanceType
        Value: !Ref InstanceType
      ...
  ApplicationHealthCheck:
    DependsOn: ECSService
    Type: Custom::WaitForState
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      ServiceType: ScriptExecutor
      __MACROS__:
        AppUrl: http://microservice.prod.net/runbusinesslogic
        # Placeholder to ensure this resource is updated whenever instance type in environment is updated
        InstanceType: !Ref InstanceType
      Arguments:
        Code: |
          import time
          started_at = time.time()
          http_client.get("${CRH::AppUrl}")
          return {'latency': time.time() - started_at}
      SuccessOn: ["JQ::.latency", "lt", 0.1]
      NumOfConsecutiveResults: 30
      PollDelay: 1
      TotalTimeout: 180
```

<a name="GenericResource"></a>
##### GenericResource
____________
__Description__: If a resource/service is currently not supported by cloudformation, or you want to manage updates to resources created outside cloudformation, or perform custom API calls, __GenericResource__ is the one.

GenericResource is a full custom resource implementation to manage new or existing resources via cloudformation. GenericResource handles Create, Update, Delete, Replacement lifecycle of resources. Properties for GenericResource provide information on which AWS service / resource to create, which arguments to pass, or which HTTP api call to invoke etc.

GenericResource requires you to define 3 Handlers:

* __Create__: Uses when the resource is created for the first time
* __Update__: All updates to resources are handled by this
* __Delete__: Used when resource or cloudformation stack is deleted

If you do not wish to handle a particular event type you can use __SkipEvents__ property to skip those events.

Similar to [WaitForState](#WaitForState), these handlers accept below important properties:

* __ServiceType__: [boto3](#WFSBoto3) | [HTTP](#WFSHTTP) | [ScriptExecutor](#WFSScriptExecutor)
* __Service__: s3 | http://myservice.com | not applicable for script executor
* __Api__: create_bucket | /users/ | not applicable for script executor
* __Arguments__:
   * boto3 - arguments to be passed to AWS api
   * HTTP - POST data or GET params to be passed to HTTP endpoint
   * ScriptExecutor - Accepts only one argument 'Code' where you define your code snippet for handling the event type.

* __All of these APIs are expected to return a JSON/Dictionary/HashMap response__

__Lifecycle of a GenericResouce__

Consider the following sample GenericResource Template (generated using _crh_helper.sh generate-template_) which helps in creating a cloudformation stack set (there is no pre-defined resource type for this yet).

```
Resources:
  MyStackSet:
    Type: Custom::GenericResource
    Properties:
      ServiceType: boto3
      ServiceToken: !ImportValue 'CustomResourceHelper'
      Service: cloudformation
      Handlers:
        Create:
          Api: create_stack_set
          PropertyNameForPhysicalResourceId: StackSetName
        CommonsForCreateAndUpdate:
          StackSetName: DemoStackSet
	  TemplateURL: http://url1
        Update:
          Api: update_stack_set
        Delete:
          Api: delete_stack_set
```

__Create Event__: We first create a cloudformation stack with this template, and the following actions take place:

* Since the event is 'Create', GCR will execute the Api defined in __Handlers/Create__, that is __create_stack_set__
* 'Create' has a special property called __PropertyNameForPhysicalResourceId__ (Optional) which defines which property value to use as [PhysicalResourceId](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html). In this case it is 'StackSetName'.
* GCR sends a __SUCCESS__ response to cloudformation indicating the resource created successfully with PhysicalResourceId as 'GCR-MyStackSet-DemoStackSet' (the format here is GCR-<LogicalResourceId>-<PhysicalResourceId>)

__Update Event__: We update the __TemplateURL__ from http://url1 to http://url2 and perform an update on the stack. The following actions take place:

* Since the event is 'Update', GCR will execute the Api defined in __Handlers/Update__, that is __update_stack_set__
* GCR sends a __SUCCESS__ response to cloudformation with __same physical id__ as used in Create, indicating the resource has been updated successfully.

__Replacement Update__: Say we update __StackSetName__, from "DemoStackSet" to "DemoStackSet2", which is an __immutable property__ because it is used as __PropertyNameForPhysicalResourceId__. The following actions take place:

* GCR detects that an immutable property has changed and marks the update for __Replacement__
* Since it is marked for Replacement, GCR first creates a new stack set with new StackSetName using __Handlers/Create__ api (create_stack_set)
* GCR __changes the PhysicalResourceId__ from 'GCR-MyStackSet-DemoStackSet' to __'GCR-MyStackSet-DemoStackSet2'__
* GCR sends __SUCCESS__ to cloudformation with new physical resource id.

--- Cloudformation detects that the physcial id has changed and considers the update as __Replacement__. See [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cfn-customresource.html#w2ab2c21c10d182c19). Hence, sends a 'Delete' event with old properties, which means it will send a 'Delete' event with 'StackSetName' as 'DemoStackSet'.

* GCR uses the api in __Handlers/Delete__ which is delete_stack_set to delete the stackset
* GCR sends __SUCCESS__ signal to cloudformation

__Delete__: When we delete the stack or remove the resource from the template and update the stack, cloudformation sends a 'Delete' event to GCR. The following actions take place.

* GCR uses the api in __Handlers/Delete__ which is delete_stack_set to delete the stackset
* GCR sends __SUCCESS__ signal to cloudformation

__Managing PhysicalResourceId__
____________
When you create a resource, one of most important property for you to define would be PhysicalResourceId. Eg: EC2 Instance ID.

By default, GCR tries to fetch for fields like 'id' and 'arn' in the output of the API call to determine the PhysicalResourceId. If these fields are not found, CRH will generate a random UUID and use it as physical ID for the entire lifecycle of the resource. However, one situation where you will need to change this behavior is __Replacement Update__. In Replacement Update you would like to create a new resource instead of updating the existing one. This happens when you try to change any immutable property of the resource like "name", "id" etc. To let cloudformation know that you have performed a replacement of the resource instead of updating the existing one, you need to change the physical ID that you send as part of the response, so that cloudformation can detect the replacement and send a 'Delete' event for old resource to be deleted. Check [Replacement Update](#ReplacementUpdate) for more details.

* __PhysicalResourceId__: provide a [search query](#SearchQueries). The result value is used as PhysicalResourceId.

```
MyResource:
  Type: Custom::GenericResource
  Properties:
    Service: codestar
    Handlers:
      Create:
        Api: create_project
        Arguments:
          id: projectid     
          name: My Project Name
        PhysicalResourceId: "JQ::.id"      # create_project API call returns JSON {'id': 'projectid' ...}
   ...

# When this template is executed and resource is created it will return Physcial ID using 'projectid'
```

* __PropertyNameForPhysicalResourceId__: provide a name of the property that is fetched from the output of the API and the value is used as PhysicalResourceId.

Eg:

```
MyResource:
  Type: Custom::GenericResource
  Properties:
    Service: codestar
    Handlers:
      Create:
        Api: create_project
        Arguments:
          id: projectid     
          name: My Project Name
        PropertyNameForPhysicalID: id
      ...

# When this template is executed and resource is created it will return Physcial ID using 'projectid'
```

**BEWARE OF PhysicalResourceId** If you provide a query whose values changes during the updates, then it will trigger a "Replacement Update" and cloudformation will send 'Delete' event to your resource. Make sure you provide a proper tested query for physical ID, or leave it blank so that a default is generated automatically.

<a name="ReplacementUpdate"></a>
__How to setup Update Replacement__
____________
Update replacement means, creating a new resource and deleting the old resource when an immutable property like id, name etc are changed in the template. You can define immutable properties using:

* __ReplaceOn__: A list of immutable property names
```
MyResource:
  Type: Custom::GenericResource
  Properties:
    Service: codestar
    Handlers:
      ReplaceOn: ['id'] # Perform Update Replacement if 'id' is changed
      SkipEvents: ['Delete'] # Do not delete the project ever
      Create:
        Api: create_project
        Arguments:
          id: projectid     
          name: My Project Name
      Update:
        Api: update_project
        Arguments:
          id: projectid2         # --> 'projectid' is changed to 'projectid2', trigger 'Replacement Update'
          name: MyProjectNewName
```

<a name="ConvenienceProperties"></a>
__Grouping Common Properties__
____________
You also have convinience properties called __Commons__ and __CommonsForCreateAndUpdate__ to group common properties for handlers to make the template look more consise. It is optional.

For example the above template can be more consise:
```
MyResource:
  Type: Custom::GenericResource
  Properties:
    Service: codestar
    Handlers:
      ReplaceOn: ['id'] # Perform Update Replacement if 'id' is changed
      Commons:
        Arguments:
          id: projectid                # Arguments/id is merged into 'Create', 'Update', 'Delete'
      CommonsForCreateAndUpdate:
        Arguments:
          name: My Project Name        # Arguments/name is merged into 'Create' and 'Update' only
      Create:
        Api: create_project
      Update:
        Api: update_project
      Delete:
        Api: delete_project

=== Equivalent Template ===
MyResource:
  Type: Custom::GenericResource
  Properties:
    Service: codestar
    Handlers:
      ReplaceOn: ['id'] # Perform Update Replacement if 'id' is changed
      Create:
        Api: create_project
        Arguments:
          id: projectid
          name: My Project Name
      Update:
        Api: update_project
        Arguments:
          id: projectid
          name: My Project Name
      Delete:
        Api: delete_project
        Arguments:
          id: projectid        # Only 'id' is merged here
```

__Introducing CRH::Resp__
____________
This new intrinsic function can be used to query values from the response of the Create/Update/Delete API. It accepts a [search query](#SearchQueries).

```
MyResource:
  Type: Custom::GenericResource
  Properties:
    Service: codestar
    Handlers:
      ReplaceOn: ['id'] # Perform Update Replacement if 'id' is changed
      Create:
        Api: create_project
        Arguments:
          id: projectid
          name: My Project Name
        PhysicalResourceId:
          CRH::Resp: ".name"    # Gets 'name' property value from output of create_project
```

__Example__: Manage a CodeStar project with Custom Resource.

```
Resources:
  CodeStarProject:
    Type: Custom::GenericResource
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      Service: codestar
      Handlers:
        ReplaceOn: ['id'] # Property means changes to 'id' will trigger 'Replacement'
        Commons:
          Arguments:
            id: project-id-01
        CommonsForCreateAndUpdate:
          Arguments:
            name: MyFirstProject # If you change the name here it will trigger an update
        Create:
          Api: create_project
        Update:
          Api: update_project
        Delete:
          Api: delete_project
```

__Example__: Manage ECR Life Cycle Policy of ECR repository created outside cloudformation.

```
Resources:
  ECRLifeCyclePolicy:
    Type: Custom::GenericResource
    Properties:
      ServiceType: boto3
      ServiceToken: !ImportValue 'CustomResourceHelper'
      Service: ecr
      Handlers:
        Commons:
          Arguments:
            repositoryName: MyExistingRepository
        CommonsForCreateAndUpdate:
          Arguments:
            lifecyclePolicyText: <JSONSTRING>
        Create:
          Api: put_lifecycle_policy
        Update:
          Api: put_lifecycle_policy
        Delete:
          Api: delete_lifecycle_policy
```

<a name="GRWaitForState"></a>
##### WaitForState in GenericResource
____________
Boom! You can use WaitForState in GenericResource. After executing a particular handler if you want to wait until the created/updated/deleted resource is completely stabilized (reached desired state) you could do it using this method.

WaitForState works exactly the same way as described [earlier](#WaitForState).

```
MyResource:
  Type: Custom::GenericResource
  Properties:
    Service: iam
    Handlers:
      Create:
        Api: create_service_role
        Arguments:
          name: My Service Role
        WaitForState:
          Api: describe_service_role
          Arguments:
            id:
              CRH::Resp: ".id"
          SuccessOn: ...
```

Similar to __CRH::Resp__, you can use __CRH::WaitResp__ intrinsic function to query values from the output of WaitForState Api execution. __CRH::WaitResp__ accepts [search query](#SearchQueries).

__Example__: Take backup of DynamoDB Table and wait until backup is ACTIVE before proceeding to delete DynamoDB Table. Also, output the size of the backup which is provided as stack outputs.

```
...
Resources:
  MyDynamoDBTable:
    Type: AWS::DynamoDB::Table
    Properties:
      ...
  DynamoDBBackup:
    Type: Custom::GenericResource
    Properties:
      ServiceToken: !ImportValue CustomResourceHelper
      Service: dynamodb
      SkipEvents: ['Create', 'Update'] # You can skip particular events
      Handlers:
        Delete:
          Api: create_backup
          Arguments:
            TableName: !Ref MyDynamoDBTable
            BackupName:
              CRH::Join: ["-", ["backup", "${CRH::EpochSeconds}", !Ref MyDynamoDBTable]]
            WaitForState:
              Api: describe_backup
              Arguments:
                BackupArn:
                  CRH::Resp: "Search::.BackupDetails.BackupArn"
              SuccessOn: ["JQ::.BackupDescription.BackupDetails.BackupStatus", "eq", "ACTIVE"]
      Outputs:
        BackupSize:
          CRH::WaitResp: "JQ::.BackupDescription.BackupDetails.BackupSizeBytes"
...
```
<a name="Timer"></a>
##### Timer
Timer resource type allows you to monitor execution of a given resource within the same stack (or even different stack) and if Timeout exceeds it would fail itself causing the entire stack operation to rollback.

__Example__: You are testing your new application and you would like to rollback the stack if AWS::ECS::Service resource takes more than 5 minutes to update/stabilize.

Eg:
```
Parameters:
  TaskDefinition:
    Type: String
Resources:
  StackLevelTimeout:
    Type: Custom::Timer
    Properties:
      TimerTimeout: 5
      TaskDefinition: !Ref TaskDefinition # Ensuring that timer resource is started whenever TaskDefinition is changed.
      ServiceToken: !ImportValue 'CustomResourceHelper'
  ECSService:
    Type: AWS::ECS::Service
    Properties:
      TaskDefinition: !Ref TaskDefinition

Result: If update to ECSService does not complete in 5 min, StackLevelTimeout will timeout causing the entire stack to rollback to previous state.
```

<a name="ResourceOutputs"></a>
## Resource Outputs
____________
For any Resource Type, you can define __Outputs__ which can be fetched using [Fn::GetAtt](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html) in the cloudformation template. Define all the outputs under __Properties__ -> __Outputs__ section as follows:

__Example__: Outputs with Custom::Resolver
```
Resources:
  MyGCRResource:
    Type: Custom::Resolver
    Properties:
      InputNumber: 3
      Outputs:
	SquareOfInput:
	  CRH::Python: |
	    properties['InputNumber'] ** 2
	WelcomeMessage:
	  CRH::Join:
	    - "***"
	    -
	      - Welcome
	      - Customer
Outputs:
  MyOutput:
    Value: !GetAtt MyGCRResource.SquareOfInput            # --> 9
  MyOutput2:
    Value: !GetAtt MyGCRResource.WelcomeMessage           # --> Welcome *** Customer
```

__Example__: Outputs with Custom::WaitForState and CRH::WaitResp
```
Resources:
  MyGCRResource:
    Type: Custom::WaitForState
    Properties:
      Service: iam
      Api: get_user
      Arguments:
      	UserName: bob
      Outputs:
	UserArn:
	  CRH::WaitResp: "JQ::User.Arn"
Outputs:
  MyOutput:
    Value: !GetAtt MyGCRResource.UserArn            # --> Arn of user 'bob'
```

__Example__: Outputs with Custom::GenericResource and CRH::Resp
```
Resources:
  MyGCRResource:
    Type: Custom::GenericResource
    Properties:
      Service: iam
      Handlers:
      	Create:
	  Api: create_group
	  Arguments:
	    GroupName: admingroup
      Outputs:
	GroupArn:
	  CRH::Resp: "JQ::Group.Arn"

Outputs:
  MyOutput:
    Value: !GetAtt MyGCRResource.GroupArn            # --> Arn of group 'admingroup'
```

__Example__: Outputs with Custom::GenericResource using WaitForState
```
Resources:
  MyGCRResource:
    Type: Custom::GenericResource
    Properties:
      Service: iam
      SkipEvents: ['Update', 'Delete']
      __MACROS__:
        GroupName: admingroup
	UserName: bob
      Handlers:
      	Create:
	  Api: add_user_to_group
	  Arguments:
	    UserName: ${CRH::UserName}
	    GroupName: ${CRH::GroupName}
	  WaitForState:
	    Api: get_group
	    Arguments:
	      GroupName: ${CRH::GroupName}
      Outputs:
	GroupName:
	  CRH::Resp: "JQ::.GroupName"
	NumOfUsers:
	  CRH::WaitResp: "JQ::.Users[] | length"

Outputs:
  GroupName:
    Value: !GetAtt MyGCRResource.GroupName            # --> Name of group 'admingroup'
  NumberOfUsers:
    Value: !GetAtt MyGCRResource.NumOfUsers            # --> Number of users in group 'admingroup'
```

<a name="AssumeRoles"></a>
## Assume Roles
____________
While working with boto3, ScriptExecutor or [Input Stores](#IOStores), we will need to make boto3 API calls. By default, boto3 uses the Execution Role attached to the GCR lambda function. If you need to work with services which the Execution Role does not have access to, or work with cross account resources etc, then you have two options:

1) Add permissions to the role attached to the Lambda Function. This can be done by updating the crh-serverless.template file in the source code and running 'build_crh.sh' to deploy the changes.

2) Use __AssumeRoleConfig__ property to specify the RoleArn to be used in the resource. Assume Role Config accepts a property name 'RoleArn'. This config can be specified at different levels.

* For Resolver/Delay/Timer/WaitForState this can be specified at __Top Level__:

```
MyCustomResource:
  Type: Custom::Resolver # (or Delay or Timer or WaitForState)
  Properties:
    AssumeRoleConfig:
      RoleArn: <priviliged role arn>
```

* For GenericResource type this can be specified at __Top Level__ and __Handler Level__:
```
MyCustomResource:
  Type: Custom::GenericResource
  Properties:
    AssumeRoleConfig:
      RoleArn: <MyRoleArn>
    Service: iam
    Handlers:
      Create:
        Api: create_user                      # --> This Api uses <RoleArnForCreate> from Handler Level
	AssumeRoleConfig:
	  RoleArn: <RoleArnForCreate>               
	...
      Update:
      	Api: update_user                      # --> This Api uses <MyRoleArn> from Top Level
      Delete:
        Api: delete_user                      # --> This Api uses <MyRoleArn> from Top Level
```

* For GenericResource with WaitForState type this can be specified at __Top Level__ and __Handler Level__ and __WaitForState Level__:
```
MyCustomResource:
  Type: Custom::GenericResource
  Properties:
    AssumeRoleConfig:
      RoleArn: <MyRoleArn>
    Service: iam
    Handlers:
      Create:
        Api: create_user                      # --> This Api uses <RoleArnForCreate> from Handler Level
	AssumeRoleConfig:
	  RoleArn: <RoleArnForCreate>               
	WaitForState:
	  Api: get_user                       # --> This Api uses <MyRoleArn> from WaitForState Level
	  AssumeRoleConfig:
	    RoleArn: <RoleArnForGet>
      Update:
      	Api: update_user                      # --> This Api uses <RoleArnForUpdate> from Handler Level
	AssumeRoleConfig:
	  RoleArn: <RoleArnForUpdate>
	WaitForState:
	  Api: get_user                       # --> This Api uses <RoleArnForUpdate> from Handler Level (its parent)
      Delete:
        Api: delete_user                      # --> This Api uses <MyRoleArn> from Top Level
	WaitForState:
	  Api: get_user                       # --> This Api uses <MyRoleArn> from Top Level
```

<a name="ConnectionConfigurations"></a>
## Connection Configurations
____________
We can specify connection configurations for boto3, HTTP and ScriptExecutor using [Boto3ConnectionConfig](/docs/configuration.md#Boto3ConnectionConfig), [HTTPConnectionConfig](/docs/configuration.md#HTTPConnectionConfig), and [ScriptExecutorConfig](/docs/configuration.md#ScriptExecutorConnectionConfig)

Similar to AssumeRoleConfig, these configs can be specified at __Top Level__, __Handler Level__, or __WaitForState__ level. For details on the levels see [AssumeRoleConfig](#AssumeRoles)

<a name="IOStores"></a>
## Input/Output Stores
____________
##### Input Stores
Input Stores can be used to read values from "S3", other "boto3" or "HTTP" api calls. You can also define outputs which are pushed as JSON to given S3 store (s3 location (bucket+key)) which can later be used in another resource in another stack of another account of another region.

The input stores are downloaded before the actual execution or intrinsic function resolution happens. To retrieve values from input stores you can use __CRH::S3Store__, __CRH::Boto3Store__ and __CRH::HTTPStore__ intrinsic function and all of them accept [search queries](#SearchQueries)

##### Output Store
You can configure an output store which is an S3 bucket/key to which all the properties defined under __S3Outputs__ are pushed in JSON format. This is useful, if you want to push the outputs from StackA to S3 and retrieve them from another stack in another region/account using Input Stores.

Output Store follows a slightly standardized approach when pushing the JSON. It organizes all the outputs in the following hirearchy. This hirearchy helps in avoiding conflicts when the same Output Store bucket/key is used in multiple stacks specially in case of stacksets where same template is deployed in multiple regions/accounts.

```
{
	'AccountId': {
		'RegionName': {
			'StackId': {
				'LogicalResourceId': {
					'S3Outputs': {
						... All your 'S3Outputs' go here
					}
				}
			}
		}
	}
}
```

__Use Cases__: Cross account/region/stack/resource data sharing. Reading / writing secret/sensitive information to S3 buckets. Overcoming size / number limits of cloudformation outputs / export values. Reading attributes of AWS resources which are not exposed via GetAtt etc.

__Example__: Output a large value from _SourceStack_ and read it from _DestStack_

__Notes__:
1) We have a large json in emroutputbucket/data.json which is created by EMR job earlier which needs to be exported as outputs.
2) We define 'S3Outputs' and use __CRH::S3Store__ intrinsic function to read values from input store using JQ
3) All values in 'S3Outputs' are pushed to 'Location' mentioned in 'S3OutputStore' under the path "region_name/stack_id/logical_id/S3Outputs".

__SourceStack__
```
Resources:
  EmitValue:
    Type: Custom::Resolver
    S3InputStores:
      Stores:
	 MyLocalStore1: emroutputbucket/data.json
    S3Outputs:
      LargeValue: # Format .<StoreName>.<PathInside>
        CRH::S3Store: "JQ::.MyLocalStore1.Data.MyLargeValue"
      CrossAccountValue: # Format .<StoreName>.<PathInside>
        CRH::S3Store: "JQ::.MyLocalStore1.Data.CrossAccountValue"
    S3OutputStore:
      Location: sharedemrbucket/data.json
```

Once the above stack is executed you will have values in __sharedemrbucket/data.json__ as follows:
```
{
 "ACCOUNTID-123": {
  "us-east-1": {
    "arn:aws:cloudformation:ap-southeast-2:namespace:stack/stack-name/guid": {
      "EmitValue": {
        "S3Outputs": {
          "LargeValue": <value>,
	  "CrossAccountValue": <value>
       }
     }
  }
 }
}
```

__DestinationStack__
```
Resources:
  GetValue:
    Type: Custom::Resolver
    S3InputStores:
      Stores:
        MyLocalStore: sharedemrbucket/data.json
    __MACROS__: # For convinience
      S3Path: ".ACCOUNTID-123.us-east-1.arn:aws:cloudformation:ap-southeast-2:namespace:stack/stack-name/guid.EmitValue"
    Outputs:
      LargeValue:
        CRH::S3Store: ${S3Path}.S3Outputs.LargeValue
      CrossAccountValue:
        CRH::S3Store: ${S3Path}.S3Outputs.CrossAccountValue
Outputs:
  LargeValue:
    Value: !GetAtt GetValue.LargeValue
  CrossAccountValue:
    Value: !GetAtt GetValue.CrossAccountValue
```

__Example__: Read password from S3 using restricted IAM role for RDS Database

__Notes__:
1) Introducing 'AssumeRoleConfig' which can be used with any resource above as well.
2) RDS Password is available in _orgsecretbucket/passwords.json_ which is readable only by _arn:aws:iam::123456789:role/priviligedrole_.

```
Resources:
  PasswordResource:
    Type: Custom::Resolver
    S3InputStores:
      Stores:
        SecretStore: orgsecretbucket/passwords.json
      AssumeRoleConfig:
        RoleArn: arn:aws:iam::123456789:role/priviligedrole
    Outputs:
      RDSPassword:
        CRH::S3Store: "Search::.SecretStore.RDS.Password"
  RDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: !GetAtt PasswordResource.RDSPassword
      ...
```

__Aim__: _Cross Account Sharing_, read Cloudformation Export _SNSTopicName_ from us-east-1 in AccountA and make it available as exports in ap-southeast-2 in AccountB

__Notes__:
1) Introducing _Boto3InputStores_ and _Boto3ConnectionConfig_ which can be used with any resource using 'boto3' ServiceType above as well.

__Stack in AccountB in region ap-southeast-2__
```
Resources:
  ImportCrossAccountExports:
    Type: Custom::Resolver
    Boto3InputStores:
      Stores:
        CrossAccountExports: ['cloudformation', 'list_exports', {}]
      AssumeRoleConfig:
        RoleArn: arn:aws:iam::ACCOUNT_A_ID:role/crossaccountrole
      Boto3ConnectionConfig:
        region_name: us-east-1
    Outputs:
      ImportedSNSTopicName:
        CRH::Boto3Store: 'JQ::.Exports[] | select(Name == "SNSTopicName") | .Value"
Outputs:
  Name: AccountASNSTopicName
  Value: !GetAtt ImportCrossAccountExports.ImportedSNSTopicName
  Export:
    Name: SNSTopicName
```

<a name="SearchQueries"></a>
##### Search Queries
____________
While working with GCR you will encouter situations where you will need to search / query for a value from:

* 'request' object (CRH::Request)
* 'event->properties' object (CRH::Prop)
* response of API calls executed by GenericResource (CRH::Resp)
* response of API calls executed by WaitForState (CRH::WaitResp)
* S3 Input Stores (CRH::S3Store)
* Boto3 Input Stores (CRH::Boto3Store)
* HTTP Input Stores (CRH::HTTPStore)

To make this possible GCR allows you to pass two types of query strings to these intrinsic functions.

1) __JQ__ (__Recommended__) - (prefix - JQ::<query>) JQ is a popular and powerful JSON query tool/module. GCR uses python's 'jq' module to perform search queries. For query string format and examples see [doc](https://stedolan.github.io/jq/manual/)

2) __Search__ (Memory Optimized (beta)) - (prefix - Search::<query>) Search is implemented explicitly for simple query operations like find value of nested key in dictionary, or find an element in a list. This operation is less memory intensive compared to JQ. However, if your queries are not simple, please use JQ.

When providing query strings, you prefix the string with either "JQ::" or "Search::" to explicitly mention the search query type to use. In case of intrinsic functions, it is __optional__ to provide this prefix. If you do not provide a prefix, GCR will automatically detect and use Search:: for simple queries and JQ:: for non-simple queries.

Eg: Consider the below response returned by boto3 api call for list_roles iam API call. Below is python dictionary object on which we can perform JQ queries.

```
{
	'ResponseMetadata': {
		'RetryAttempts': 0,
		'HTTPStatusCode': 200,
		'RequestId': 'xxx',
		'HTTPHeaders': {
			'x-amzn-requestid': 'xxx',
			'date': 'Mon, 14 May 2018 18:14:23 GMT',
			'content-length': '1090',
			'content-type': 'text/xml'
		}
	},
	'IsTruncated': False,
	'Groups': [{
	  'Path': '/',
		'CreateDate': datetime.datetime(2015, 8, 30, 0, 8, 3, tzinfo = tzutc()),
		'GroupId': 'ABCDEFGHIJKLMNOPQRSTUVWXYZ',
		'Arn': 'arn:aws:iam::ACCOUNTID:group/Administrators',
		'GroupName': 'Administrators'
	}, {
		'Path': '/',
		'CreateDate': datetime.datetime(2015, 7, 6, 1, 20, 14, tzinfo = tzutc()),
		'GroupId': 'ABCDEFGHIJKLMNOPQRSTUVWXYZ',
		'Arn': 'arn:aws:iam::ACCOUNTID:group/Admins',
		'GroupName': 'Admins'
	}, {
		'Path': '/',
		'CreateDate': datetime.datetime(2015, 4, 21, 5, 47, 34, tzinfo = tzutc()),
		'GroupId': 'ABCDEFGHIJKLMNOPQRSTUVWXYZ',
		'Arn': 'arn:aws:iam::ACCOUNTID:group/cloud9demo',
		'GroupName': 'crh-admins'
	}]
}
```

* Fetch Status Code of the API call execution
```
-- using JQ --
JQ::.ResponseMetadata.HTTPStatusCode

-- using Search --
Search::.ResponseMetadata.HTTPStatusCode

(or) (simple query auto-detect Search)

.ResponseMetadata.HTTPStatusCode

---
Return Value: 200
```

* Fetch GroupName of first group in the list
```
-- using JQ --
JQ::.Groups[0].GroupName

-- using Search --
Search::.Groups.0.GroupName                        # --> notice there are no [] around '0' unlike JQ

(or) (simple query auto-detect Search)

.Groups.0.GroupName

---
Return Value: Administrators
```

* Fetch Arn of group whose name is 'Admins'
```
-- using JQ --
JQ::.Groups[] | select(GroupName == "Admins") | .Arn

(or) (non-simple query JQ is default)

.Groups[] | select(GroupName == "Admins") | .Arn

-- NOT POSSIBLE using Search --

---
Return Value: arn:aws:iam::ACCOUNTID:group/Admins
```

* Fetch Number of groups in the result
```
-- using JQ --
JQ::.Groups[] | length

(or) (non-simple query JQ is default)

.Groups[] | length

-- NOT POSSIBLE using Search --

---
Return Value: 3
```
Though you can use intrinsic functions for all properties, there are some special properties which expect search query by default and for convinience you can provide the query string directly instead of using intrinsic functions for those. Those properties are:

* SuccessOn
```
SucessOn:
  - CRH::Resp: ".ResponseMetadata.HTTPStatusCode"
  - 'eq'
  - 200

-- short form (you can omit the CRH::Resp intrinsic function and add JQ:: prefix to query) --
SuccessOn:
  - "JQ::.ResponseMetadata.HTTPStatusCode"
  - 'eq'
  - 200
```
* FailOn
```
FailOn:
  - CRH::Resp: ".ResponseMetadata.HTTPStatusCode"
  - 'ne'
  - 200

-- short form (you can omit the CRH::Resp intrinsic function and add JQ:: prefix to query) --

FailOn:
  - "JQ::.ResponseMetadata.HTTPStatusCode"
  - 'ne'
  - 200
```
* PhysicalResourceId
```
PhysicalResourceId:
  CRH::Resp: ".Groups[] | select(GroupName == "Admins") | .Arn"

-- short form (you can omit the CRH::Resp intrinsic function and add JQ:: prefix to query) --

PhysicalResourceId: "JQ::.Groups[] | select(GroupName == "Admins") | .Arn"

```
<a name="CommandLine"></a>
## Command Line
___________
GCR provides command line utility __crh_helper.sh__ for maintenance tasks.
<a name="GenerateSampleTemplate"></a>
##### Generate Sample Template
```
$ bash crh_helper.sh generate-template


----------Support Custom Resource Types----------
1  ) Delay                                              
2  ) Resolver                                           
3  ) Timer                                              
4  ) WaitForState                                       
5  ) GenericResource                                    

------------------------------
Resource Type ?
```

Start by providing inputs and a sample template will be generated. Fill in the required values in the Template and test it before using in production environments.

<a name="GetLogs"></a>
##### Get Logs of custom resource Execution
GCR is executed in Lambda environment and hence all the logs are pushed to cloudwatch by default. To fetch the logs of particular custom resource execution use:
```
bash crh_helper.sh logs -s <stackname> -l <LogicalResourceId> --region <RegionName>
```
<a name="ManualCFNSignal"></a>
##### Send manual CFN Signal

You can manually send CFN signal SUCCESS/FAILED for any custom resource in any stack as long as you have logged the event object into cloudwatch logs. GCR by default logs the event object so it is possible to send signals to GCR custom resources always.

```
bash crh_helper.sh signal -s <stackname>
```
<a name="RunLocal"></a>
##### Testing GCR Template Execution locally
crh_helper.sh provides a basic lambda context to be able to run your cloudformation templates locally. It runs in DryRun mode by default and you change this using __--no-dry-run__ (GCR executes handlers) and __--no-aws-dry-run__ (actual aws API calls are made) flags.

```
bash crh_helper.sh runlocal -t <templatefilepath>
```

<a name="Configuration"></a>
## Configuration
____________
Go through the configuration [here](/docs/configuration.md)

<a name="Architecture"></a>
## Architecture

<a name="Diagram"></a>
##### Flow Construct
```
[Cloudformation Template]
  [Cloudformation sends Create/Update/Delete event with ResourceProperties]
    [GCR Lambda receives the event]
      [index.py/lambda_handler function gets executed]
        [lambda_handler calls -> handlers/proxy.py/handle_request function to handle the request]
	  [handle_request performs phase0 validations]
	  [handle_request performs phase1 resolutions]
	  [handle_request performs post phase1 validations]
	  [handle_request checks for Type of resource and calls appropraite handler method to start phase2]
	[lambda_handler calls -> handlers/proxy.py/close_request to close the request]
	  [close_request marks phase2 as complete]
	  [close_request resolves 'Outputs' and 'S3Outputs' and uploads 'S3Outputs' if configured]
	  [close_request sends CFN response to cloudformation with SUCCESS/FAILURE message]
	[lambda_handler finishes execution]
```

<a name="TemplateProcessor"></a>
##### Template processor
GCR process the template in multiple phases.

__Phase0__: Perform basic validations and download input stores if configured
__Phase1__: Validate all properties and their semantics and Resolve all intrinsic functions (except for CRH::Resp and CRH::WaitResp as they depend on output of API calls which are done in phase2)
__Phase2__: Perform actual API calls. This is where the resource creation/updation/deletion happens.
__Phase3__(close phase): Resolve 'Outputs' and 'S3Outputs' which can now use CRH::Resp and CRH::WaitResp function as Phase2 is complete. Once resolved, upload 'S3Outputs' to 'S3OutputStore' (if configured) and send SUCCESS or FAILURE message to cloudformation.

<a name="Validations"></a>
##### Validations and Things to watch out for
**Validations**:

GCR validates every input property to ensure it follows the right semantics in Phase1 and gives an appropriate errors message to cloudformation which can be seen in [cloudformation stack events](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-listing-event-history.html).

**Watch Out for Auto Type Conversions**:

Cloudformation by default converts all property values to 'string' before passing it to custom resource. I believe this is for security reasons. To work around this, GCR converts the data type of all input properties to their normal types using the following logic:

```
MyResource:
  Type: Custom::Resolver
  Properties:
    IntegerValue: 1
    FloatValue: 0.2
    BooleanValue: true
    StringValue: GCR
    ...

# When this resource gets executed we get 'event' object as follows:

{
	...
	'ResourceProperties': {
		"IntegerValue": "1",
		"FloatValue": "0.2",
		"BooleanValue": "true",
		"StringValue": "GCR",
		"None": --> You cannot pass Nones in Cloudformation Templates instead use,
		"None": "none"
	}
}
```

As you can see all the datatypes are converted to string types. To work around this GCR performs the following action:

* If value is a digit, convert to integer. Eg: "IntegerValue" is converted to 1.
* If value is a boolean, convert to boolean. Eg: "BooleanValue" is converted to true.
* If value is string 'none', convert to null. Eg: "None" is converted to null (or None in python).
* If value is a float, convert to float. Eg: "FloatValue" is converted to 0.2.

If you do not wish GCR to perform any of these auto type conversions, or want to exclude certain properties from these conversions use __AUTO_TYPE__ property to provide a list of path regex where you would like to avoid these conversions. By default, __AUTO_TYPE__ is set to regex __'^.*$'__ which means convert all paths.

Eg: To convert only "BooleanValue" and "IntegerValue" and ignore the rest.
```
MyResource:
  Type: Custom::Resolver
  Properties:
    __AUTO_TYPE__: [".*(BooleanValue|IntegerValue).*]
    IntegerValue: 1
    FloatValue: 0.2
    BooleanValue: true
    StringValue: GCR
    ...
```

##### Limitations and Workarounds
____________
- GCR uses a single lambda backed custom resource. When executing a single lambda function multiple times at the same time, [concurrency](https://docs.aws.amazon.com/lambda/latest/dg/concurrent-executions.html) comes into picture. By default, GCR reserves a concurrency of 100 which means you can execute 100 custom resources based on GCR per account per region at the same time. To increase the limit increase the Concurrency limit for the GCR lambda function.

- Cloudformation restricts the size of 'Response' from custom resources to 4096 bytes. Which means we cannot send 'Outputs' more than 4096 bytes out of GCR. To workaround this limitation GCR use 'S3Outputs' with S3OutputStore configuration instead.

<a name="Samples"></a>

## Samples
____________
For more basic/intermediate/advanced samples browse [here](/docs/samples.md)

<a name="UnderstandingTheCode"></a>
##### Understanding the Code
___________
Majority of the business logic is implemented in:

* handlers/proxy.py - Handles incoming requests and invokes appropriate handler for the type of request.
* handlers/delay.py - Handles Custom::Delay Type
* handlers/resolver.py - Handles Custom::Resolver Type
* handlers/wait_for_state.py - Handles Custom::WaitForState Type
* handlers/generic_resource.py - Handles Custom::GenericResource Type
* handlers/timer.py - Handles Custom::Timer Type and TimerTimeout property
* helpers/function_resolver.py - Validates, Resolves intrinsic functions and macros
* clients/ -> involves raw clients for boto3, http, I/O stores, scriptexecutor, cfnresponse, lambda etc.
* helpers/utils.py -> Has plethora of utility functions used across all modules

<a name="Contributing"></a>
##### Contributing to project

Please submit your contributions/changes in github by raising pull requests which will be reviewed and merged to master.
