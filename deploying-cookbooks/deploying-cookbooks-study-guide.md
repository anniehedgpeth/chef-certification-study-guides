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
_Candidates should understand:_
 - What happens during the 'compile' phase?
 - What happens during the 'execute' phase?

_The chef-client processes recipes in two phases:_

_First, each resource in the node object is identified and a resource collection is built. All recipes are loaded in a specific order, and then the actions specified within each of them are identified. This is also referred to as the “compile phase”._
_Next, the chef-client configures the system based on the order of the resources in the resource collection. Each resource is mapped to a provider, which then examines the node and performs the necessary steps to complete the action. This is also referred to as the “execution phase”._
_Typically, actions are processed during the execution phase of the chef-client run. However, sometimes it is necessary to run an action during the compile phase. For example, a resource can be configured to install a package during the compile phase to ensure that application is available to other resources during the execution phase._

> Note: Use the chef_gem resource to install gems that are needed by the chef-client during the execution phase.

 - What happens when you place some Ruby at the start of a recipe?
 - What happens when you place some Ruby at the end of a recipe?
 - When are attributes evaluated?
 - What happens during a 'node.save' operation?"

## RUN_STATUS
_Candidates should understand:_
 - How can you tap into the chef-client run?
 - What is ‘run_status’?
 - What is ‘run_state’?
 - What is the ‘resource collection’?

## AUTHENTICATION
_Candidates should understand:_
 - Deploying Cookbooks Page 2 v1.0
 - How does the chef-client authenticate with the Chef Server?
 - Authentication and using NTP

## CHEF COMPILE PHASE
_Candidates should understand:_
 - Do all cookbooks always get downloaded?
 - What about dependencies, and their dependencies?
 - What order do the following get loaded - libraries, attributes, resources/providers, 
 - definitions, recipes?

## CONVERGENCE
_Candidates should understand:_
 - What happens during the execute phase of the chef-client run?
 - The test/repair model
 - When do notifications get invoked?
 - What happens if there are multiple start notifications for a particular resource?
 - When is node data sent to the Chef Server?

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
 - Deploying Cookbooks Page 3 v1.0

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
 - Deploying Cookbooks Page 4 v1.0

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
 - Deploying Cookbooks Page 5 v1.0

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
 - Deploying Cookbooks Page 6 v1.0

# SEARCH 

## BASIC SEARCH USAGE 
_Candidates should understand:_
 - What information is indexed and available for search?
 - Search operators and query syntax
 - Wildcards, range and fuzzy matching

## SEARCH USING KNIFE AND IN A RECIPE
_Candidates should understand:_
 - Knife command line search syntax
 - Recipe search syntax

## FILTERING RESULTS 
_Candidates should understand:_
 - How do you filter on Chef Server
 - Selecting attributes to be returned

# CHEF SOLO 

## WHAT CHEF SOLO IS
_Candidates should understand:_
 - Advantages & disadvantages of Chef-solo vs Chef Server
 - Chef-solo executable and options
 - Cookbooks, nodes and attributes
 - Using Data Bags, Roles & Environments
 - Chef-solo run intervals
 - Retreiving cookbooks from remote locations
 - Chef-solo and node object

# DATA BAGS 

## WHAT IS A DATA_BAG
_Candidates should understand:_
 - When might you use a data_bag?
 - Indexing data_bags
 - What is a data_bag?
 - Data_bag and Chef-solo

## DATA_BAG ENCRYPTION
_Candidates should understand:_
 - Deploying Cookbooks Page 7 v1.0
 - How do you encrypt a data_bag
 - What is Chef Vault

## USING DATA_BAGS
_Candidates should understand:_
 - How do you create a data_bag?
 - How can you edit a data_bag