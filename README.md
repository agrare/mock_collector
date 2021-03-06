# Mock collector for OpenShift
Mock collector for Topological Inventory service

It implements OpenShift (more providers planned) collector and provides fake(generated) data so it also implements simple OpenShift server.   
Based on https://github.com/agrare/openshift-collector sends data through Kafka messaging server to Topological Inventory Persister service, 
which parses received data and saves them as an inventory in database.  

## Usage:

start collector:
`bin/openshift-mock-collector --source <source> --config <type>`

@param `source` is sources.uid from topological_inventory db
(service https://github.com/ManageIQ/topological_inventory-core)

@param `config` [optional] - YAML files in /config/openshift dir (without ".yml")
 - default (default value)
 - small
 - large

    
Example:
```
bin/openshift-mock-collector --source 31b5338b-685d-4056-ba39-d00b4d7f19cc --config small
```    
_Note: Source is manager for this provider (like ExtManagementSystem in ManageIQ)_

Collector has two modes, both can be switched on/off in config file (described later):
* Full refresh
* Event generator

Options in config: `refresh_mode`

### Full refresh

Full refresh generates specified amount of different entities (`amounts` in config file)
and sends them through Swagger API to Kafka.   

*Implemented entity types:*
* namespaces
* nodes
* pods
* templates
* images
* service instances
* cluster service classes
* cluster service plans

Available options in config:
* `amounts/*` - generated amount of each entity type
* `default_limit` - batch size for data sent in one shot
* `multithreading` - entities of different types can be generated in threads/sequentially
* `full_refresh/repeats_count` - number of sequential full refreshes (should generate the same data in db)
* `full_refresh/send_order` - generates entities of one type **after/before** references (links) exist in db    

### Event generator

Runs in specified number of intervals and generates events of type: 
* *add*, *modify*, *delete*

Event contains information about entity in the same structure as OpenShift server.
Event is also sent through Swagger API to Kafka like entity in full refresh. 

At this moment, events can contain only **Pods**.

Available options in config:
* `events/check_interval` - number of seconds between ticks
* `events/checks_count` - number of ticks  
* `events/per_check/add` - number of generated *Add* events per tick
* `events/per_check/modify` - number of generated *Modify* events per tick
* `events/per_check/delete` - number of generated *Delete* events per tick 

**Added** events contains Pods with incrementally generated ID, starting with full-refresh final state.  
**Modified** events always modifies first active(non-archived) entity, changing entity's `resourceVersion`  
**Deleted** events start archiving Pod with ID == 0 and then increments until some active Pod remains.   

## Config
  
All values are documented in YML files located under `config` folder.

## Development files
Following local files can be created:
* bundler.d/Gemfile.dev.rb - specification of local gems
* lib/mock_collector/require.dev.rb - specification of local requires
