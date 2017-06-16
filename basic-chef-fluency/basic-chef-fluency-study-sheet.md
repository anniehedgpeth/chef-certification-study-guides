# Study Sheet for Basic	Chef Fluency

> The topics for this guide are taken directly from [training.chef.io](https://training.chef.io). I copied the bulleted lists, and I filled in the study information from either my own understanding or from docs.chef.io. 

# [BASIC CHEF FLUENCY BADGE TOPICS](https://training.chef.io/static/Basic_Chef_Fluency_Badge_Scope.pdf)
https://training.chef.io/static/Basic_Chef_Fluency_Badge_Scope.pdf 

**The	Basic	Chef Fluency badge is	awarded	when someone proves	that they	understand the core	elements that	underpin Chef. Candidates	must show:**

- An understanding of basic Chef terminology.
- An understanding of Chef product offerings.
- An understanding of Chef's design philosophy.
- An understanding of Chef's approach to workflow and compliance.
- An understanding of basic Chef coding skills.

Here is a detailed breakdown of each area.

# CHEF BASIC TERMINOLOGY	
## RESOURCES
*Candidates should understand:*

### Idempotency/convergence	- test & repair model
When a node is converged, chef-client first tests to see if the node is in the same state as defined by the recipe. If it is not, then it repairs the node in accordance to the recipe.

### Common resources and their actions
*Default actions*<br/>
- The `:nothing` action
  - This simply means that this resource will do nothing to the node unless told otherwise.

- The `supports` directive
  - The `supports` directive specifies the platform that the cookbook supports. This is specified within the cookbook's `metadata.rb`. 

- The `not_if` and `only_if` directives
  - You may specify that a resource takes action only if or not if another requirement is met or not met.

### Resource extensibility

## RECIPES
*Candidates	should understand:*

### What a recipe is						
- A recipe is an ordered script within a cookbook that can be called within a chef-client run.

### Importance of the resource order
- The resources of a recipe are called in the order in which they are written because some require that others exist or not before they can be called.

### How to use `include_recipe`
- To include another recipe within a recipe, one must first make the cookbook dependent upon that recipe within the metadata of that cookbook. Then you may use the `include_recipe` resource within the recipe to actually execute that dependent recipe.

### What happens if a recipe is included multiple times in a run_list
- The node will be idempotent, so as the recipe runs a subsequent time, it will again test and repair.

### The `notifies` and `subscribes` directives
- A resource can notify another resource of its actions in order for the secondary resource to take action based upon the primary resource's actions. Conversely, a resource can subscribe to another resource to listen for its actions and take action itself based on the actions of the resource to which is it subscribed.

# COOKBOOKS
*Candidates should understand:*

### Cookbook contents
- A cookbooks' primary contents include recipes, attributes, resources, metadata, `.kitchen.yml`, data bags, roles, environments.

### Naming conventions
- Underscores should be used instead of hyphens.

### Cookbook dependencies
- You can declare the dependencies of your cookbook on other cookbooks by including those cookbooks in the `metadata.rb` by using `depends '<cookbookname>', 'versionoptional'`. If you're using dependencies that are not stored in source control or the Supermarket, you may use Berks to upload those dependencies to the Chef server from your workstation.

### The default recipe
- If there is only one recipe in the cookbook, then this will be called the default recipe. The default recipe is called when the cookbook is listed without a recipe name in a run list.

# CHEF SERVER
*Candidates should understand:*

### How the Chef server acts as an artifact repository
- The Chef server can store a number of different artifacts for use by the chef-client. These include but are not limited to: roles, environments, data bags with data bag items stored inside, attributes, cookbooks, policies, clients, organizations, node attributes / OHAI data.

### How the Chef server acts as an index of node data
- The Chef server receives OHAI data, which contains all of the node data from the chef-client after it is run.

### Chef solo vs Chef server
- When using Chef solo, all of the artifacts are stored on the workstation in which you are working, whereas when you are using the Chef server, those artifacts are stored on the Chef server.

### [Chef server's distributed architecture](https://docs.chef.io/install_server_ha.html)
- If Chef server being a single point of failure being a huge risk for you, then you can run Chef server in high availability mode, which means it has redundancy. 

### Scalability
- If the chef-client is on the node being configured, then it scales more easily because a server doesn't have to do all the work. Scaling with High Availabililty (HA) is also an option.

## SEARCH
*Candidates should understand:*

### What search is
- `knife search` commands can be use to search the Chef server.

### How to search for node information
- `knife search node "<index>:<search_query>"` may be invoked or you may search in the Chef server UI (nodes > attributes).

### [What and how many search indexes Chef server maintains](https://docs.chef.io/chef_search.html)
- Node data is indexed on the Chef server. That data may be accessed through a search query in any of the following ways:
 1) within the recipe
 2) by using the `knife search` command
 3) the search method in the Recipe DSL
 4) the search box in the Chef management console
 5) by using the `/search` or `/search/INDEX` endpoints in the Chef server API

