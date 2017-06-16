# LOCAL COOKBOOK DEVELOPMENT BADGE TOPICS
_This page is for the purpose of studying for this exam. Most of the information found on this page is taken from [docs.chef.io](https://docs.chef.io)._

The Local Cookbook Development badge is awarded when someone proves that they understand the process of developing cookbooks locally. Candidates must show:

- An understanding of authoring cookbooks and setting up the local environment.
- An understanding of the Chef DK tools.
- An understanding of Test Kitchen configuration.
- An understanding of the available testing frameworks.
- An understanding of troubleshooting cookbooks.
- An understanding of search and databags.

_Here is a detailed breakdown of each area._

# COOKBOOK AUTHORING AND SETUP THEORY

## REPO STRUCTURE - MONOLITHIC VS SINGLE COOKBOOK
_Candidates should understand:_

### The pros and cons of a single repository per cookbook
 - _Cookbooks should not reside in the Chef Repo but rather be pulled in via dependency management tools like Berkshelf. Each cookbook should have its own Git repository, build process, and test suite. Cookbooks should be treated as software projects of their own. The suggested structure is to completely remove the idea of putting Cookbooks in your Chef Repo all together. Every cookbook would be contained within its own Git repository and every cookbook has its own Berksfile._
 - From there they suggest creating a build job on a CI server for every cookbook. This job would test and then upload the cookbook it is managing to your Chef Server.

**Pros -**
 - Each cookbook can be tested, uploaded, and verified by a build server 
 - Change management is simpler 
   - pinning versions to environments is straightforward
   - there can be an open source mentality toward changes to the cookbook
   - the run-list will only allow for one cookbook of that name per organization otherwise confusion abounds
 - You can use the "monolithic" chef-repo pattern as a "management console".
 - A different Berksfile per cookbook means that each cookbook can have different dependencies. So if cookbook A depends on apache v 2.0.0 and cookbook B depends on apache v 2.2.2, B's dependencies don't cancel out A's.

**Cons -**
 - The cookbook is shared with other applications, so a change might cause issues.
   - you may or may not be the maintainer of that cookbook which means you may not have rights to change it on the master branch
   - you may need a wrapper cookbook to extend the functionality of that cookbook

### [The pros and cons of an application repository](https://chef.github.io/chef-rfc/rfc019-chef-workflows.html)
**Pros -**
 - Everything needed for your application is in one place so it lessens confusion.
 - `chef vendor dependencies` does what berks would do to install cookbook dependencies.
 - You can test all your cookbooks together.

**Cons -**
Everything needed for your application must be the same version, creating ownership and change management issues. 

### How the Chef workflow supports monolithic vs single cookbooks
 - Monolithic: All of your Chef related source code, including any 3rd party dependencies, are tracked in one source control repository using Git. External dependencies, and any local modifications to them, are made with built-in vendor branches, allowing you to easily track the upstream for modifications.
 - Single: All of the Chef cookbooks are treated as independent software projects, that can be built in isolation from any other cookbook. External dependencies are fetched as-needed, and treated as artifacts. Changes to the upstream creates a new software projects, and is tracked as such.

### How to create a repository/workspace on the workstation
 - You can either run `chef generate repo [name]` or download a starter kit from the Chef server for your organization.

## VERSIONING OF COOKBOOKS
_Candidates should understand:_

### Why cookbooks should be versioned
 - Cookbooks need versions for running different cookbooks on their different environments.

### The recommended methods of maintaining versions (e.g. knife spork)
 - `knife spork bump <cookbook>` will automatically update the version number in your metadata if you don't manually change it in the `metadata.rb`.

### How to avoid overwriting cookbooks
 - by Freezing it with the `knife cookbook upload <cookbook> --freeze` or `berks upload` which freezes

### Where to define a cookbook version
 - in `metadata.rb`

### Semantic versioning
 - `MAJOR.MINOR.PATCH` or `BreakingChanges.BackwardsCompatibleChanges.BackwardsCompatibleBugFixes`
   - MAJOR version when you make incompatible API changes
   - MINOR version when you add functionality in a backwards-compatible manner
   - PATCH version when you make backwards-compatible bug fixes

### [Freezing cookbooks](https://docs.chef.io/cookbook_versions.html#freeze-versions)
 - A cookbook version can be frozen, which will prevent updates from being made to that version of a cookbook. (A user can always upload a new version of a cookbook.) Using cookbook versions that are frozen within environments is a reliable way to keep a production environment safe from accidental updates while testing changes that are made to a development infrastructure.

### Re-uploading and freezing cookbooks
 - To freeze a cookbook version using knife, enter: `knife cookbook upload redis --freeze`
 - Once a cookbook version is frozen, only by using the --force option can an update be made. `knife cookbook upload redis --force`

## STRUCTURING COOKBOOK CONTENT
_Candidates should understand:_

### Modular content/reusability

### Best practices around cookbooks that map 1:1 to a piece of software or functionality vs monolithic cookbooks
 - 1:1 cookbooks are preferred as each piece of software needs to be managed separately within its own git repo.
   - Versioning - you can tag specific releases 
   - Hands in the pot - you can easily give everyone clone access, but restrict push access to select teams for certain repos 
   - History - if you need to history for a certain cookbook, you shouldn't have to run a complex git command to parse the logs 
   - Monolithic things are generally anti-patterns

### How to use common, core resources
 - (This is accomplished with the [chefkata](https://github.com/anniehedgpeth/chefkata).)
 - `:nothing` is the only action that can be used with any resource.
 - `file` `cookbook_file` `remote_file` `template` actions: 
   - `:create` `:create_if_missing` `:delete` `:nothing` `:touch`
 - Properties common to every resource include:
   - `ignore_failure` `provider` `retries` `retry_delay` `sensitive` `supports`
   - [`sensitive`](https://docs.chef.io/resource_common.html#properties) Ensure that sensitive resource data is not logged by the chef-client. Default value: false. This property only applies to the `execute`, `file` and `template` resources.
 - `execute` actions: `:nothing` `:run`
 - Common `execute` properties: `command` `notifies` `creates` (Prevent a command from creating a file when that file already exists.) `path` `returns`
 - A notification is a property on a resource that listens to other resources in the resource collection and then takes action(s) based on the notification type (notifies or subscribes).
   - Timers: `:before` `:delayed` `:immediate` `:immediately`
   - Notifies: `notifies :action, 'resource[name]', :timer`
   - Subscribes: `subscribes :action, 'resource[name]', :timer`

```ruby
cookbook_file '/var/www/customers/public_html/index.php' do
  source 'index.php'
  owner 'web_admin'
  group 'web_admin'
  mode '0755'
  action :create
end
```

## HOW METADATA IS USED
_Candidates should understand:_

### How to manage dependencies
 - If you're using Berks, you would add `depends 'cookbook', 'version'` in the `metadata.rb`, and in the `Berksfile` you would add the `source`, such as the supermarket at `'https://supermarket.chef.io'`. 
 - If you're not using Berks, then you would add the dependencies to the `metadata.rb` in the same way, but you would run `knife upload `knife deps nodes/*.json` to use the output of knife deps to pass command to knife upload.

### Cookbook dependency version syntax
 - Inside of `metadata.rb` put `depends '<cookbook>'` or for a version constraint, `depends '<cookbook>', '> 2.0'`
   - The operators are `=, >=, >, <, <=, and ~>`. 
     - That last operator `~>` will go up to the next biggest version.
     - `>= 2.2.0, < 3.0` == `~>2.2`

### What information to include in a cookbook - author, license, etc
**Metadata settings**

```ruby
name 'chefkata'
maintainer 'Annie Hedgpeth'
maintainer_email 'annie.hedgpeth@gmail.com'
license 'all_rights'
description 'Installs/Configures chefkata'
long_description 'Installs/Configures chefkata'
version '0.1.0'
issues_url 'https://github.com/<insert_org_here>/chefkata/issues' if respond_to?(:issues_url)
source_url 'https://github.com/<insert_org_here>/chefkata' if respond_to?(:source_url)
```

### What 'suggests' in metadata means
 - Same thing as depends, but that you suggest a cookbook be there. The cookbook will still run if that dependency doesn't exist.

### What 'issues_url' in metadata means
 - The `issues_url` points to the location where issues for this cookbook are tracked.  A `View Issues` link will be displayed on this cookbook's page when uploaded to a Supermarket.
 - The `source_url` points to the development repository for this cookbook.  A `View Source` link will be displayed on this cookbook's page when uploaded to a Supermarket.


## WRAPPER COOKBOOK METHODS
_Candidates should understand:_

### How to consume other cookbooks in code via wrapper cookbooks
 - You would add those cookbooks as dependencies in the metadata and then consume them by either:
   - `include_recipe` in the recipe and set the attributes in the attributes file (i.e. chef-client cookbook with a lot of recipes)
   - use the resources provided by the cookbook (i.e. Tomcat cookbook with lots of resources and no recipes)

### How to change cookbook behavior via wrapper cookbooks
 - You would add those cookbooks as dependencies in the metadata and then consume them by either:
   - `include_recipe` in the recipe and set the attributes in the attributes file (i.e. chef-client cookbook with a lot of recipes)
   - use the resources provided by the cookbook (i.e. Tomcat cookbook with lots of resources and no recipes)
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) Set the attributes of the cookbook with `node.default['attribute'] = 'overridden_value'` and use `include_recipe`

### Attribute value precedence
 - OHAI will trump all other attributes. Then `override` will trump other attributes in the order of role, environment, node,/recipe, attribute files. Then the default attributes in that same order.
 - OHAI >> Normal (R, E, N, A) >> default (R, E, N, A)
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) [later overrides earlier](https://docs.chef.io/attributes.html):
   - Attribute -> Recipe -> Environment -> Role
   - Default -> Normal -> Override -> OHAI
  - declared in:
    - attribute: `default['attribute'] = value`, `normal['attribute'] = value`, `override['attribute'] = value`
    - cookbook: `node.default['attribute'] = value`, `node.normal['attribute'] = value`, `node.override['attribute'] = value`
    - environment: `default_attributes({ 'attribute' => 'value'})`, `override_attributes({'attribute' => 'value'})`
    - role: `default_attributes({ 'attribute' => 'value'})`, `override_attributes({'attribute' => 'value'})`

### How to use the `include_recipe` directive**
 - Add the recipe to your metadata.rb file, then use the `include_recipe` resource to run that entire recipe within the recipe in which you're including it.
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) add `depends 'cookbook_name'` to `metadata.rb` and then add `include_recipe 'cookbook_name::recipe_name'` to a recipe in your runlist

### What happens if the same recipe is included multiple times
 - If the `include_recipe` method is used more than once to include a recipe, only the first inclusion is processed and any subsequent inclusions are ignored.
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) Only the first inclusion is processed and any subsequent inclusions are ignored.

### How to use the `depends` directive
 - If you need a cookbook as a dependency, then you would include `depends '<cookbook>' '<version>'` in the metadata of the wrapper / main cookbook.

## USING COMMUNITY COOKBOOKS
_Candidates should understand:_

### How to use a public and private Supermarket
 - Private Supermarket -
   - Create a server to serve as your private supermarket which hosts your cookbooks
   - add the URL of that server as your source to the private supermarket to your Berksfile so that it can look there for the dependent cookbooks and also add it to the knife.rb to upload cookbooks.
   - add all dependencies in the metadata.rb with `depends '<cookbook>' '<version>'`
   - use `knife cookbook site share COOKBOOK_NAME CATEGORY (options)` to upload cookbooks to the supermarket
 - Public Supermarket - 
   - add the public supermarket link as your source in the Berksfile
   - add all dependencies in the metadata.rb with `depends '<cookbook>' '<version>'`
   - you may only update a cookbook in the public supermarket if you are the owner/maintainer
   - anyone can upload a cookbook to the public supermarket

### How to use community cookbooks
 - You can either use it from the Supermarket, in which you'd depend on it in metadata, or you can clone it from version control and upload it to the Chef server with berks.
 - The deprecated way to initialize Berkshelf is to run `berks init`, but now, when you run `chef generate cookbook` you will get a Berkshelf file in the cookbook.

### How to wrap community cookbooks
 - You would add those cookbooks as dependencies in the metadata and then consume them by either:
    - `include_recipe` in the recipe and set the attributes in the attributes file (i.e. chef-client cookbook with a lot of recipes)
    - use the resources provided by the cookbook (i.e. Tomcat cookbook with lots of resources and no recipes)

### How to fork community cookbooks
 - Fork it to your own repo in git and assume the responsibility to maintain that cookbook from your own repo

### How to use Berkshelf to download cookbooks
 - run `berks install` with a valid `Berksfile` containing the source for the dependencies

### How to configure a Berksfile
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) `source` should point to a supermarket (and the first one gets precedence, so you could use a private then public supermarket)
   - Always include a `metadata` line to get all the `depends` attributes from the `metadata.rb`.
   - If your cookbook comes from another location than the supermarket, specify it, as in: `cookbook "<cookbook>", git: "http://github.com/username/repo'"`

