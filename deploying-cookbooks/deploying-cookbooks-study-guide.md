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
 - Configuring chef-client 
 - COMPILE VS EXECUTE

_Candidates should understand:_
 - What happens during the 'compile' phase?
 - What happens during the 'execute' phase?
 - What happens when you place some Ruby at the start of a recipe?
 - What happens when you place some Ruby at the end of a recipe?
 - When are attributes evaluated?
 - What happens during a 'node.save' operation?"
 - RUN_STATUS

_Candidates should understand:_
 - How can you tap into the chef-client run?
 - What is ‘run_status’?
 - What is ‘run_state’?
 - What is the ‘resource collection’?
 - AUTHENTICATION

_Candidates should understand:_
 - Deploying Cookbooks Page 2 v1.0
 - How does the chef-client authenticate with the Chef Server?
 - Authentication and using NTP
 - CHEF COMPILE PHASE


_Candidates should understand:_
 - Do all cookbooks always get downloaded?
 - What about dependencies, and their dependencies?
 - What order do the following get loaded - libraries, attributes, resources/providers, 
 - definitaions, recipes?
 - CONVERGENCE

_Candidates should understand:_
 - What happens during the execute phase of the chef-client run?
 - The test/repair model
 - When do notifications get invoked?
 - What happens if there are multiple start notifications for a particular resource?
 - When is node data sent to the Chef Server?
 - WHY-RUN

_Candidates should understand:_
 - What is the purpose of ‘why-run’
 - How do you invoke a ‘why-run’
 - What are limitations of doing a why run?
 - ENVIRONMENTS 
 - WHAT IS AN ENVIRONMENT/USE CASES

_Candidates should understand:_
 - What is the purpose of an Environments?
 - What is the '_default' environment?
 - What information can be specified in an Environment?
 - What happens if you do not specify an Environment?
 - Creating environments in Ruby
 - Creating environments in JSON
 - Using environments within a search
 - ATTRIBUTE PRECEDENCE AND COOKBOOK CONSTRAINTS

_Candidates should understand:_
 - What attribute precedence levels are available for Environments
 - Overriding Role attributes
 - Syntax for setting cookbook constraints.
 - How would you allow only patch updates to a cookbook within an environment?
 - Deploying Cookbooks Page 3 v1.0
 - SETTING AND VIEWING ENVIRONMENTS

_Candidates should understand:_
 - How can you list Environments?
 - How can you move a node to a specific Environment?
 - Using `knife exec` to bulk change Environments.
 - Using 'chef_environment' global variable in recipes
 - Environment specific knife plugins, e.g. `knife flip`
 - Bootstrapping a node into a particular Environment
 - ROLES 
 - USING ROLES

_Candidates should understand:_
 - What is the purpose of a Role?
 - Creating Roles
 - Role Ruby & JSON DSL formats
 - Pros and Cons of Roles 
 - The Role cookbook pattern
 - Creating Role cookbooks
 - Using Roles within a search
 - SETTING ATTRIBUTES AND ATTRIBUTE PRECEDENCE

_Candidates should understand:_
 - What attribute precedence levels are available for Roles
 - Setting attribute precedence
 - BASE ROLE & NESTED ROLES

_Candidates should understand:_
 - What are nested roles?
 - Whats the purpose of a ‘base’ role?
 - USING KNIFE

_Candidates should understand:_
 - The `knife role` command
 - How would you set the run_list for a node using `knife`?
 - Listing nodes
 - View role details
 - Using Roles within a search
 - Deploying Cookbooks Page 4 v1.0
 - UPLOADING COOKBOOKS TO CHEF SERVER 
 - USING BERKSHELF

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
 - USING KNIFE 
 - BASIC KNIFE USAGE

_Candidates should understand:_
 - How does knife know what Chef Server to interact with?
 - How does knife authenticate with Chef Server
 - How/When would you use `knife ssh` & `knife winrm`?
 - Verifying ssl certificate authenticity using knife
 - Where can/should you run the `knife` command from?
 - KNIFE CONFIGURATION

_Candidates should understand:_
 - How/where do you configure knife?
 - Common options - cookbook_path, validation_key, chef_server_url, validation_key
 - Setting the chef-client version to be installed during bootstrap
 - Setting defaults for command line options in knife’s configuration file
 - Using environment variables and sharing knife configuration file with your team
 - Managing proxies
 - KNIFE PLUGINS

_Candidates should understand:_
 - Common knife plugins
 - What is 'knife ec2' plugin
 - What is 'knife windows' plugin
 - What is ‘knife block’ plugin?
 - What is ‘knife spork’ plugin?
 - Installing knife plugins
 - Deploying Cookbooks Page 5 v1.0
 - TROUBLESHOOTING

_Candidates should understand:_
 - Troubleshooting Authentication
 - Using `knife ssl check` command
 - Using `knife ssl fetch` command
 - Using '-VV' flag
 - Setting log levels and log locations
 - BOOTSTRAPPING 
 - USING KNIFE

_Candidates should understand:_
 - Common ‘knife bootstrap’ options - UserName, Password, RunList, and Environment
 - Using `winrm` & `ssh`
 - Using knife plugins for bootstrap - `knife ec2 ..`, `knife bootstrap windows ...`
 - BOOTSTRAP OPTIONS

_Candidates should understand:_
 - Validator' vs 'Validatorless' Bootstraps
 - Bootstrapping in FIPS mode, 
 - What are Custom Templates 
 - UNATTENDED INSTALLS 

_Candidates should understand:_
 - Configuring Unattended Installs
 - What conditions must exists for unattended install to take place?
 - FIRST CHEF-CLIENT RUN

_Candidates should understand:_
 - How does authentication work during the first chef-client run?
 - What is ‘ORGANIZATION-validator.pem’ file and when is it used?
 - What is the ‘first-boot.json’ file?
 - POLICY FILES 
 - BASIC KNOWLEDGE AND USAGE

_Candidates should understand:_
 - What are policy files, and what problems do they solve?
 - Policy file use cases?
 - What can/not be configured in a policy file?
 - Policy files and Chef Workflow
 - Deploying Cookbooks Page 6 v1.0
 - SEARCH 
 - BASIC SEARCH USAGE 

_Candidates should understand:_
 - What information is indexed and available for search?
 - Search operators and query syntax
 - Wildcards, range and fuzzy matching
 - SEARCH USING KNIFE AND IN A RECIPE

_Candidates should understand:_
 - Knife command line search syntax
 - Recipe search syntax
 - FILTERING RESULTS 

_Candidates should understand:_
 - How do you filter on Chef Server
 - Selecting attributes to be returned
 - CHEF SOLO 
 - WHAT CHEF SOLO IS

_Candidates should understand:_
 - Advantages & disadvantages of Chef-solo vs Chef Server
 - Chef-solo executable and options
 - Cookbooks, nodes and attributes
 - Using Data Bags, Roles & Environments
 - Chef-solo run intervals
 - Retreiving cookbooks from remote locations
 - Chef-solo and node object
 - DATA BAGS 
 - WHAT IS A DATA_BAG

_Candidates should understand:_
 - When might you use a data_bag?
 - Indexing data_bags
 - What is a data_bag?
 - Data_bag and Chef-solo
 - DATA_BAG ENCRYPTION

_Candidates should understand:_
 - Deploying Cookbooks Page 7 v1.0
 - How do you encrypt a data_bag
 - What is Chef Vault
 - USING DATA_BAGS

_Candidates should understand:_
 - How do you create a data_bag?
 - How can you edit a data_bag