### What a databag is
- A databag is a directory of data that is stored on the Chef server.

### How to use search for dynamic orchestration
- You would use the `knife search` command to get a list of nodes on which you need to perform actions.

### How to invoke a search from the command line
- `knife search node "<index>:<search_query>"`

## CHEF CLIENT
*Candidates should understand:*

### What the Chef client is
- The Chef client is an agent installed on the nodes being configured. 

### The function of Chef client vs the function of Chef server
- While the client is installed on the nodes being configured, the server stores the necessary configuration information for the client to access.

### What 'why-run' is
- The `--why-run` command is similar to Terraform's `terraform plan` command. It tells you what would happen and change if you were to converge. It does run the executable, but it doesn't actually modify the system.

### How to use '--local-mode'
- `chef_zero` would be the provisioner, and you'd use local mode if you weren't using a proper Chef server. All of the data would be accessed from 

### How the Chef client and the Chef server communicate
- The node on which Chef client is installed has a client.rb file which contains a link and credentials to access the Chef server. The Chef server never accesses the Chef client; it merely holds the data that the client will need to access. The Chef client communicates to the Chef server over https (port 443) or http (port 80).

### The Chef client configuration
- The client is configures the node that it is on based on the run-list it is given during the bootstrap or knife commands.

## NODES
*Candidates should understand:*

### What a node is
- A node is the machine that is being configured by the Chef client.

### What a node object is
- A node object is data that is given to the Chef server to store by the Chef client after `chef-client` is run which contains OHAI data as well as attributes.

### How a node object is stored on Chef server
- A node object is data that is given to the Chef server to store by the Chef client after `chef-client` is run which contains OHAI data as well as attributes.

### How to manage nodes
- Nodes may be managed through the Chef server UI or through knife commands.

### How to query nodes
- Nodes may be searched through using the `knife search node` command. For example `knife search node "platform:ubuntu"`.

### How to name nodes
- Nodes are named either during a bootstrap or by changing the `client.rb` file on the node.

## RUN LIST
*Candidates should understand:*

### What a run_list is
- A run_list is added to a node either during a bootstrap or by changing the `client.rb` file on the node. It contains all of the cookbooks, recipes, and roles for the node.

### What nested run_lists are
- A nested run_list simply means that when a role has a run-list and you add that role to a run-list, it becomes nested. 
  - For example: `security` role has a run-list that includes `cloudpassage::default, hardening::centos` and then your node has a run-list that includes `role[security]` in it

### Where a run_list is stored
- A run_list is stored both on the Chef server and in the node's `client.rb`.

### What does a run_list consist of
- A run_list consists of all of the recipes and cookbooks to be run on the node as well as the roles assigned to it, which contain their own recipes and cookbooks.

### How to look up run_lists
- You may find a node's run_list in 3 ways:
  1) Look in the UI under Nodes > Details > Run List.
  2) Run `knife node show <nodename>`
  3) In the node, look at the `client.rb` file.

### How to set and change run_lists
- You can set and change run_lists in much the same way that you can view them.
  1) Look in the UI under Nodes > Details > Run List > Edit.
  2) After you've insured that your desired cookbooks are uploaded to the Chef server, run `knife node run_list add <nodename> 'recipe[<cookbookname>::<recipe>]'` OR `knife node run_list add chefkata7 'role[security]'`
  3) In the node, look at the `client.rb` file (not best practice).

## ROLES
*Candidates should understand:*

### What roles are
- A role sets the patterns and processes for a node. Each node may have any amount of roles or none. A role can be thought of in two different ways: 
   1) the role of the machine
   2) the role of the team/person setting the run lists (not best practice)

### How a role ensures code consistency across nodes
- A role ensures code consistency by giving that node all of the same patterns and processes as well as attributes.