### How to use a Berksfile to manage a community cookbook and a local cookbook with the same name
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) By using a private supermarket or specifying the exact location of that cookbook on your private git server, thus making it ignore the supermarket as a default source

## USING CHEF RESOURCES VS ARBITRARY COMMANDS
_Candidates should understand:_

### [How to shell out to run commands](https://docs.chef.io/dsl_recipe.html#shell-out)
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) Use the `shell_out` (when errors don't matter) or `shell_out!` (raises error when command fails)
 - The `shell_out` method can be used to run a command against the node, and then display the output to the console when the log level is set to debug.
   - `shell_out(command_args)` where command_args is the command that is run against the node
 - The `shell_out!` method can be used to run a command against the node, display the output to the console when the log level is set to debug, and then raise an error when the method returns false.
   - `shell_out!(command_args)` where command_args is the command that is run against the node. This method will return true or false.
 - The `shell_out_with_systems_locale` method can be used to run a command against the node (via the `shell_out` method), but using the LC_ALL environment variable.
   - `shell_out_with_systems_locale(command_args)` where command_args is the command that is run against the node.

### How to do [logging](https://docs.chef.io/resource_log.html#chef-log-entries) with Chef
 - Set the [sensitive property](https://docs.chef.io/resource_common.html#properties) to `true` to keep your sensitive data from being shown in the logs.

```ruby
unless node['splunk']['upgrade_enabled']
  Chef::Log.fatal('The chef-splunk::upgrade recipe was added to the node,')
  Chef::Log.fatal('but the attribute `node["splunk"]["upgrade_enabled"]` was not set.')
  Chef::Log.fatal('I am bailing here so this node does not upgrade.')
  raise
end
```

### When/not to shell out
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) As read-only, not to change state

### How to use the `execute` resource
 - `execute '/usr/sbin/apachectl configtest'`

### When/not to use the `execute` resource
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) When you are changing the state of the system

