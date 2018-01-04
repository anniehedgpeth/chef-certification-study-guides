# Study Sheet for DEPLOYING COOKBOOKS

> The topics for this guide are taken directly from [training.chef.io](https://training.chef.io/static/Deploying_Cookbooks.pdf). I copied the bulleted lists, and I filled in the study information from either my own understanding or from docs.chef.io.

The Deploying Cookbooks badge is awarded when someone proves that they understand 
how to use Chef server to manage nodes and ensure they're in their expected state.

Candidates must show:
 - An understanding of the chef-client run phases.
 - An understanding of using Roles and Environments.
 - An understanding of maintaining cookbooks on the Chef server.
 - An understanding of using knife.
 - An understanding of using how bootstrapping works.
 - An understanding of what policy files are.
 - An understanding of what Chef solo is.
 - An understanding of using search.
 - An understanding of databags.

Here is a detailed breakdown of each area.

# ANATOMY OF A CHEF RUN

## CHEF-CLIENT OPERATION
_Candidates should understand:_
 - chef-client modes of operation
` -z`, `--local-mode`
Run the chef-client in local mode. This allows all commands that work against the Chef server to also work against the local chef-repo.

 - Configuring chef-client 

_A sample client.rb file that contains the most simple way to connect to https://manage.chef.io:_

```
log_level        :info
log_location     STDOUT
chef_server_url  'https://api.chef.io/organizations/<orgname>'
validation_client_name '<orgname>-validator'
validation_key '/etc/chef/validator.pem'
client_key '/etc/chef/client.pem'
```

## COMPILE VS EXECUTE
_The chef-client processes recipes in two phases:_

_First, each resource in the node object is identified and a resource collection is built. All recipes are loaded in a specific order, and then the actions specified within each of them are identified. This is also referred to as the “compile phase”._

_Next, the chef-client configures the system based on the order of the resources in the resource collection. Each resource is mapped to a provider, which then examines the node and performs the necessary steps to complete the action. This is also referred to as the “execution phase”._

_Typically, actions are processed during the execution phase of the chef-client run. However, sometimes it is necessary to run an action during the compile phase. For example, a resource can be configured to install a package during the compile phase to ensure that application is available to other resources during the execution phase._

> Note: Use the chef_gem resource to install gems that are needed by the chef-client during the execution phase.

_Candidates should understand:_
 - What happens during the 'compile' phase?

_The chef-client identifies each resource in the node object and builds the resource collection. Libraries are loaded first to ensure that all language extensions and Ruby classes are available to all resources. Next, attributes are loaded, followed by lightweight resources, and then all definitions (to ensure that any pseudo-resources within definitions are available). Finally, all recipes are loaded in the order specified by the expanded run-list. This is also referred to as the “compile phase”._

 - What happens during the 'execute' phase?

_The chef-client configures the system based on the information that has been collected. Each resource is executed in the order identified by the run-list, and then by the order in which each resource is listed in each recipe. Each resource in the resource collection is mapped to a provider. The provider examines the node, and then does the steps necessary to complete the action. And then the next resource is processed. Each action configures a specific part of the system. This process is also referred to as convergence. This is also referred to as the “execution phase”._


 - What happens when you place some Ruby at the start of a recipe?

_It is compiled during the compile phase._

 - What happens when you place some Ruby at the end of a recipe?
 
_It is compiled during the compile phase._

 - When are attributes evaluated?

_during the compile phase_

 - What happens during a 'node.save' operation?"

_It sends the node object details to the Chef server._

> per https://discourse.chef.io/t/node-save/2080/8
`node.save?` _is used explicitly when information must be saved, even if the chef run fails. Otherwise, default is for the node information to be given to the Chef server after a successful chef-client run._

_A good example is when a random password (e.g. for MySQL) has been generated and applied to the database. If you don’t call node.save and the run fails, then the next run will generate a new password which doesn’t match the password in the database so access to the database will fail._

_By calling `node.save` right after generating the password but before applying it to the database, you ensure that even when the chef run fails later on, the same password will be used in subsequent chef runs._

_Chef doesn't save the node when the run fails because, by default, Chef would have no way of knowing whether that is a desirable thing to do. There may be something in your recipes that you wouldn’t want saved until the full convergence finished._

## RUN_STATUS
_Candidates should understand:_
 - How can you tap into the chef-client run?

https://docs.chef.io/handlers.html#run-status-object
_Use a handler to identify situations that arise during a chef-client run, and then tell the chef-client how to handle these situations when they occur._

 - What is ‘run_status’?

_Tracks various aspects of a Chef run, including the Node and RunContext, start and end time, and any Exception that stops the run. RunStatus objects are passed to any notification or exception handlers at the completion of a Chef run._
_The run_status object is initialized by the chef-client before the report interface is run for any handler. The run_status object keeps track of the status of the chef-client run and will contain some (or all) of the following properties:_

```
all_resources
backtrace
elapsed_time
end_time
exception
failed?
node
run_context
start_time
success?
updated_resources
```

 - What is ‘run_state’?

_The start of the chef-client run_

 - What is the ‘resource collection’?

_During the compile phase of the `chef-client` run, when it is loading all of the cookbooks, all of the resources are compiled and set to run in order._

## AUTHENTICATION
_Candidates should understand:_
 - How does the chef-client authenticate with the Chef Server?

_A client is an actor that has permission to access the Chef server. A client is most often a node (on which the chef-client runs), but is also a workstation (on which knife runs), or some other machine that is configured to use the Chef server API. Each request to the Chef server that is made by a client uses a private key for authentication that must be authorized by the public key on the Chef server._

 - Authentication and using NTP

_In some cases, the chef-client may receive a 401 response to the authentication request and a 403 response to an authorization request. If the authentication is happening on the node, NTP may be a cause. The system clock has drifted from the actual time by more than 15 minutes. This can be fixed by syncing the clock with an Network Time Protocol (NTP) server._

## CHEF COMPILE PHASE
https://docs.chef.io/chef_client_overview.html
_Candidates should understand:_
 - Do all cookbooks always get downloaded?

_No, they get synchronized. The chef-client asks the Chef server for a list of all cookbook files (including recipes, templates, resources, providers, attributes, libraries, and definitions) that will be required to do every action identified in the run-list for the rebuilt node object. The Chef server provides to the chef-client a list of all of those files. The chef-client compares this list to the cookbook files cached on the node (from previous chef-client runs), and then downloads a copy of every file that has changed since the previous chef-client run, along with any new files._

 - What about dependencies, and their dependencies?

_When a chef-client is run, it will perform all of the steps that are required to bring the node into the expected state, including synchronizing cookbooks and compiling the resource collection by loading each of the required cookbooks, including recipes, attributes, and all other dependencies._

 - What order do the following get loaded - libraries, attributes, resources/providers, definitions, recipes?

```
attributes
resources/providers
libraries
definitions
recipes
```

## CONVERGENCE
_Candidates should understand:_
 - What happens during the execute phase of the chef-client run?

_The chef-client configures the system based on the information that has been collected. Each resource is executed in the order identified by the run-list, and then by the order in which each resource is listed in each recipe. Each resource in the resource collection is mapped to a provider. The provider examines the node, and then does the steps necessary to complete the action. And then the next resource is processed. Each action configures a specific part of the system. This process is also referred to as convergence. This is also referred to as the “execution phase”._

 - The test/repair model

_Chef-client "tests" the node to see if it is in the desired state as defined by the compilation of resources. If it is not, then it is "repaired" to be in that desired state._

 - When do notifications get invoked?

_During resource collection - A notification is a property on a resource that listens to other resources in the resource collection and then takes actions based on the notification type (notifies or subscribes)._

 - What happens if there are multiple start notifications for a particular resource?

_It is started the first time and is tested for being started each additional time but not restarted._

 - When is node data sent to the Chef Server?

_after a successful chef-client run unless explicitly told to send it sooner by using_ `node.save?`

## WHY-RUN
_Candidates should understand:_
 - What is the purpose of `why-run`

` -W`, `--why-run`
Use why-run mode to understand why the chef-client makes the decisions that it makes and to learn more about the current and proposed state of the system.

 - How do you invoke a ‘why-run’

`chef-client -W` `chef-client --why-run`

 - What are limitations of doing a why run?

A type of `chef-client` run that does everything except modify the system

# ENVIRONMENTS 

## WHAT IS AN ENVIRONMENT/USE CASES
_Candidates should understand:_
 - What is the purpose of an Environments?

_An environment is a way to map an organization’s real-life workflow to what can be configured and managed when using Chef server.  Additional environments can be created to reflect each organization’s patterns and workflow. For example, creating production, staging, testing, and development environments. Generally, an environment is also associated with one (or more) cookbook versions._

 - What is the '_default' environment?

_Every organization begins with a single environment called the _default environment, which cannot be modified (or deleted)._

 - What information can be specified in an Environment?

_cookbook, description, name, default attributes, override attributes, cookbook versions_

 - What happens if you do not specify an Environment?

_It defaults to using `_default` environment._

 - Creating environments in Ruby

_A Ruby file for each non-default environment must exist in the environments/ subdirectory of the chef-repo. (If the chef-repo does not have this subdirectory, then it should be created.) The complete environment has the following syntax:_
```
name 'environment_name'
description 'environment_description'
cookbook OR cookbook_versions  'cookbook' OR 'cookbook' => 'cookbook_version'
default_attributes 'node' => { 'attribute' => [ 'value', 'value', 'etc.' ] }
override_attributes 'node' => { 'attribute' => [ 'value', 'value', 'etc.' ] }
```
 - Creating environments in JSON

_The JSON format for environments maps directly to the domain-specific Ruby format: the same settings, attributes, and values, and a similar structure and organization, just formatted as JSON. When an environment is defined as JSON the file that contains that data must be defined as a file that ends with .json. For example:_
```
{
  "name": "dev",
  "default_attributes": {
    "apache2": {
      "listen_ports": [
        "80",
        "443"
      ]
    }
  },
  "json_class": "Chef::Environment",
  "description": "",
  "cookbook_versions": {
    "couchdb": "= 11.0.0"
  },
  "chef_type": "environment"
}
```
_The JSON format has two additional settings:_
`chef_type`	_Always set this to environment. Use this setting for any custom process that consumes environment objects outside of Ruby._
`json_class`	_Always set this to Chef::Environment. The chef-client uses this setting to auto-inflate an environment object. If objects are being rebuilt outside of Ruby, ignore it._

 - Using environments within a search

_When searching, an environment is an attribute._
```knife search node "chef_environment:QA AND platform:centos"```

_Or in a recipe_
```qa_nodes = search(:node,"chef_environment:QA")
qa_nodes.each do |qa_node|
    # Do useful work specific to qa nodes only
end
```

## ATTRIBUTE PRECEDENCE AND COOKBOOK CONSTRAINTS
_Candidates should understand:_
 - What attribute precedence levels are available for Environments
 - Overriding Role attributes
 - Syntax for setting cookbook constraints.
 - How would you allow only patch updates to a cookbook within an environment?

## SETTING AND VIEWING ENVIRONMENTS
_Candidates should understand:_
 - How can you list Environments?
 - How can you move a node to a specific Environment?
 - Using `knife exec` to bulk change Environments.
 - Using 'chef_environment' global variable in recipes
 - Environment specific knife plugins, e.g. `knife flip`
 - Bootstrapping a node into a particular Environment

# ROLES

## USING ROLES
_Candidates should understand:_
 - What is the purpose of a Role?
 - Creating Roles
 - Role Ruby & JSON DSL formats
 - Pros and Cons of Roles 
 - The Role cookbook pattern
 - Creating Role cookbooks
 - Using Roles within a search

## SETTING ATTRIBUTES AND ATTRIBUTE PRECEDENCE
_Candidates should understand:_
 - What attribute precedence levels are available for Roles
 - Setting attribute precedence

## BASE ROLE & NESTED ROLES
_Candidates should understand:_
 - What are nested roles?
 - Whats the purpose of a ‘base’ role?

## USING KNIFE
_Candidates should understand:_
 - The `knife role` command
 - How would you set the run_list for a node using `knife`?
 - Listing nodes
 - View role details
 - Using Roles within a search

# UPLOADING COOKBOOKS TO CHEF SERVER 

## USING BERKSHELF
_Candidates should understand:_
 - The Berksfile & Berksfile.lock
 - `berks install` and `berks upload`
 - Where are dependant cookbooks stored locally?
 - Limitations of using knife to upload cookbooks
 - Listing cookbooks on Chef Server using knife
 - What happens if you try to upload the same version of a cookbook?
 - How can you upload the same version of a cookbook?
 - Downloading cookbooks from Chef Server
 - Bulk uploading cookbooks using knife

# USING KNIFE 

## BASIC KNIFE USAGE
_Candidates should understand:_
 - How does knife know what Chef Server to interact with?
 - How does knife authenticate with Chef Server
 - How/When would you use `knife ssh` & `knife winrm`?
 - Verifying ssl certificate authenticity using knife
 - Where can/should you run the `knife` command from?

## KNIFE CONFIGURATION
_Candidates should understand:_
 - How/where do you configure knife?
 - Common options - cookbook_path, validation_key, chef_server_url, validation_key
 - Setting the chef-client version to be installed during bootstrap
 - Setting defaults for command line options in knife’s configuration file
 - Using environment variables and sharing knife configuration file with your team
 - Managing proxies

## KNIFE PLUGINS
_Candidates should understand:_
 - Common knife plugins
 - What is 'knife ec2' plugin
 - What is 'knife windows' plugin
 - What is ‘knife block’ plugin?
 - What is ‘knife spork’ plugin?
 - Installing knife plugins

## TROUBLESHOOTING
_Candidates should understand:_
 - Troubleshooting Authentication
 - Using `knife ssl check` command
 - Using `knife ssl fetch` command
 - Using '-VV' flag
 - Setting log levels and log locations

# BOOTSTRAPPING 

## USING KNIFE
_Candidates should understand:_
 - Common ‘knife bootstrap’ options - UserName, Password, RunList, and Environment
 - Using `winrm` & `ssh`
 - Using knife plugins for bootstrap - `knife ec2 ..`, `knife bootstrap windows ...`

## BOOTSTRAP OPTIONS
_Candidates should understand:_
 - Validator' vs 'Validatorless' Bootstraps
 - Bootstrapping in FIPS mode, 
 - What are Custom Templates 

## UNATTENDED INSTALLS 
_Candidates should understand:_
 - Configuring Unattended Installs
 - What conditions must exists for unattended install to take place?

## FIRST CHEF-CLIENT RUN
_Candidates should understand:_
 - How does authentication work during the first chef-client run?
 - What is ‘ORGANIZATION-validator.pem’ file and when is it used?
 - What is the ‘first-boot.json’ file?

# POLICY FILES 

## BASIC KNOWLEDGE AND USAGE
_Candidates should understand:_
 - What are policy files, and what problems do they solve?
 - Policy file use cases?
 - What can/not be configured in a policy file?
 - Policy files and Chef Workflow

# SEARCH 

## BASIC SEARCH USAGE 
_Candidates should understand:_
 - What information is indexed and available for search?

_- client_
_- environment_
_- node_
_- role_
_- data bag_

 - Search operators and query syntax

_A search query is comprised of two parts: the key and the search pattern. A search query has the following syntax:_ `key:search_pattern`
_where key is a field name that is found in the JSON description of an indexable object on the Chef server (a role, node, client, environment, or data bag) and search_pattern defines what will be searched for, using one of the following search patterns: exact, wildcard, range, or fuzzy matching. Both key and search_pattern are case-sensitive; key has limited support for multiple character wildcard matching using an asterisk (“*”) (and as long as it is not the first character)._

```
knife search INDEX SEARCH_QUERY
```
_defaults to search for node_
```
knife search '*:*' -i
```
_is the same as_
```
$ knife search node '*:*' -i
```

 - Wildcards, range and fuzzy matching

_- To search for any node that contains the specified key, enter the following:_ `knife search node 'foo:*'`
_- To search using an inclusive range, enter the following:_ `knife search sample "id:[bar TO foo]"`
_- To use a fuzzy search pattern enter something similar to:_ `knife search client "name:boo~"`


## SEARCH USING KNIFE AND IN A RECIPE
_Candidates should understand:_
 - Knife command line search syntax

```
knife search INDEX SEARCH_QUERY
```

 - Recipe search syntax

`search(:node, "key:attribute")`


## FILTERING RESULTS 
_Candidates should understand:_
 - How do you filter on Chef Server

_Use `:filter_result` as part of a search query to filter the search output based on the pattern specified by a Hash. Only attributes in the Hash will be returned._

_The syntax for the search method that uses :filter_result is as follows:_
```
search(:index, 'query',
  :filter_result => { 'foo' => [ 'abc' ],
                      'bar' => [ '123' ],
                      'baz' => [ 'sea', 'power' ]
                    }
      ).each do |result|
  puts result['foo']
  puts result['bar']
  puts result['baz']
end
```
_where:_
_- `:index` is of name of the index on the Chef server against which the search query will run: `:client`, `:data_bag_name`, `:environment`, `:node`, and `:role`_
_- 'query' is a valid search query against an object on the Chef server_
_- `:filter_result` defines a Hash of values to be returned_

_For example:_

```
search(:node, 'role:web',
  :filter_result => { 'name' => [ 'name' ],
                      'ip' => [ 'ipaddress' ],
                      'kernel_version' => [ 'kernel', 'version' ]
                    }
      ).each do |result|
  puts result['name']
  puts result['ip']
  puts result['kernel_version']
end
```

 - Selecting attributes to be returned

_Use :filter\_result as part of a search query to filter the search output based on the pattern specified by a Hash. Only attributes in the Hash will be returned._

# CHEF SOLO 

## WHAT CHEF SOLO IS
_Candidates should understand:_
 - Advantages & disadvantages of Chef-solo vs Chef Server

_chef-solo is a command that executes chef-client in a way that does not require the Chef server in order to converge cookbooks. chef-solo uses chef-client’s Chef local mode, and does not support the following functionality present in chef-client / server configurations:_

_- Centralized distribution of cookbooks_
_- A centralized API that interacts with and integrates infrastructure components_
_- Authentication or authorization_

 - Chef-solo executable and options
```
chef-solo OPTION VALUE OPTION VALUE ...
```
_lots of options found here https://docs.chef.io/chef\_solo.html_

 - Cookbooks, nodes and attributes

_chef-solo supports two locations from which cookbooks can be run:
_- A local directory._
_- A URL at which a tar.gz archive is located._

_Unlike chef-client, where the node object is stored on the Chef server, chef-solo stores its node objects as JSON files on local disk. By default, chef-solo stores these files in a nodes folder in the same directory as your cookbooks directory. You can control the location of this directory via the node_path value in your configuration file._

_chef-solo does not interact with the Chef server. Consequently, node-specific attributes must be located in a JSON file on the target system, a remote location (such as Amazon Simple Storage Service (S3)), or a web server on the local network._

 - Using Data Bags, Roles & Environments

_chef-solo will look for data bags in /var/chef/data_bags, but this location can be modified by changing the setting in solo.rb._ `data_bag_path '/var/chef-solo/data_bags'`

_chef-solo will look for roles in /var/chef/roles, but this location can be modified by changing the setting for role_path in solo.rb._ `role_path '/var/chef-solo/roles'`

_chef-solo will look for environments in /var/chef/environments, but this location can be modified by changing the setting for environment_path in solo.rb._ `environment_path '/var/chef-solo/environments'`

 - Chef-solo run intervals

```
-i SECONDS, --interval SECONDS
```
_The frequency (in seconds) at which the chef-client runs. When the chef-client is run at intervals, `--splay` and `--interval` values are applied before the chef-client run._

 - Retrieving cookbooks from remote locations

```
-r RECIPE_URL, --recipe-url RECIPE_URL
```
_The URL location from which a remote cookbook tar.gz is to be downloaded._

 - Chef-solo and node object

# DATA BAGS 

## WHAT IS A DATA_BAG
_Candidates should understand:_
 - When might you use a data_bag?

_A data bag is a global variable that is stored as JSON data and is accessible from a Chef server. A data bag is indexed for searching and can be loaded by a recipe or accessed during a search. It is used when different clients need to access the same information._

 - Indexing data_bags

_Any search for a data bag (or a data bag item) must specify the name of the data bag and then provide the search query string that will be used during the search. For example, to use knife to search within a data bag named “admin\_data” across all items, except for the “admin\_users” item, enter the following:_
```
knife search admin_data "(NOT id:admin_users)"
```
_Or, to include the same search query in a recipe, use a code block similar to:_
```
search(:admin_data, "NOT id:admin_users")
```
_Data bags can be accessed through the search indexes. Use this approach when more than one data bag item is required or when the contents of a data bag are looped through. The search indexes will bulk-load all of the data bag items, which will result in a lower overhead than if each data bag item were loaded by name._

_To load the secret from a file:_
```
data_bag_item('bag', 'item', IO.read('secret_file'))
```
_To load a single data bag item named admins:_

```
data_bag('admins')
```
_The contents of a data bag item named justin:_
```
data_bag_item('admins', 'justin')
```

 - What is a data_bag?

_A data bag is a global variable that is stored as JSON data and is accessible from a Chef server. A data bag is indexed for searching and can be loaded by a recipe or accessed during a search._

 - Data_bag and Chef-solo

_chef-solo can load data from a data bag as long as the contents of that data bag are accessible from a directory structure that exists on the same machine as chef-solo. The location of this directory is configurable using the data_bag_path option in the solo.rb file. The name of each sub-directory corresponds to a data bag and each JSON file within a sub-directory corresponds to a data bag item. Search is not available in recipes when they are run with chef-solo; use the data_bag() and data_bag_item() functions to access data bags and data bag items._

## DATA_BAG ENCRYPTION
_Candidates should understand:_
 - How do you encrypt a data_bag

_A data bag item is encrypted using a knife command similar to:_
```
knife data bag create passwords mysql --secret-file /tmp/my_data_bag_key
```
_where “passwords” is the name of the data bag, “mysql” is the name of the data bag item, and “/tmp/my_data_bag_key” is the path to the location in which the file that contains the secret-key is located. knife will ask for user credentials before the encrypted data bag item is saved._

 - What is Chef Vault

_chef-vault is a RubyGems package that is included in the Chef development kit. chef-vault allows the encryption of a data bag item by using the public keys of a list of nodes, allowing only those nodes to decrypt the encrypted values. chef-vault uses the knife vault subcommand._

## USING DATA_BAGS
_Candidates should understand:_
 - How do you create a data_bag?

_knife can be used to create data bags and data bag items when the knife data bag subcommand is run with the create argument. For example:_

```
knife data bag create DATA_BAG_NAME (DATA_BAG_ITEM)
```
_knife can be used to update data bag items using the from file argument:_
```
knife data bag from file BAG_NAME ITEM_NAME.json
```
_As long as a file is in the correct directory structure, knife will be able to find the data bag and data bag item with only the name of the data bag and data bag item. For example:_

```knife data bag from file BAG_NAME ITEM_NAME.json
```

 - How can you edit a data_bag

_Continuing the example above, if you are in the “admins” directory and make changes to the file charlie.json, then to upload that change to the Chef server use the following command:_
```
knife data bag from file admins charlie.json
```