### Where roles can be stored
- Roles are stored in the chefrepo sibling to cookbooks on your workstation or on the node if you're running it in local mode. Otherwise, they're stored on the Chef server.

### How roles are defined
- Roles are defined in a `.json` or `.rb` file within a `roles` directory sibling to the `cookbooks` directory in a `chefrepo` which is uploaded to the Chef server. Or you may create a role within the UI of the Chef server: Policy > Roles > Create. 

### What the components of a role are
- The main components Roles consist of a run list and attributes (both default and override).

### Roles vs recipes vs run_lists
- A role, recipe, and run_list can each call various recipes in different ways. 
   1) A role will be assigned to a node and may or may not have its own run_list that will be included for that node.
   2) A recipe can use the `include_recipe` resource in order to run a recipe that is included in that cookbook's dependencies within the `metatdata.rb` file. 
   3) A run_list is assigned to a node using the `bootstrap` or `knife` command. It may contain both recipes and roles.

### How to name roles
- The name of the role is defined by the name of the `.json` or `.rb` file that you create in the `roles` directory as well as the metadata within that file. This may also be done within the UI of the Chef server. The requirements for the name are that it must be:
   1) unique to its organization
   2) made of letters (upper and/or lower-case), numbers, underscores, and/or hyphens (no spaces allowed). 

### How to apply roles to nodes
- Roles are assigned to nodes through the bootstrap, knife, or UI, and they are then applied when `chef-client` is run. 

## How to edit roles ENVIRONMENTS
*Candidates should understand:*

### The purpose of environments
- Environments are assigned to nodes to determine which phase of the release cycle that node represents. This include but are not limited to development, user acceptance testing, and production. 

### How to use environments to manage cookbook release cycles
- An environment is assigned to a node in order to give it the appropriate version of the cookbooks in the run_lists. The earlier in the release cycle that the environment assigned to the role is, the later versions of the recipes it will be assigned to.

### How to use environments to constrain cookbooks
- Environments are assigned to nodes, and then that node data is returned to the Chef server. You can then use that data within a recipe to constrain attributes to particular environments.

### How to put nodes into an environment
- Environments are assigned to nodes through:
  1) the UI under Nodes > Attributes > Edit.
  2) bootstrap and knife commands
  3) by editing the `client.rb` file
- You may also change a node's environment with a `knife exec` command.

## INFRASTRUCTURE AS CODE
*Candidates should understand:*

### What the advantages are of defining infrastructure as code
- If your infrastructure is code, then you know your the state of your infrastructure just by looking at the code. You can then easily alter your configuration and use version control to promote changes easily through development.

### The reasons for defining infrastructure as code
- Scalability
- version control
- dynamic infrastructure
- simplicity and flexibility for managing and provisioning infrastructure

### The difference between rolling back and rolling forward
- Rolling back would be to undo the configuration changes made to your infrastructure. This is not best practice with Chef, as you're only allowed to roll back due to the nature of idempotency. You would need to first change your recipe to intentionally undo the changes to the infrastructure that you wanted to "roll back". Then you'd redefine the desired state in your recipe and instill those changes by converging again.

## DESIRED STATE CONFIGURATION
*Candidates should understand:*

### The imperative vs the declarative approach to configuration management
- An imperative approach to configuration management would be something like running a shell or Powershell script that consecutively executes commands on a node. Chef, however, uses a declarative approach which would declare the desired state of configuration within the Chef server and perform that configuration management through the Chef client installed on the node.

### The push vs the pull approach
- A push approach would be if the node that was being configured was idle while the server pushed a policy of configuration desired state to the node. This would mean that there is nothing additional being installed on the node being configured, only what is in the policy. A pull approach is what Chef employs, which means that the Chef server is idle while the client that is installed on the node being configured pulls the necessary data from the Chef server that it needs to complete its configuration given to it through a bootstrapping process.

### What Windows DSC is
- Windows Desired State Configuration is Windows' configuration management tool based on Powershell which employs declarative programming.

### What happens if you remove a resource from a recipe
- If you remove a resource from a recipe that has already run on the node, then nothing will happen. Whatever that resource did the first time it ran will still be in effect unless you intentionally undo its effects.

## SUPERMARKET
*Candidates should	understand:*