### How to ensure idempotence
 - by using guard clauses
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) By using notifies or the `not_if`/`only_if` clauses

# CHEF DK TOOLS

## `CHEF` COMMAND
_Candidates should understand:_

### What the `chef` command does
 - The `chef` command is used for generation and is like the `knife` command. You'd use it in place of `knife` when you use `policyfiles`.
 - `chef command [arguments...] [options...]`

```
Available Commands:
    exec                    Runs the command in context of the embedded ruby
    env                     Prints environment variables used by ChefDK
    gem                     Runs the `gem` command in context of the embedded ruby
    generate                Generate a new app, cookbook, or component
    shell-init              Initialize your shell to use ChefDK as your primary ruby
    install                 Install cookbooks from a Policyfile and generate a locked cookbook set
    update                  Updates a Policyfile.lock.json with latest run_list and cookbooks
    push                    Push a local policy lock to a policy group on the server
    push-archive            Push a policy archive to a policy group on the server
    show-policy             Show policyfile objects on your Chef Server
    diff                    Generate an itemized diff of two Policyfile lock documents
    provision               Provision VMs and clusters via cookbook
    export                  Export a policy lock as a Chef Zero code repo
    clean-policy-revisions  Delete unused policy revisions on the server
    clean-policy-cookbooks  Delete unused policyfile cookbooks on the server
    delete-policy-group     Delete a policy group on the server
    delete-policy           Delete all revisions of a policy on the server
    undelete                Undo a delete command
    verify                  Test the embedded ChefDK applications
  ```

### What `chef generate` can create

```
Available generators:
  app             Generate an application repo
  cookbook        Generate a single cookbook
  recipe          Generate a new recipe
  attribute       Generate an attributes file
  template        Generate a file template
  file            Generate a cookbook file
  lwrp            Generate a lightweight resource/provider
  repo            Generate a Chef code repository
  policyfile      Generate a Policyfile for use with the install/push commands
  generator       Copy ChefDK's generator cookbook so you can customize it
  build-cookbook  Generate a build cookbook for use with Delivery
```

### [How to customize content using `generators`](https://blog.chef.io/2014/12/09/guest-post-creating-your-own-chef-cookbook-generator/)
 - `chef generate cookbook my_cookbook_name -g ~/chef/pan`
  - This is a way that you can templatize the way in which you create a chef repo with settings specific to your organization.

### The recommended way to create a template
 - `chef generate template <templatename>`

### How to add the same boilerplate text to every recipe created by a team
 - According to [this](https://medium.com/@echohack/creating-your-own-chef-cookbook-generator-5e998db1255b), you'd add it to the `code_generator` at `/opt/chefdk/embedded/apps/chef-dk/lib/chef-dk/skeletons/code_generator`.

### The ['chef gem'](https://docs.chef.io/ctl_chef.html#chef-gem) command
 - The chef gem subcommand is a wrapper around the gem command in RubyGems and is used by Chef to install RubyGems into the Chef development kit development environment. All knife plugins, drivers for Kitchen, and other Ruby applications that are not packaged within the Chef development kit will be installed to the .chefdk path in the home directory: ~/.chefdk/gem/ruby/ver.si.on/bin (where ver.si.on is the version of Ruby that is packaged within the Chef development kit).

## [FOODCRITIC](https://docs.chef.io/foodcritic.html)
_Candidates should understand:_

### What Foodcritic is
 - Chef-specific linting of cookbooks
 - Use Foodcritic to check cookbooks for common problems:
   - Style
   - Correctness
   - Syntax
   - Best practices
   - Common mistakes
   - Deprecations

### Why developers should lint their code
 - Consistency

### Foodcritic errors and how to fix them
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) They all start with `FC001`; you can google that to get to the exact rule.

### Community coding rules & custom rules
 - Various nice people in the Chef community have also written extra rules for foodcritic that you can install and run. Or write your own!
   - [Custom Customink Rules](https://github.com/customink-webops/foodcritic-rules/blob/master/rules.rb)
   - [Custom Etsy Rules](https://github.com/etsy/foodcritic-rules)

### Foodcritic commands
 - `foodcritic /path/to/cookbook`
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) Just run `foodcritic .` to do a scan from the cookbook folder. Add `--epic-fail` to make the command fail when foodcritic fails (to cause your build to fail).
 - A Foodcritic evaluation has the following syntax: `RULENUMBER: MESSAGE: FILEPATH:LINENUMBER`

### [Foodcritic rules](http://www.foodcritic.io/)
 - It comes with 60 built-in rules that identify problems ranging from simple style inconsistencies to difficult to diagnose issues that will hurt in production. 

### How to exclude Foodcritic rules
 - `foodcritic . --tags ~RULE`
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) For the entire cookbook, add `FC###` to the `.foodcritic` file in the root of the cookbook folder
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) For single line of code, add the comment at the end of the line: `# ~FC003`

## BERKS
_Candidates should understand:_

### How to use berks to work with upstream dependencies
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) by using the supermarket for dependencies, or by doing version pinning

### How to work with GitHub & Supermarket
 -  [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) in the `Berksfile` you can state the `git:` location or have another supermarket listed as your `source:` (or even multiple ones if you want public to be a backup)

### How to work with dependent cookbooks
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) Most of the time it works with `metadata.rb` inclusion, but you can extend it with the `cookbook` line (by including further version pinning and overriding location).

