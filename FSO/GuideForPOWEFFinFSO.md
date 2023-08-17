Practical guide to start with FSO

# 1st Day training:

## Install local environment

Get the code running on laptop

mpalmero@MPALMERO-M-X1KS FSO % curl -fSL -o fsoc "https://github.com/cisco-open/fsoc/releases/latest/download/fsoc-darwin-amd64"
mpalmero@MPALMERO-M-X1KS FSO % chmod +x fsoc
mpalmero@MPALMERO-M-X1KS FSO % sudo mv fsoc /usr/local/bin


validate:
mpalmero@MPALMERO-M-X1KS FSO % fsoc version

fsoc version 0.47.0


## Basic commands to configure solution:

### Service vs Agent:

service-principal is based on JSON file, i.e. fso-dpp-enablement-sp.json
vs
agent-principal, test if the platform aspect is ready, is based on YAML file, i.e. fso-dpp-ap.yaml


mpalmero@MPALMERO-M-X1KS FSO % fsoc config set --profile fsodpp auth=service-principal secret-file=/Users/mpalmero/Downloads/fso-dpp-enablement-sp.json 

### Validate
validate:
mpalmero@MPALMERO-M-X1KS FSO % fsoc config list
    USE     NAME      AUTH METHOD     URL  USER  

  Current  fsodpp  service-principal  

mpalmero@MPALMERO-M-X1KS FSO % fsoc config use fsodpp
Switched to context "fsodpp"

mpalmero@MPALMERO-M-X1KS FSO % fsoc login            
✓ Exchange service principal for auth token
Login completed successfully.


mpalmero@MPALMERO-M-X1KS FSO % fsoc config list                                                                                                         
    USE     NAME      AUTH METHOD                       URL                    USER  

  Current  fsodpp  service-principal  https://fso-dpp.observe.appdynamics.com  


note: all of the training members are working on the same tenant, which is part of the json file...

mpalmero@MPALMERO-M-X1KS FSO % fsoc config get
       Name: fsodpp
Auth Method: service-principal
        URL: https://fso-dpp.observe.appdynamics.com
     Tenant: 30387634-1b99-45ff-a253-04109dcee074
      Token: (present)
Secret File: /Users/mpalmero/Downloads/fso-dpp-enablement-sp.json


mpalmero@MPALMERO-M-X1KS FSO % fsoc solution list
knowledge-store/v1/objects/extensibility:solution
✓ Platform API call (GET /knowledge-store/v1/objects/extensibility:solution)
              NAME                TAG    ISSYSTEM  ISSUBSCRIBED                                DEPENDENCIES                                

  extensibility                  stable  true      true          []                                                                        
  environment                    stable  true      true          ["extensibility"]                                                         
  fmm                            stable  true      true          ["iam"]                                                                   
  common                         stable  true      true          ["fmm"]                                                                   
  zodiac                         stable  true      true          []                                                                        
  …

TAG, identify the environment, stable means production…
C-zero, is the integration testing environment… CI life in C-zero is not easy, it is too fast …. this will be the environment that SMARTLOOK will base deployment.

if a solution development needs access to something, that something should be part of a solution
none platform solution vs platform soluiton -> ISSYSTEM is TRUE

dependencies vary a lot!! 
a module, is a solution that extend another solution, for instance "cost insights" …. cost insights cannot exists in its own… because is enriching another domain model, there is a dependency…
subscription model might need to be a bit more flexible… if a user is subscribed can get it all or only one part… for instance one view of the Data… without all the core content


GOAL is to create a small solution: TWO entities that will correlate



## Initialize solution

to create the package, called "slmpalmero":

mpalmero@MPALMERO-M-X1KS FSO % fsoc solution init slmpalmero
Preparing the solution directory structure for "slmpalmero"... 
Adding the manifest.json 