### The Supermarket value proposition
- The Supermarket is a storehouse full of FREE cookbooks available to use and maintained by Chef. They are easily depended upon in your own cookbooks.

### What you can store in Supermarket
- The Supermarket can store cookbook and other tools such as InSpec profiles.

### What a private Supermarket is
- A company may have their own private supermarket for use internally. These must be maintained by them and can only be accessed by them.

### When to use a private Supermarket
- You would want to use a private Supermarket when you wanted to produce and promote your own cookbooks.

### If Supermarket is a free or a premium feature
- The Supermarket is FREE for anyone to use.

### If the items in Supermarket are free or cost money
- Everything within the Supermarket is FREE to use for anyone.

## CHEF DK
*Candidates	should	understand:*

### The	Chef DK	value	proposition						
- When the Chef DK is installed on your workstation, you're able to develop with Chef and therefor able to completely configure your infrastructure. 

### Specific features of test-driven development (TDD)
- Audit > Remediate > Certify - TDD allows you to develop based on red/green/refactor. You first test for the desired state, and when it fails you write the desired state in your recipe. You then test again, and it passes.

### Tools packaged in Chef DK
- Chef-client and OHAI
- Chef CLI
- InSpec / ChefSpec / Cookstyle / Foodcritic
- Chef provisioning
- Knife / Berks / Spork
- Test Kitchen

## TEST KITCHEN
*Candidates	should understand:*

### The Test Kitchen value proposition
- Test Kitchen is at the heart of TDD within Chef. Being able to test your configuration before changing the state of your machine is invaluable because you can configure the state of a throw-away machine without having to change your actual infrastructure.

### What TDD is
- Test Kitchen is at the heart of TDD within Chef. Being able to test your configuration before changing the state of your machine is invaluable because you can configure the state of a throw-away machine without having to change your actual infrastructure.

### The platforms supported by Test Kitchen
- Linux and Windows

### How to include Test Kitchen in a pipeline
- You can add `kitchen test` into your pipeline, and it will run through the entire Kitchen cycle (create, converge, verify, destroy). If it does not pass, then you can halt your build or promote if it passes.

### Basic `kitchen` commands
- `kitchen create`
- `kitchen converge`
- `kitchen login`
- `kitchen verify`
- `kitchen destroy`
- `kitchen test`
 
### Basic `kitchen` configuration
- Driver- What is creating your VM?
- Provisioner- What is running Chef?
- Verifier- What is running the tests? (probably InSpec)
- Transport- What are you using to remote to a machine?
- Suites- What are the machines are you making?
- Platform- What OS are you using?

# DESCRIBING WHAT CHEF IS

## PRODUCTS AND FEATURES
*Candidates should understand:*

### [The Chef Automate value proposition](https://www.chef.io/automate/)
- "Effective cross-team collaboration" Chef Automate tracks and tests dependencies between projects and teams. Deploy safely, even in a multi-team, multi-project environment where there are complex inter-dependencies.
   1) Full oversight
   2) Change management
   3) Governance
   4) Built-in compliance  

### The Chef Automate features
### What the workflow feature is and how it affects productivity
- Workflow is similar to Jenkins or Team City, as it defines and automates the CI/CD pipeline. It affects productivity by promoting the next build in the previous one had passed. You're able to release code faster when the automation process is in charge of promotion. The safeguard of promoting only the builds that pass validation contributes to productivity, as well.

### What the compliance feature is and how it affects workflow
- In the same way that workflow adds safeguards which contribute to productivity
 
### What the visibility feature is and how it affects workflow
- It provides insight on how changes affect the state of Chef on the nodes.
 
### How a private Supermarket fits into a workflow
- You create your own server, then you reference your private supermarket in your Berksfile so that Chef server grabs the cookbooks from there.
 
### The Chef Automate open source components
- The open source components of Automate include:
   1) InSpec
   2) Chef
   3) Habitat
 
### What Visibility is
- Visibility is a tool within Chef Automate that allows you to see the state of infrastructure across your entire organization. 
 
### What Habitat is
- Habitat is a tool within Chef Automate for the managing and packaging of applications.
 
### What InSpec is
- InSpec is a framework with which to automate security and compliance tests. 
 
### What Chef Compliance is
- Compliance is the tool created on top of the InSpec framework which integrates within Chef Automate.
 

## END-TO-END WORKFLOW
*Candidates	should	understand:*