### How to troubleshoot berks issues
 - https://github.com/berkshelf/berkshelf/wiki/Troubleshooting
 - `berks test KITCHEN_COMMAND (options)`
 - `berks show COOKBOOK (options)`
 - `berks info COOKBOOK (options)`
 - `berks list (options)`
 - `berks search QUERY (options)`
 - `berks verify (options)`

### How to lock cookbook versions
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) cookbooks are frozen by default with a `berks upload`

### How to manage dependencies using berks
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) berks commands
   - `berks install` loads dependencies locally
   - `berks upload` uploads dependencies to chef server
   - `berks info <cookbook>` will display information for that cookbook
   - `berks list` will list cookbooks and their dependencies
   - `berks apply production Berksfile.lock` will apply the settings in berksfile to the provided environment

## [RUBOCOP](https://docs.chef.io/rubocop.html)
_Candidates should understand:_

### How to use RuboCop to check Ruby styles
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) `rubocop .` command from the cookbook folder

### RuboCop vs Foodcritic
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) Rubocop is for ruby style, Foodcritic is for chef specifid linting
 - Use RuboCop to author better Ruby code:
   - Enforce style conventions and best practices
   - Evaluate the code in a cookbook against metrics like “line length” and “function size”
   - Help every member of a team to author similary structured code
   - Establish uniformity of source code
   - Set expectations for fellow (and future) project contributors

### RuboCop configuration & commands
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) `--fail-fast` a good option for CI

### Auto correction
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) `-a` or `--auto-correct` to auto-correct offenses

### How to be selective about the rules you run
 - Each cookbook has its own `.rubocop.yml` file, which means that each cookbook may have its own set of enabled, disabled, and custom rules. That said, it’s more common for all cookbooks to have the same set of enabled, disabled, and custom rules. When RuboCop is run against a cookbook, the full set of enabled and disabled rules (as defined in the `enabled.yml` and `disabled.yml` files in RuboCop itself) are loaded first and are then compared against the settings in the cookbook’s `.rubocop.yml` file.

```ruby
NAME_OF_RULE:
  Description: 'a description of a rule'
  Enabled : (true or false)
  KEY: VALUE
```
 - Use a `.rubocop_todo.yml` file to capture the current state of all evaluations, and then write them to a file. This allows evaluations to reviewed one at a time. Disable any evaluations that are unhelpful, and then address the ones that are.
 - To generate the `.rubocop_todo.yml` file, run the following command: `rubocop --auto-gen-config`
   - Rename this file to `.rubocop.yml` to adopt this evaluation state as the standard. 
   - Include this file in the `.rubocop.yml` file by adding `inherit_from: .rubocop_todo.yml` to the top of the `.rubocop.yml` file.


## TEST KITCHEN
_Candidates should understand:_

### Writing tests to verify intent
 - One should use InSpec to write tests that verify that the desired state was achieved, not necessarily that all of the resources simply converged but that they're functioning in the desired state.

### How to focus tests on critical outcomes
 - One should use InSpec to write tests that verify that the desired state was achieved, not necessarily that all of the resources simply converged but that they're functioning in the desired state.

### How to test each resource component vs how to test for desired outcomes
 - One should use InSpec to write tests that verify that the desired state was achieved, not necessarily that all of the resources simply converged but that they're functioning in the desired state.

### Regression testing
 - Running `kitchen converge` twice will ensure your policy applies without error to existing instances
 - Running `kitchen test` will ensure your policy applies without error to any new instances

# TEST KITCHEN 
 - The basic structure of a `.kitchen.yml` file is as follows:

```yaml
driver:
  name: driver_name

provisioner:
  name: provisioner_name

verifier:
  name: verifier_name

transport:
  name: transport_name

platforms:
  - name: platform-version
    driver:
      name: driver_name
  - name: platform-version

suites:
  - name: suite_name
    run_list:
      - recipe[cookbook_name::recipe_name]
    attributes: { foo: "bar" }
    excludes:
      - platform-version
  - name: suite_name
    driver:
      name: driver_name
    run_list:
      - recipe[cookbook_name::recipe_name]
    attributes: { foo: "bar" }
    includes:
      - platform-version
```

## DRIVERS
_Candidates should understand:_

### Test Kitchen provider & platform support
 - `platforms:` contains a list of all the platforms that Kitchen will test against when executed. This should be a list of all the platforms that you want your cookbook to support.

### How to use .kitchen.yml to set up complex testing matrices
 - In the `.kitchen.yml` file we define two fields that create a test matrix; the number of **platforms** we want to support multiplied by the number of test **suites** that we defined.

### How to test a cookbook on multiple deployment scenarios
 - In the `.kitchen.yml` file we define two fields that create a test matrix; the number of **platforms** we want to support multiplied by the number of test **suites** that we defined.

### How to configure drivers
 - Kitchen uses a driver plugin architecture to enable Kitchen to simulate testing on cloud providers, such as Amazon EC2, OpenStack, and Rackspace, and also on non-cloud platforms, such as Microsoft Windows. Each driver is responsible for managing a virtual instance of that platform so that it may be used by Kitchen during cookbook testing.
   - Most drivers have driver-specific configuration settings that must be added to the `.kitchen.yml` file before Kitchen will be able to use that platform during cookbook testing. For information about these driver-specific settings, refer to the driver-specific documentation.
   - Common drivers include: 
     - `kitchen-all`   A driver for everything, or “all the drivers in a single Ruby gem”.
     - `kitchen-bluebox` A driver for Blue Box.
     - `kitchen-cloudstack` A driver for CloudStack.
     - `kitchen-digitalocean` A driver for DigitalOcean.
     - `kitchen-docker` A driver for Docker.
     - `kitchen-dsc` A driver for Windows PowerShell Desired State Configuration (DSC).
     - `kitchen-ec2` A driver for Amazon EC2.
     - `kitchen-fog` A driver for Fog, a Ruby gem for interacting with various cloud providers.
     - `kitchen-google` A driver for Google Compute Engine.
     - `kitchen-hyperv` A driver for Hyper-V Server.
     - `kitchen-joyent` A driver for Joyent.
     - `kitchen-linode` A driver for Linode.
     - `kitchen-opennebula` A driver for OpenNebula.
     - `kitchen-openstack` A driver for OpenStack.
     - `kitchen-pester` A driver for Pester, a testing framework for Microsoft Windows.
     - `kitchen-rackspace` A driver for Rackspace.
     - `kitchen-vagrant` A driver for Vagrant. The default driver packaged with the Chef development kit.
   - How to customize a driver:

```yaml
---
driver:
  customize:
    memory: 1024
    cpuexecutioncap: 50
```

```yaml
driver:
  customize:
    createhd:
      filename: /tmp/disk1.vmdk
      size: 1024
    storageattach:
      storagectl: SATA Controller
      port: 1
      device: 0
      type: hdd
      medium: /tmp/disk1.vmdk
```