mpalmero@MPALMERO-M-X1KS FSO % cd slmpalmero 
mpalmero@MPALMERO-M-X1KS slmpalmero % ls
manifest.json
mpalmero@MPALMERO-M-X1KS slmpalmero % ls -al
total 8
drwxr-xr-x@ 3 mpalmero  staff   96 Aug 15 11:40 .
drwxr-xr-x  3 mpalmero  staff   96 Aug 15 11:40 ..
-rw-r--r--  1 mpalmero  staff  414 Aug 15 11:40 manifest.json
mpalmero@MPALMERO-M-X1KS slmpalmero % more manifest.json 
{
    "manifestVersion": "1.0.0",
    "name": "slmpalmero",
    "solutionVersion": "1.0.0",
    "dependencies": [],
    "description": "description of your solution",
    "contact": "the email for this solution's point of contact",
    "homepage": "the url for this solution's homepage",
    "gitRepoUrl": "the url for the git repo holding your solution",
    "readme": "the url for this solution's readme file"


## Entity creation

Entity - attributes and relationships definition between the two entities
User ——> Device

Example:
entity - attributes and relationships
User ——> Device

create "user" entity:

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-entity user
Added new fmm:namespace definition to the solution manifest 
Added objects/model/namespaces/slmpalmero.json file to your solution 
Added new fmm:entity definition to the solution manifest 
Added objects/model/entities/user.json file to your solution 


create "device" entity:


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-entity device
Added objects/model/entities/device.json file to your solution 


validate solution:

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution validate --tag dev
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero631977896.zip"
Validating solution slmpalmero version 1.0.0 with tag dev
✓ Platform API call (POST /solution-manager/v1/solutions)
Successfully validated solution slmpalmero version 1.0.0 with tag dev.


Defining the entities, will define the domain model:
we have defined our domain model, NEXT we will be waiting for data to come ...

## Resource Mapping of an entity: 

how to identify an entity? also called instantiate, for that it needs to define the resouce mapping.
note: I like to call it automatic onboarding.
An entity will be identified based on an array of key value pairs.
Same concept as per NetFlow… to identify the entities… but you need to define the key that the tuple will be based on, to be identified as such entity.

An entity is called a resource 
The context is for us an entity = resource mapping


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-resourceMapping user
Added new fmm:resourceMapping definition to the solution manifest 
Added objects/model/resource-mappings/user-resourceMapping.json file to your solution 

To be able to instantiate specific users, we just did resource mapping for the user. We will add telemetry for user, and we will be able to understand data coming from users…


lets validate and push:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution validate --tag dev               
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero237948189.zip"
Validating solution slmpalmero version 1.0.0 with tag dev
✓ Platform API call (POST /solution-manager/v1/solutions)
Successfully validated solution slmpalmero version 1.0.0 with tag dev.

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution push --tag dev
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero1610594930.zip"
Deploying solution slmpalmero version 1.0.0 with tag dev
✓ Platform API call (POST /solution-manager/v1/solutions)
Successfully uploaded solution slmpalmero version 1.0.0 with tag dev.

let’s subscribe to it, to be able to operate with it, where Tenant will appear as subscribed to the solution:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution subscribe slmpalmero.dev
✓ Platform API call (PATCH /knowledge-store/v1/objects/extensibility:solution/slmpalmero.dev)
Tenant 30387634-1b99-45ff-a253-04109dcee074 has successfully subscribed to solution slmpalmero.dev


## UI connectivity

credentials to connect to the UI:
URL:  https://fso-dpp.observe.appdynamics.com  
User: fsodeveloper00@gmail.com
Password: @Fs0D3v3l0p3r


a few checks:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc config list
    USE     NAME      AUTH METHOD                       URL                    USER  

  Current  fsodpp  service-principal  https://fso-dpp.observe.appdynamics.com  


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc config get
       Name: fsodpp
Auth Method: service-principal
        URL: https://fso-dpp.observe.appdynamics.com
     Tenant: 30387634-1b99-45ff-a253-04109dcee074
      Token: (present)
Secret File: /Users/mpalmero/Downloads/fso-dpp-enablement-sp.json



mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc melt model
Adding 2 entities to the fsoc data model
Adding 0 metrics to the fsoc data model
Adding 0 events to the fsoc data model
Generating slmpalmero-1.0.0-melt.yaml


## Agent principal 

Agent principal will allow to run unitest for the solution

We need to create a agent principal, to make sure that we have a working model, we can feed information with yaml to ingest data, this servers to run unitest:

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc config set --profile fsodppap auth=agent-principal secret-file=/Users/mpalmero/Downloads/fso-dpp-ap.yaml 

Once created, we need to push the configuration:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc melt push slmpalmero-1.0.0-melt.yaml --profile fsodppap
Generating new MELT telemetry... 

Exporting metrics... 

Exporting logs... 
✓ Exchange agent principal for auth token
✓ Platform API call (POST /data/v1/logs)

+With this we will be able to send a payload, to feed data to the solution


validate:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc config list
    USE      NAME       AUTH METHOD                       URL                    USER  

  Current  fsodpp    service-principal  https://fso-dpp.observe.appdynamics.com        
           fsodppap  agent-principal    https://fso-dpp.observe.appdynamics.com  


Create a resource mapping for entity "device":

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-resourceMapping device
Added objects/model/resource-mappings/device-resourceMapping.json file to your solution 

And then push to the solution:

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution push --bump --tag dev
Solution version updated to 1.0.1
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero2067550887.zip"
Deploying solution slmpalmero version 1.0.1 with tag dev
   • Current token is no longer valid; trying to refresh
✓ Exchange service principal for auth token
✓ Platform API call, retry after login (POST /solution-manager/v1/solutions)
Successfully uploaded solution slmpalmero version 1.0.1 with tag dev.



validate:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution status slmpalmero --tag dev
⠦ Platform API call (GET /knowledge-store/v1/objects/extensibility:solutionRelease?order=desc&filter=data.solutionName+eq+%22slmpalmero%22+and+data.tag+eq+%22d✓ Platform API call (GET /knowledge-store/v1/objects/extensibility:solutionRelease?order=desc&filter=data.solutionName+eq+%22slmpalmero%22+and+data.tag+eq+%22dev%22&max=1)
✓ Platform API call (GET /knowledge-store/v1/objects/extensibility:solutionInstall?order=desc&filter=data.solutionName+eq+%22slmpalmero%22+and+data.tag+eq+%22dev%22&max=1)
               Solution Name: slmpalmero
Solution Subscription Status: Subscribed
     Solution Upload Version: 1.0.1
            Upload Timestamp: 2023-08-15T13:34:34.943Z
    Solution Install Version: 1.0.1
Solution Install Successful?: true
       Solution Install Time: 2023-08-15T13:34:42.036Z
    Solution Install Message: 



Now we can create/generate a 2nd yaml file because the version deployed has changed, this time based nn 1.0.1:
We can generate a 2nd yaml file (because of the version has changed, we create a new yaml based on 1.0.1

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc melt model 
Adding 2 entities to the fsoc data model
Adding 0 metrics to the fsoc data model
Adding 0 events to the fsoc data model
Generating slmpalmero-1.0.1-melt.yaml

After making any changes to the yaml file, we can push yaml file to perform unitest, and run queries:

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc melt push slmpalmero-1.0.1-melt.yaml --profile fsodppap
Generating new MELT telemetry... 

Exporting metrics... 

Exporting logs... 
✓ Platform API call (POST /data/v1/logs)


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc uql "fetch id, attributes from entities(slmpalmero:user).out.to(slmpalmero:device)"
✓ Platform API call (POST /monitoring/v1/query/execute)
 id | attributes       
    | name | value     
=======================


## Entity association:

How to associate or link entities which have been defined?
this can be done from the Association declaration and derivation:

(a) fastest is DECLARATION, it uses resource mapping . Based on the owner … in our case the user

(b) DERIVATION, a bit more tricky, as there is no warranty that both entities will be in the cash, normally is quering for the parent


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-associationDeclarations user
Added new fmm:associationDeclaration definition to the solution manifest 
Added objects/model/association-declarations/user-associationDeclarations.json file to your solution 

file user-associationDeclarations.json:

[
    {
        "namespace": {
            "name": "slmpalmero",
            "version": 1
        },
        "kind": "associationDeclaration",
        "name": "user_to_device_relationship",
        "displayName": "Declared Relationship between user and device",
        "scopeFilter": "true",
        "fromType": "slmpalmero:user",
        "toType": "slmpalmero:device",
        "associationType": "common:consists_of"
    }
]



note: "scopeFilter" is set to TRUE


push and validate:

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution push --bump --tag dev
Solution version updated to 1.0.2
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero2200713976.zip"
Deploying solution slmpalmero version 1.0.2 with tag dev
✓ Platform API call (POST /solution-manager/v1/solutions)
Successfully uploaded solution slmpalmero version 1.0.2 with tag dev.
mpalmero@MPALMERO-M-X1KS slmpalmero % 


check if deployment is ok:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution status slmpalmero --tag dev                                               
⠄ Platform API call (GET /knowledge-store/v1/objects/extensibility:solutionRelease?order=desc&filter=data.solutionName+eq+%22slmpalmero%22+and+data.tag+eq+%22d⠤ Platform API call (GET /knowledge-store/v1/objects/extensibility:solutionRelease?order=desc&filter=data.solutionName+eq+%22slmpalmero%22+and+data.tag+eq+%22d✓ Platform API call (GET /knowledge-store/v1/objects/extensibility:solutionInstall?order=desc&filter=data.solutionName+eq+%22slmpalmero%22+and+data.tag+eq+%22d✓ Platform API call (GET /knowledge-store/v1/objects/extensibility:solutionRelease?order=desc&filter=data.solutionName+eq+%22slmpalmero%22+and+data.tag+eq+%22dev%22&max=1)
               Solution Name: slmpalmero
Solution Subscription Status: Subscribed
     Solution Upload Version: 1.0.2
            Upload Timestamp: 2023-08-15T14:01:25.573Z
    Solution Install Version: 1.0.2
Solution Install Successful?: true
       Solution Install Time: 2023-08-15T14:01:27.354Z
    Solution Install Message: 


push yaml file, which now might need to contain the relationship between user and device.

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc melt push slmpalmero-1.0.2-relationship.yaml --profile fsodppap
Generating new MELT telemetry... 

Exporting metrics... 

Exporting logs... 
   • Current token is no longer valid; trying to refresh
✓ Exchange agent principal for auth token
✓ Platform API call, retry after login (POST /data/v1/logs)


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc uql "fetch id, attributes from entities(slmpalmero:user).out.to(slmpalmero:device)"
✓ Platform API call (POST /monitoring/v1/query/execute)
 id                                       | attributes               
                                          | name         | value     
=====================================================================
 slmpalmero:device:kU/nvs53MbuaLTwW/TPV1A | serialnumber | XCVYU     
                                          | os           | iOS       
                                          | name         | iPhone 13 
------------------------------------------+--------------+-----------

                       
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc uql "fetch id, attributes from entities(slmpalmero:device).in.from(slmpalmero:user)"
✓ Platform API call (POST /monitoring/v1/query/execute)
 id                                     | attributes        
                                        | name   | value    
============================================================
 slmpalmero:user:M1ZZ0pmsPbS9ViNzuXofrA | userid | mpalmero 
----------------------------------------+--------+----------



mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc uql "from u: entities(slmpalmero:user), d: u.out.to(slmpalmero:device) fetch u.id, u.attributes, d.events(logs:generic_record)"

✓ Platform API call (POST /monitoring/v1/query/execute)
 id                                     | attributes            | events                                                                                    
                                        | name   | value        | timestamp                         | raw                                                   
============================================================================================================================================================
 slmpalmero:user:qUtJyBX3OTCCA6kmMd/6LA | userid | NEW_mpalmero | 2023-08-15 14:46:46.024 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |--------+--------------| 2023-08-15 14:46:46.024 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:42:59.779 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:42:59.779 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:42:59.779 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:42:59.779 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
----------------------------------------+--------+--------------+-----------------------------------+-------------------------------------------------------
 slmpalmero:user:M1ZZ0pmsPbS9ViNzuXofrA | userid | mpalmero     | 2023-08-15 14:46:46.024 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |--------+--------------| 2023-08-15 14:46:46.024 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:45:48.196 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:33:09.558 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-1 for an entity of type slmpalmero:device 
                                        |                       | 2023-08-15 14:09:22.579 +0000 UTC | hello world-0 for an entity of type slmpalmero:device 
----------------------------



# 2nd day training:

## Associations declaration & derivation:
• when semantics are not there to use the resource mapping, or
• if the context of telemetry is not the owner of the association, the child might be the owner of the association.

Two options:
### Option 1: Declaration
process , OTLP compliant
common ingestion service CIS
fsoc takes yaml and converted to OTLP and send it to CIS

### Option 2: Derivation
data collector being part of FSO platform, to send data directly if that module is set!
Declare a metric in order to receive the data.
It is not yet ready. It is part of the dynamic metric and FMM driven:
FSO will be able to make up the metric even if it is not defined.

### Declaration
metric mapper… 2nd step
attributes on the metrics, with units, to allow units conversions, and also other attributes like allowing slices…

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-metric batterylevel
Added new fmm:metric definition to the solution manifest 
Added objects/model/metrics/batterylevel.json file to your solution 

### batterylevel.json file:

{
    "namespace": {
        "name": "slmpalmero",
        "version": 1
    },
    "kind": "metric",
    "name": "batterylevel",
    "displayName": "batterylevel",
    "category": "current",
    "contentType": "gauge",
    "aggregationTemporality": "unspecified",
    "isMonotonic": false,
    "type": "long",
    "unit": ""
}



### slmpalmero-1.0.3-melt.yaml file
note: adding batterylevel metric to be visualized

melt:
- typename: slmpalmero:device
  attributes:
    slmpalmero.device.name: "iPhone 13 NEW!"
    slmpalmero.device.os: "iOS"
    slmpalmero.device.serialnumber: "XCVYUZZ"
  metrics:
  - typename: slmpalmero:batterylevel
    contenttype: gauge
    unit: ""
    type: long
  logs:
  - body: hello world-0 for an entity of type slmpalmero:device
    severity: INFO
    attributes:
      level: info
  - body: hello world-1 for an entity of type slmpalmero:device
    severity: INFO
    attributes:
      level: info
  relationships: []
  spans: []
- typename: slmpalmero:user
  attributes:
    slmpalmero.user.age: 20
    slmpalmero.user.name: "Marisolita"
    slmpalmero.user.userid: "NEW_mpalmero"
  metrics: []
  logs:
  - body: hello world-0 for an entity of type slmpalmero:user
    severity: INFO
    attributes:
      level: info
  - body: hello world-1 for an entity of type slmpalmero:user
    severity: INFO
    attributes:
      level: info
  relationships: []
  spans: []

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc melt push slmpalmero-1.0.3-melt.yaml --profile fsodppap 
Generating new MELT telemetry... 

Exporting metrics... 
✓ Platform API call (POST /data/v1/metrics)

Exporting logs... 
✓ Platform API call (POST /data/v1/logs)



mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc uql "fetch id, attributes, metrics(slmpalmero:batterylevel) from entities(slmpalmero:device)"
✓ Platform API call (POST /monitoring/v1/query/execute)
 id                                       | attributes                    | metrics                                           
                                          | name       | value        | source    |  metrics                               
                                          |                               |           | timestamp                     | value 
==============================================================================================================================
 slmpalmero:device:kU/nvs53MbuaLTwW/TPV1A | serialnumber | XCVYU          |                                                   
                                          | os           | iOS            |                                                   
                                          | name         | iPhone 13      |                                                   
------------------------------------------+--------------+----------------+-----------+-------------------------------+-------
 slmpalmero:device:nThBCOJtP02bMMBJpyAxOA | serialnumber | XCVYUZZ        | fsoc-melt | 2023-08-16 08:29:00 +0000 UTC | 22    
                                          | os           | iOS            |           | 2023-08-16 08:30:00 +0000 UTC | 34    
                                          | name         | iPhone 13 NEW! |           | 2023-08-16 08:31:00 +0000 UTC | 12    
                                          |--------------+----------------|           | 2023-08-16 08:32:00 +0000 UTC | 45    
                                          |                               |           | 2023-08-16 08:33:00 +0000 UTC | 29    
                                          |                               |           | 2023-08-16 08:34:00 +0000 UTC | 36    
                                          |                               |           | 2023-08-16 08:35:00 +0000 UTC | 17    
                                          |                               |           | 2023-08-16 08:36:00 +0000 UTC | 20    
                                          |                               |           | 2023-08-16 08:37:00 +0000 UTC | 19    
------------------------------------------+--------------+----------------+-----------+-------------------------------+-------
mpalmero@MPALMERO-M-X1KS slmpalmero % 


## Events and logs

If comparing events with logs, you can think about log record as an event type, where events are structured and logs are unstructured records.



mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-event slevent
Added new fmm:event definition to the solution manifest 
Added objects/model/events/slevent.json file to your solution 



It creates  objects/model/events/slevent.json

{
    "namespace": {
        "name": "slmpalmero",
        "version": 1
    },
    "kind": "event",
    "name": "slevent",
    "displayName": "slevent",
    "attributeDefinitions": {
        "attributes": {
            "type": {
                "type": "string",
                "description": "The type of the slevent"
            },
            "description": {
                "type": "string",
                "description": "The description of the slevent"
            }
        }
    }
}

And we need to add the event to the user json file:

{
    "namespace": {
        "name": "slmpalmero",
        "version": 1
    },
    "kind": "entity",
    "name": "user",
    "displayName": "user",
    "attributeDefinitions": {
        "required": [
            "userid"
        ],
        "optimized": [],
        "attributes": {
            "name": {
                "type": "string",
                "description": "The name of the user"
            },
            "age": {
                "type": "long",
                "description": "The age of the user"
            },
            "userid": {
                "type": "string",
                "description": "The userid of the user"
            }
        }
    },
    "lifecycleConfiguration": {
        "purgeTtlInMinutes": 4200,
        "retentionTtlInMinutes": 1440
    },
    "associationTypes": {
        "common:consists_of": [
            "slmpalmero:device"
        ]
    },
    "eventTypes": [
        "slmpalmero:slevent"
    ]
}


validate and push:


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution validate --tag dev                                                                                         
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero204160519.zip"
Validating solution slmpalmero version 1.0.3 with tag dev
✓ Platform API call (POST /solution-manager/v1/solutions)
Successfully validated solution slmpalmero version 1.0.3 with tag dev.

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution push --bump --tag dev
Solution version updated to 1.0.4
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero2007476600.zip"
Deploying solution slmpalmero version 1.0.4 with tag dev
✓ Platform API call (POST /solution-manager/v1/solutions)
Successfully uploaded solution slmpalmero version 1.0.4 with tag dev.



To identify the event scheme:
mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc knowledge get-type --type fmm:event

## Dashboard / UI:

Dashboard is not context driven, it is more open approach


It is based on a TEMPLATE, that can be extended:
template is your experience, and you can link this template with another template
properties 


Template is not in FMM
Observe experience object: ecpHome

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-ecpHome
Added new dashui:templatePropsExtension definition to the solution manifest 
Added objects/dashui/templatePropsExtensions/ecpHome.json file to your solution 


in manifest.json is added:
 //
        {
            "type": "dashui:templatePropsExtension",
            "objectsDir": "objects/dashui/templatePropsExtensions"
        }


It also creates an structure under:
creates structure: under "objects" folder, dashui/templatePropsExtensions:
ecpHome.json

{
    "kind": "templatePropsExtension",
    "id": "slmpalmero:slmpalmeroEcpHomeExtension",
    "name": "dashui:ecpHome",
    "view": "default",    >>>>> deprecated
    "target": "*",            >>>>> deprecated
    "requiredEntityTypes": [
        "slmpalmero:device",
        "slmpalmero:user"
    ],
    "props": {.                >>>>> those are parameters
        "sections": [.      >>>>> title of the section under Observe
            {
                "index": 6,
                "name": "slmpalmeroCoreSection",
                "title": "slmpalmero - 1.0.6"
            }
        ],
        "entities": [
            {
                "index": 0,
                "section": "slmpalmeroCoreSection",
                "entityAttribute": "id",
                "targetType": "slmpalmero:device"
            },
            {
                "index": 1,
                "section": "slmpalmeroCoreSection",
                "entityAttribute": "id",
                "targetType": "slmpalmero:user"
            }
        ]
    }
}

to add listing entities:

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-ecpList user
Added new dashui:template definition to the solution manifest 
Added objects/dashui/templates/user/ecpList.json file to your solution 
Added objects/dashui/templates/user/userGridTable.json file to your solution 
Added objects/dashui/templates/user/ecpRelationshipMap.json file to your solution 
Added objects/dashui/templates/user/ecpListInspector.json file to your solution 
Added objects/dashui/templates/user/userInspectorWidget.json file to your solution 
Added objects/dashui/templates/user/ecpName.json file to your solution 

mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution push --bump --tag dev
Solution version updated to 1.0.9
Creating solution zip: "/var/folders/lj/qqgwb1wx5gn_jzvk4_1y1fgh0000gn/T/slmpalmero3485458607.zip"
Deploying solution slmpalmero version 1.0.9 with tag dev
✓ Platform API call (POST /solution-manager/v1/solutions)
Successfully uploaded solution slmpalmero version 1.0.9 with tag dev.


mpalmero@MPALMERO-M-X1KS slmpalmero % fsoc solution extend --add-ecpDetails user 
Added objects/dashui/templates/user/ecpDetails.json file to your solution 
Added objects/dashui/templates/user/userDetailsList.json file to your solution 
Added objects/dashui/templates/user/ecpDetailsInspector.json file to your solution 





there is a bug!!!
objects/dashui/templates/user/userDetailsList.json file when created automatically.

{
    "kind": "template",
    "name": "slmpalmero:userDetailsList",
    "target": "slmpalmero:user",
    "view": "default",
    "element": {
        "instanceOf": "html",
        "elements": [
            {
                "instanceOf": {
                    "name": "logsWidget"
                },
                "source": "derived_metric”.  >>> change to singular! bug!!! 
            }
        ],
        "style": {
            "display": "flex",
            "flexDirection": "column",
            "gap": 12
        }
    }
}