### How all Chef products, features, and technologies fit together
- Audit (InSpec/Compliance) > Remediate (Chef/Workflow/Habitat) > Certify (Visibility)
 
### The workflow scope
- Workflow defines and automates the CI/CD pipeline.
 
### The compliance scope
- Compliance defines and automates compliance validation. 
 
### The Chef Automate scope
- Audit (InSpec/Compliance) > Remediate (Chef/Workflow/Habitat) > Certify (Visibility)

### How Chef Automate enhances DevOps behaviors
- Audit (InSpec/Compliance) > Remediate (Chef/Workflow/Habitat) > Certify (Visibility); TDD
- "Effective cross-team collaboration" Chef Automate tracks and tests dependencies between projects and teams. Deploy safely, even in a multi-team, multi-project environment where there are complex inter-dependencies.
  1) Full oversight
  2) Change management
  3) Governance
  4) Built-in compliance  
 
### The aspects of Chef that are relevant to security and compliance teams
- Cookbooks for hardening and patch management
- InSpec and Compliance within Workflow for promotion of safe cookbooks
- Visibility for validation of compliance
 
### The aspects of Chef that are relevant to development teams
- Habitat for application packaging
- Test Kitchen for TDD
- Workflow for promotion of code through the pipeline
 
### The aspects of Chef that are relevant to operations teams
- Compliance for health checks
- Chef for infrastructure as code
- Visibility for state of affairs
 
### The aspects of Chef that are relevant to change	advisory boards
- Visibility for state of affairs to assess risk of change and to verify the reliability of a version from a previous environment
- Workflow for approval process
 
### How Chef enforces consistency across infrastructure
- Chef enforces consistency across infrastructure by having roles and environments, and Workflow helps to ensure that changes are consistently applied.  

## DESIGN PHILOSOPHY
## CHEF IS WRITTEN IN RUBY
*Candidates should understand:*

### How Chef uses a Ruby-based DSL
- Chef is a thin domain-specific-language, meaning that it was built on top of Ruby. This means that you're able to easily customize with Ruby.

### How to use raw Ruby to extend Chef
- If you'd like to customize your cookbook, then you can use raw Ruby anywhere in the cookbook.

### What a library is
- Ruby code can be stored in a cookbook's `libraries` directory and can be implemented / called anywhere in the cookbook's recipes or custom resources.

## EXPLICIT ACTIONS
*Candidates should understand:*

### How Chef uses explicit actions and only does what you tell it to
- You must state explicitly what you want Chef to do in your configuration. 

### Actions for common resources such as the :nothing action
- Common actions include :nothing, :create, :sync, :delete, :run, :start, :restart, :stop, :status, etc.

### What it means to roll back infrastructure
- Rolling back would be to undo the configuration changes made to your infrastructure. This is not best practice with Chef, as you're only allowed to roll back due to the nature of idempotency. You would need to first change your recipe to intentionally undo the changes to the infrastructure that you wanted to "roll back". Then you'd redefine the desired state in your recipe and instill those changes by converging again.

### What happens if you reverse the order of resources in a recipe
- Resources are executed in the order in which they are written. So if you were to reverse them, Chef client would test and repair them in that new order.

### [If Chef can automatically detect what patches should be applied to a system](https://blog.chef.io/2017/01/05/patch-management-system-using-chef-automate/)
- Yes and no. 
   1) Identify which patches are required
   2) Apply and evaluate the patches in a testing environment
   3) Distribute the patches to the rest of the fleet
   4) Confirm the patches have been applied successfully with InSpec

## PUSH VS. PULL
*Candidates should understand:*

### The difference between push and pull models
- A push approach would be if the node that was being configured was passive while the server pushed a policy of configuration desired state to the node. This could mean that there is nothing additional being installed on the node being configured, only what is in the policy. A pull approach is what Chef employs, which means that the Chef server is passive while the client that is installed on the node being configured pulls the necessary data from the Chef server that it needs to complete its configuration given to it through a bootstrapping process.

### The benefits of a pull model
- The major benefit of a pull model is scalability. The server that houses the data needed for configuration is not strained trying to configure all of the nodes. Instead the client installed on the node does the configuration itself.

### When a push model is appropriate
- When you need to run a job independently of a `chef-client` run, a push job may be appropriate. This will allow you to send a job to a node independently of the client. A push model may also be required when orchestrating changes in an order amongst different types of servers.