```yaml
---
driver:
  network:
    - ["forwarded_port", {guest: 80, host: 8080}]
    - ["private_network", {ip: "192.168.33.33"}]
```

## PROVISIONER
_Candidates should understand:_

### The available provisioners
 - `chef_zero` and `chef_solo` are the most common provisioners used for testing cookbooks.

### How to configure provisioners

```yaml
provisioner:
  name: chef_zero
  http_proxy: http://10.0.0.1
```

 - The environment variables `http_proxy`, `https_proxy`, and `ftp_proxy` are honored by Kitchen for proxies. The `client.rb` file is read to look for proxy configuration settings. If `http_proxy`, `https_proxy`, and `ftp_proxy` are specified in the `client.rb` file, the `chef-client` will configure the ENV variable based on these (and related) settings. 

```ruby
http_proxy 'http://proxy.example.org:8080'
http_proxy_user 'myself'
http_proxy_pass 'Password1'
```

 - `ENV['http_proxy'] = 'http://myself:Password1@proxy.example.org:8080'`
 - Kitchen also supports `http_proxy` and `https_proxy` in the `.kitchen.yml` file:

### When to use `chef-client` vs. `chef-solo` vs. `Chef`
 - `chef-client` to converge the node when it is bootstrapped to the Chef server.
 - `chef-solo` as the provisioner for Test Kitchen when you run a light-weight version of Chef on the VM being tested
 - `Chef` when you interact with the Chef server
 
### How to use the [shell provisioner](https://www.morethanseven.net/2014/01/12/shell-provisioner-for-test-kitchen/)  
 - to run a command during a converge
 - `provisioner: shell`
 - The shell provisioner is going to look for a file called `bootstrap.sh` by default.
   - In this case our script is completely self contained but if it needed some additional files we could put them in a directory called data and they would be copied to the newly created virtual machine under `/tmp/kitchen`.

## SUITES
_Candidates should understand:_

### What a suite is
 - a scenario that you want to run that consists of set of names for each of the platforms specified

```yaml
  - name: suite_name
    driver:
      name: driver_name
    run_list:
      - recipe[cookbook_name::recipe_name]
    attributes: { foo: "bar" }
    includes:
      - platform-version
```

 - including or excluding platforms is available 

```yaml
 driver:
   name: vagrant

 provisioner:
   name: chef_zero

 platforms:
   - name: ubuntu-12.04
   - name: centos-6.4
   - name: debian-7.1.0

suites:
  - name: default
    run_list:
      - recipe[apache::httpd]
    excludes:
      - debian-7.1.0
```

### How to use suites to test different recipes in different environments
 - Each suite is given a runlist. (see above)
 - Attributes are given for each suite.

### Testing directory for InSpec
 - The InSpec tests are to be stored in `<cookbook>/test/<integration-OR-recipes>/<suite-name>`
 - If your path is different, then you must specify as such in the `.kitchen.yml`.

```yaml
  verifier:
    inspec_tests:
      - test/foo/bar
```

### How to configure suites
 - Answered above
 - 1) edit their run lists, and 2) define a platform for the suite in the `platforms` setting

## PLATFORMS
_Candidates should understand:_

### How to specify platforms
 - Within the `suites:` you may include or exclude platforms that are listed in your `platforms:` section.

```yaml
    includes:
      - platform-version
```

### Common platforms
 - ubuntu, debian, windows, centos, rhel