### What firewall rules need to be enabled for Chef client
- The Chef client communicates to the Chef server over https (port 443) or http (port 80).

### The Chef client converge intervals and how to invoke immediate updates
- Converge intervals are configured by the Chef client cookbook that you would include in your run list. Push jobs would be employed to invoke immediate updates.

## RECOMMENDED WORKFLOWS
*Candidates should understand:*

### What wrapper cookbooks are
- A wrapper cookbook is the main cookbook that you write that you include dependent cookbooks inside that don't contain all of the functionality that you need. The wrapper would extend the dependent cookbook to include the additional functionality that you need.

### How to use source control, e.g. GitHub
- Storing your repos and cookbooks in source control, such as GitHub, allows you to collaborate with others on development as well as branch, fork, and promote.

### How to use the TDD approach
- TDD allows you to develop based on red/green/refactor. You first test for the desired state, and when it fails you write the desired state in your recipe. You then test again, and it passes. This is done with Test Kitchen.

# CHEF WORKFLOW BASICS

## CONTINUOUS DELIVERY
*Candidates should understand:*

### What continuous delivery (CD) is
- Continuous delivery is a software engineering approach in which teams produce software in short cycles, ensuring that the software can be reliably released at any time. It aims at building, testing, and releasing software faster and more frequently. (Wikipedia)

### What role Chef plays in CD		
- Chef allows for TDD with Test Kitchen, which produces reliability. And storing in source control makes it easier to develop collaboratively, leading to faster releases as the most reliable and stable branches are promoted through the environments. 

### When to run tests
- Tests should be run every time a change is made or the node is converged.

### Why automated configuration management is critical to CD
- Uniformity through automation minimizes error.

### Why CD is *more* secure than manual processes
- Removing the risk of human error, you can see your infrastructure and compliance as code, seeing its state in code before it is actually changed.

## USING	COMPLIANCE TO SCAN
*Candidates should understand:*

### The benefits of the agentless nature of Chef compliance
- A test is more reliable if nothing is installed on the node being installed because you don't have to change the state of the node being tested; you merely have to inspect it.

### How to check for compliance on nodes that don't have the Chef client installed
- You simply need to be able to have remote access into the node in order for the InSpec framework to use your profile to scan the node being inspected.

### Basic use cases for compliance
- Regulatory security and compliance standards
- Configuration validation
- State discovery 

### What language is used to express compliance requirements
- InSpec

## USING CHEF DK TO TEST YOUR CHANGES
*Candidates should understand:*

### The Test Kitchen value proposition
- Test Kitchen is at the heart of TDD within Chef. Being able to test your configuration before changing the state of your machine is invaluable because you can configure the state of a throw-away machine without having to change your actual infrastructure.

### Basic	use	cases	for	Chef DK	
- cookbook development (as well as other node data such as roles and environments)
- bootstrapping nodes with the Chef client
- communicating with the Chef server

## PUBLISHING ARTIFACTS TO CHEF SERVER AND SUPERMARKET
*Candidates should understand:*

### How to publish artifacts to Chef server
- One would use a `knife` command to upload artifacts to the Chef server.

### What Berkshelf is
- Berkshelf is a tool which one can use to upload artifacts such as cookbooks to the Chef server.

### If the Chef Automate workflow feature can push artifacts to things other than a Chef server or Supermarket
- Yes, Chef Automate can push artifacts to places like GitHub or production servers.

### How to manage cookbook dependencies
- Each cookbook that a cookbook is dependent upon must be included in that cookbook's metadata. After executing a `berks install`, 

## UNDERSTANDING BASIC CHEF CODE

## APPROACHABLE CUSTOM CODE
*Candidates should understand:*

### How to recognize custom code
- Chef is a thin domain-specific-language, meaning that it was built on top of Ruby. This means that you're able to easily customize with Ruby.

### How to use libraries
- Ruby code can be stored in a cookbook's `libraries` directory and can be implemented / called anywhere in the cookbook's recipes or custom resources.

### How to customize Chef
- Chef is a thin domain-specific-language, meaning that it was built on top of Ruby. This means that you're able to easily customize with Ruby.

## APPROACHABLE CHEF CODE
*Candidates should understand:*

### How to read a recipe that includes the 'package', 'file', and 'service' resources and describe its intent.
- (studied in kata)