### How to locate base images
 - [Bento](https://github.com/chef/bento) is a project that contains a set of base images that are used by Chef for internal testing and to provide a comprehensive set of base images for use with Kitchen. By default, Kitchen uses the base images provided by Bento. (Custom images may also be built using Packer.)
 - If you are using a vagrant image, you can go to the Atlas site to see the available [Bento boxes](https://atlas.hashicorp.com/boxes/search?utf8=%E2%9C%93&sort=&provider=&q=bento).

### Common images and custom images
 - The common images are from [Bento](https://atlas.hashicorp.com/boxes/search?utf8=%E2%9C%93&sort=&provider=&q=bento).
 - If you want to make a custom image, then you would build it with [Packer](https://www.packer.io/intro/getting-started/build-image.html), then upload those files to Atlas.
 - You may also use images built by the community on the Atlas site. 

## KITCHEN COMMANDS
_Candidates should understand:_

### The basic Test Kitchen workflow
 - `kitchen create` to create the vm instance
 - `kitchen converge` to compile and converge the cookbooks on the vm instance
 - `kitchen verify` to run the InSpec tests on the instance
 - `kitchen destroy` to destroy the instance

### 'kitchen' commands
 - `kitchen create` to create the vm instance
 - `kitchen converge` to compile and converge the cookbooks on the vm instance
 - `kitchen verify` to run the InSpec tests on the instance
 - `kitchen destroy` to destroy the instance
 - `kitchen test` to destroy the instance if one exits followed by running through all of the above tests and finishing with another destroy
 - `kitchen login` to log into the vm instance
 - `kitchen init` to initialize a new `.kitchen.yml`

### When tests get run
 - when you run `kitchen verify`

### How to install [bussers](https://docs.chef.io/kitchen.html#busser) 
 - Busser is a test setup and execution framework that is designed to work on remote nodes upon whose system dependencies cannot be relied. 
   - Kitchen uses Busser to run post-convergence tests via a plugin architecture that supports different test frameworks. Busser is installed automatically as part of Kitchen.
   - InSpec is the busser that is being used, but you could choose a different busser (but why would you?).

### What 'kitchen init' does
 - `kitchen init` to initialize a new `.kitchen.yml`

# COOKBOOK COMPONENTS 

## DIRECTORY STRUCTURE OF A COOKBOOK
_Candidates should understand:_

### What the components of a cookbook are
 - Directories are:
   - recipes
   - files
   - attributes
   - test
   - spec
   - libraries
   - templates
   - definitions (don't want to use it, but you should be aware of it)
   - resources
   - providers (only if the cookbook employs lwrp or hwrp)
 - Files are:
   - .kitchen.yml
   - Berksfile
   - metadata.rb
   - README.md
   - chefignore

### What siblings of cookbooks in a repository are
 - data_bags
 - roles
 - environments

### The default recipe & attributes files
 - The default recipe is the only one that is run when a cookbook called without a recipe specified.
 - An [attribute file](https://docs.chef.io/attributes.html) is located in the attributes/ sub-directory for a cookbook. 
   - When a cookbook is run against a node, the attributes contained in all attribute files are evaluated in the context of the node object. 
   - Node methods (when present) are used to set attribute values on a node.

### Why there is a 'default' subdirectory under 'templates’
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) It is the fallback of where to go for templates. Templates can also be platform specific, where the platform would be the directory. Or you can specify the group in the recipe code, which would be configurable

### Where tests are stored
 -  inspec: `tests/integration/default`
 -  serverspec: `spec` directory

## ATTRIBUTES AND HOW THEY WORK 
_Candidates should understand:_

### What attributes are
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) values defined in a file within the attributes directory

### Attributes as a nested hash
 - The following two blocks say the same thing. The second displays the attributes in a nested hash.

```ruby
default['cookbook_name']['category_name']['key1'] = 'value1'
default['cookbook_name']['category_name']['key2'] = 'value2'
default['cookbook_name']['different_category'] = 'other_value'
```

```ruby
default['cookbook_name'] = {
  category_name: {
    value1: 'value1',
    value2: 'value2'
  },
  different_category: 'other_value'
}
```

### How attributes are defined
 - Attributes are defined by:
   - Attributes file (can be `.json` or `.rb`)

```ruby
default['cookbook_name']['category_name']['key1'] = 'value1'
default['cookbook_name']['category_name']['key2'] = 'value2'
normal['cookbook_name']['different_category'] = 'other_value' # normal overrides the default value
```
   - In recipes - 

```ruby
node.default['cookbook_name']['category_name']['key1'] = 'recipe_value'
node.normal['cookbook_name']['category_name']['key2'] = 'recipe_value2'
```

   - Roles & Environments

```ruby
override_attributes({
  category_name: {
    value1: 'new_value1',
    value2: 'new_value2'
  }
})
default_attributes({
  category_name: {
    value1: 'new_value1-a',
    value2: 'new_value2-b'
  }
})
```

### How attributes are named
 - in a nested tree

### How attributes are referenced
 - `node[key:value]`

### Attribute precedence levels
 - OHAI >> Normal/Override (Role, Env, Node/recipe, Attribute) >> default (Role, Env, Node/recipe, Attribute)

### What [Ohai](https://docs.chef.io/ohai.html) is
 - Ohai is a tool that is used to detect attributes on a node, and then provide these attributes to the chef-client at the start of every chef-client run. Ohai is required by the chef-client and must be present on a node. (Ohai is installed on a node as part of the chef-client install process.)
 - Attributes that are collected by Ohai are automatic level attributes, in that these attributes are used by the chef-client to ensure that these attributes remain unchanged after the chef-client is done configuring the node.

### What the 'platform' attribute is
 - [Ohai](https://docs.chef.io/ohai.html) collects data for many platforms, including AIX, Darwin, Linux, FreeBSD, OpenBSD, NetBSD, Solaris, and any Microsoft Windows operating system based off the Windows_NT kernel and has access to win32 or win64 sub-systems.

### How to use the 'platform' attribute in [recipes](https://docs.chef.io/dsl_recipe.html)

```ruby
if node['platform'] == 'ubuntu'
  # do ubuntu things
end
```

## FILES AND TEMPLATES - DIFFERENCE AND HOW THEY WORK, WHEN TO USE EACH
_Candidates should understand:_

### How to instantiate [files](https://docs.chef.io/files.html) on nodes
 - Use the `cookbook_file` resource to manage files that are added to nodes based on files that are located in the /files directory in a cookbook.
 - Use the `file` resource to manage files directly on a node.
 - Use the `remote_file` resource to transfer files to nodes from remote locations.
 - Use the `template` resource to manage files that are added to nodes based on files that are located in the /templates directory in a cookbook.

### The difference between `file`, `cookbook_file`, `remote_file`, and `template`
   - `file` - Use this when you want to create a simple file. You'll use the `content` property to add the content. 
   - `cookbook_file` - Use this when you want to store a file in the cookbook to be copied directly onto the node.
   - `source` - Use this when you have a remote source on which the file is stored that you want put on the node.
   - `template` - Use this when you want to create a file out of a template based on environment variables.

### How two teams can manage the same file
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) If they can share the same cookbok, then by overriding attributes on the same cookbook. Otherwise use partial templates (see below).

### How to write templates
 - You would use the `template` resource in the recipe:

```ruby
template '/etc/motd' do
  source 'motd.erb'
  owner 'root'
  group 'root'
  mode '0755'
  variables({
    key: 'value'
  })
end
```
 - And the actual template would be the file name with its extension and an `.erb` extension. The file would look like:

```
message = <%= message %>
```

### What ['partial templates'](https://docs.chef.io/resource_template.html#partial-templates) are
 - A template can be built in a way that allows it to contain references to one (or more) smaller template files. 
 - These smaller template files are also referred to as partials.
 - A partial can be referenced from a template file in one of the following ways:
   - By using the render method in the template file
     - `<%= render "simple.txt.erb", :variables => {:user => Etc.getlogin }, :local => true %>`
   - By using the template resource and the variables property

### Common file-related resource actions and properties
 - `file` `cookbook_file` `remote_file` `template` actions: 
   - `:create` `:create_if_missing` `:delete` `:nothing` `:touch`
 - Properties common to every resource include:
   - `ignore_failure` `provider` `retries` `retry_delay` `sensitive` `supports`
- Common file-related resource properties:
   - `notifies` `mode` `group` `content` `owner` `subscribes`

### [ERB syntax](https://docs.puppet.com/puppet/4.9/lang_template_erb.html)
 - Each ERB tag has a beginning tag and a matched ending tag. `<% code %>`
   - This executes the ruby code within the brackets and does not display the result. `<% if (50 + 50) == 100 %>`
   - We use `<%= code %>` when we want to show the value stored in a variable or the result of some calculation.
     - For example: `<%= node['hostname'] %>`

## CUSTOM RESOURCES - HOW THEY ARE STRUCTURED AND WHERE THEY GO
_Candidates should understand:_

### What custom resources are
 - A custom resource can either be a hwrp, lwrp, a definition, or a custom resource.

### How to consume resources specified in another cookbook
 -   depend on cookbook, and then use them

### Naming conventions
 - If you do not explicitly name the custom resource then it will be named like `<cookbook>_<filename>`

### How to test custom resources
 - [MH:](https://github.com/mhedgpeth/mhedgpeth.github.io/blob/master/_drafts/local-cookbook-development-notes.md) by using a wrapper cookbook embedded inside of the cookbook (I usually put it in `test/cookbooks` and then included in the runlist)


## [LIBRARIES](https://docs.chef.io/libraries.html) 
_Candidates should understand:_

### What libraries are and when to use them
 - In the same way that the `resources` directory is where you'd store custom resources, you will store ruby code that you want to reuse in the `libraries` directory.
 - Use a library to:
   - Create a custom class or module; for example, create a subclass of Chef::Recipe
   - Connect to a database
   - Talk to an LDAP provider
   - Do anything that can be done with Ruby
 
### Where libraries are stored
 - A library file is a Ruby file that is located within a cookbook’s `/libraries` directory.

# AVAILALABLE TESTING FRAMEWORKS 

## INSPEC
_Candidates should understand:_

### How to test common resources with InSpec
 - see below

### InSpec syntax

```ruby
describe bash('command') do
  it { should exist }
  its('matcher') { should eq 'output' }
end

describe file('path') do
  it { should MATCHER 'value' }
end

describe http('url', auth: {user: 'user', pass: 'test'}, params: {params}, method: 'method', headers: {headers}, body: body) do
  its('status') { should eq number }
  its('body') { should eq 'body' }
  its('headers.name') { should eq 'header' }
end

describe os[:family] do
  it { should eq 'platform_name' }
end
```

### How to write InSpec tests
 - see above

### How to run InSpec tests
 - `inspec exec path/to/test`
 - After running the tests, you will receive feedback about what was retuned in regard to what was expected.

```ruby
describe some_resource('/proc/cpuinfo') do
  its('mode') { should cmp '0345' }
end

expected: 0345
got: 0444
```

### [Where InSpec tests are stored](http://www.anniehedgie.com/inspec-basics-6)
 - locally
 - Supermarket
 - git
 - Chef Compliance Server

## CHEFSPEC
Candidates should understand:

### What ChefSpec is
 - ChefSpec is a framework that tests resources and recipes as part of a simulated chef-client run.

### The ChefSpec value proposition
 - The benefit of writing tests focused around the Resource Collection will allow us to gain feedback quickly and build a better development workflow.

### What happens when you run ChefSpec
 - it tests the resources compiled, not running those resources

### ChefSpec syntax

```ruby
describe 'scenario'
  context 'when something happens' do
    let :chef_run do
      runner = ChefSpec::SoloRunner.new(platform: 'windows', version: '2012R2')
      runner.converge(described_recipe)
    end

    it 'converges successfully' do
      expect { chef_run }.to_not raise_error
    end
  end
```

### How to write [ChefSpec tests](https://docs.chef.io/chefspec.html)

```ruby
# Recipe - file resource

file '/tmp/explicit_action' do
  action :delete
end

file '/tmp/with_attributes' do
  user 'user'
  group 'group'
  backup false
  action :delete
end

file 'specifying the identity attribute' do
  path   '/tmp/identity_attribute'
 action :delete
end
```

```ruby
# Unit Test
require 'chefspec'

describe 'file::delete' do
  let(:chef_run) { ChefSpec::SoloRunner.new(platform: 'ubuntu', version: '16.04').converge(described_recipe) }

  it 'deletes a file with an explicit action' do
    expect(chef_run).to delete_file('/tmp/explicit_action')
    expect(chef_run).to_not delete_file('/tmp/not_explicit_action')
  end

  it 'deletes a file with attributes' do
    expect(chef_run).to delete_file('/tmp/with_attributes').with(backup: false)
    expect(chef_run).to_not delete_file('/tmp/with_attributes').with(backup: true)
  end

  it 'deletes a file when specifying the identity attribute' do
    expect(chef_run).to delete_file('/tmp/identity_attribute')
  end
end
```
### How to run ChefSpec tests
 - `rspec` or `chef exec rspec`

### Where ChefSpec tests are stored
 - `spec/unit/recipes`

## GENERIC TESTING TOPICS
_Candidates should understand:_

### The test-driven development (TDD) workflow
 - Define a test set for the unit first
 - Then implement the unit
 - Finally verify that the implementation of the unit makes the tests succeed.
 - Refactor

### Where tests are stored
 - Inspec: `<cookbook>/test/<integration-OR-recipes>/<suite-name>`
 - ChefSpec: `<cookbook>/spec/unit/recipes`

### How tests are organized in a cookbook
 - by suites

### Naming conventions - how Test Kitchen finds tests
 - The InSpec tests are to be stored in `<cookbook>/test/<integration-OR-recipes>/<suite-name>`
 - If your path is different, then you must specify as such in the .kitchen.yml.

```yaml
  verifier:
    inspec_tests:
      - test/foo/bar
```

### Tools to test code "at rest" 
 - Foodcritic and Rubocop

### Integration testing tools
 - InSpec

### Tools to run code and test the output
 - InSpec

### When to use ChefSpec in the workflow  
 - after linting and before Kitchen

### When to use Test Kitchen in the workflow
 - after ChefSpec and before promotion to the Chef server or Supermarket

### Testing intent
 - One should use InSpec to write tests that verify that the desired state was achieved, not necessarily that all of the resources simply converged but that they're functioning in the desired state.

### Functional vs unit testing
 - Functional is basically integration testing which you'd do with InSpec.
 - Unit testing would be done with ChefSpec.


# TROUBLESHOOTING 

## READING TEST-KITCHEN OUTPUT
_Candidates should understand:_

### Test Kitchen phases and associated output
 - `kitchen create` to create the vm instance
 - `kitchen converge` to compile and converge the cookbooks on the vm instance
 - `kitchen verify` to run the InSpec tests on the instance
 - `kitchen destroy` to destroy the instance
 - `kitchen test` to destroy the instance if one exits followed by running through all of the above tests and finishing with another destroy
 - `kitchen login` to log into the vm instance
 - `kitchen init` to initialize a new `.kitchen.yml`

## COMPILE VS. CONVERGE
_Candidates should understand:_

### What happens during the compile phase of a chef-client run
 - All of the code is run in order to get the resources ready for consumption.

### What happens during the converge phase of a chef-client run
 - All of the resources are executed on the node.

### When pure Ruby gets executed
 - This happens in the compile phase.

### When Chef code gets executed
 - This happens during the converge phase.


# SEARCH AND DATABAGS 

## [DATA BAGS](https://docs.chef.io/data_bags.html)
_Candidates should understand:_

### What databags are
 - A data bag is a global variable that is stored as JSON data and is accessible from a Chef server. 
 - A data bag is indexed for searching and can be loaded by a recipe or accessed during a search.
 - A data bag is a container of related data bag items, where each individual data bag item is a JSON file. 
   - `knife` can load a data bag item by specifying the name of the data bag to which the item belongs and then the filename of the data bag item. 
   - The only structural requirement of a data bag item is that it must have an id:

```json
{
  /* This is a supported comment style */
  // This style is also supported
  "id": "ITEM_NAME",
  "key": "value"
}
```

### Where databags are stored
 - A `data_bags` directory is sibling to the `cookbooks` directory in the `chef-repo`. Individual databags are stored `data_bags/BAG_NAME/ITEM_NAME.json`
   - When the chef-repo is cloned from GitHub, the following occurs:
     - A directory named data_bags is created.
     - For each data bag, a sub-directory is created that has the same name as the data bag.
     - For each data bag item, a JSON file is created and placed in the appropriate sub-directory.

```
- data_bags
    -  admins
        -  charlie.json
        -  bob.json
        -  tom.json
    -  db_users
        -  charlie.json
        -  bob.json
        -  sarah.json
    -  db_config
        -  small.json
        -  medium.json
        -  large.json
```

### When to use [databags](https://docs.chef.io/data_bags.html#use-data-bags)
 - when you want data to be accessed by multiple cookbooks within your Chef server
   - Values that are stored in a data bag are global to the organization and are available to any environment.

### How to use [databags](https://docs.chef.io/data_bags.html#use-data-bags)
 - Data bags can be accessed in the following ways:
   - with **Search**
     - using knife `knife search admin_data "(NOT id:admin_users)"`
     - in a recipe `search(:admins, "id:charlie")`
 
   - with **Environments**
     -  data bag that is storing a top-level key for an environment might look something like this:

```json
{
  "id": "some_data_bag_item",
  "production" : {
    # Hash with all your data here
  },
  "testing" : {
    # Hash with all your data here
  }
}
```
     - When using the data bag in a recipe, that data can be accessed from a recipe using code similar to:

```ruby
bag_item[node.chef_environment]['some_other_key']
```
   - with **Recipes**
     - Loaded by name when using the Recipe DSL. Use this approach when a only single, known data bag item is required.
       - `data_bag(bag)`, where `bag` is the name of the data bag.
         - For example, the contents of a data bag item named justin: `data_bag_item('admins', 'justin')` will return something similar to: `# => {'comment'=>'Justin Currie', 'gid'=>1005, 'id'=>'justin', 'uid'=>1005, 'shell'=>'/bin/zsh'}`
       - `data_bag_item('bag_name', 'item', 'secret')`, where `bag` is the name of the data bag and item is the name of the data bag item. If `'secret'` is not specified, the `chef-client` will look for a secret at the path specified by the `encrypted_data_bag_secret` setting in the `client.rb` file.
     - Accessed through the search indexes. Use this approach when more than one data bag item is required or when the contents of a data bag are looped through. The search indexes will bulk-load all of the data bag items, which will result in a lower overhead than if each data bag item were loaded by name.
 
   - with **Chef Solo**
     - chef-solo can load data from a data bag as long as the contents of that data bag are accessible from a directory structure that exists on the same machine as chef-solo. The location of this directory is configurable using the data_bag_path option in the solo.rb file.

### How to create a databag
 - A data bag can be created in two ways: using `knife` or manually. In general, using knife to create data bags is recommended, but as long as the data bag folders and data bag item JSON files are created correctly, either method is safe and effective.
   - `knife data bag create DATA_BAG_NAME (DATA_BAG_ITEM)`
   - `knife data bag from file BAG_NAME ITEM_NAME.json`
     - This will load the following file: `data_bags/BAG_NAME/ITEM_NAME.json`

### How to update a databag
 - A data bag can be edited in two ways: 
   - using `knife` 
     - `knife data bag edit BAG_NAME ITEM_NAME` will open the $EDITOR
     - Once opened, you can update the data before saving it to the Chef server.
     - (You can also just edit it in your own editor, then `knife data bag from file BAG_NAME ITEM_NAME.json` again.)
   - using the Chef management console (the Chef server UI on manage.chef.io)
       - Click Policy.
       - Click Data Bags.
       - Select a data bag.
       - Select the Items tab.
       - Select a data bag.
       - Click Edit.

### How to search databags
 - A data bag is a global variable that is stored as JSON data and is accessible from a Chef server. 
 - A data bag is indexed for searching and can be loaded by a recipe or accessed during a search.

```ruby
search(:admins, "*:*")
```

```ruby
search(:admins, "id:charlie")
```

```ruby
search(:admins, "id:c*")
```

```ruby
admins = data_bag('admins')

admins.each do |login|
  admin = data_bag_item('admins', login)
  home = "/home/#{login}"

  user(login) do
    uid       admin['uid']
    gid       admin['gid']
    shell     admin['shell']
    comment   admin['comment']
    home      home
    manage_home true
  end

end
```

### [Chef Vault](https://docs.chef.io/chef_vault.html)
 - Chef Vault is similar to encryped databags except that it provides two layers of encryption instead of just one.
 - chef-vault allows the encryption of a data bag item by using the public keys of a list of nodes, allowing only those nodes to decrypt the encrypted values. `chef-vault` adds the `knife vault` subcommand.
   - The chef-vault cookbook is maintained by Chef. Use it along with chef-vault itself. This cookbook adds the `chef_vault_item` helper method to the Recipe DSL and the `chef_vault_secret` resource. Use them both in recipes to work with data bag secrets.

### The difference between databags and attributes
 - Databags have no precedence and are cookbook independent.

### What `knife` commands to use to CRUD databags
 - create: `knife data bag create myproduct`
 - read: `knife data bag show myproduct values`
 - update: either `knife data bag edit myproduct values` or `knife data bag from file myproduct values.json` (preferred)
 - delete: `knife data bag delete myproduct`


## [SEARCH](https://docs.chef.io/chef_search.html)
_Candidates should understand:_

### What data is indexed and searchable
 - Search indexes allow queries to be made for any type of data that is indexed by the Chef server, including data bags (and data bag items), environments, nodes, and roles. 

### Why you would search in a recipe
 - There is a lot of information that `chef-client` will not have until it is run, like what kind of node it is on, which environment it is in, what role it has, etc. We can invoke a search to fill in the proper attributes and values to be consumed by the recipe as needed. 

### Search criteria syntax
 - A search query is comprised of two parts: the key and the search pattern. A search query has the following syntax: `key:search_pattern` where `key` is a field name that is found in the JSON description of an indexable object on the Chef server (a role, node, client, environment, or data bag) and `search_pattern` defines what will be searched for, using one of the following search patterns: exact, wildcard, range, or fuzzy matching. 
 - Both `key` and `search_pattern` are case-sensitive; `key` has limited support for multiple character wildcard matching using an asterisk (“*”) (and as long as it is not the first character).

### How to invoke a search from the command line
 - `knife search` commands can be use to search the Chef server.
 - `knife search node 'network_interfaces__addresses:192.168*'`
 - `knife search admins 'id:charlie'`
 - `knife search node 'foo:*'`
 - Use the `AND`, `NOT`, or `OR` [boolean operators](https://docs.chef.io/chef_search.html#and) 
   - `knife search node 'platform:windows AND roles:jenkins'`
   - `knife search sample "(NOT id:foo)"`
   - `knife search sample "id:foo OR id:abc"`

### How to invoke a search from within a recipe
 - `search(:node, "key:attribute")`
 - `search(:node, 'roles:load_balancer')`

```ruby
# search node
search(:node, "*:*").each do |matching_node|
  puts matching_node.to_s
end
```

```ruby
search(:node, 'platform:ubuntu AND name:CHEF*').each |matching_node|
  puts matching_node['ipaddress']
  puts matching_node['name']
end
```

```ruby
# search environments
qa_nodes = search(:node,"chef_environment:QA")
qa_nodes.each do |qa_node|
    # Do useful work specific to qa nodes only
end
```

```ruby
# search data bags (see above for more)
search(:admin_data, "NOT id:admin_users")
```