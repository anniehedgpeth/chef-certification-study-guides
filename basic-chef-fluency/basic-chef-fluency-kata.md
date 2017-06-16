# Basic Chef Fluency Kata
Originally posted in his Github repo, [Michael Hedgpeth](https://github.com/mhedgpeth/chef-by-example/blob/master/basic-chef-fluency.md) came up with this simple exercise that one can follow daily in order to gain proficiency in Chef. 

We recommend that you spend one hour a day, getting as far through the kata as you can. After a while, I expect that you'll be able to get through the entire exercise in less than an hour!

Here is an example of a [kata repo](https://github.com/anniehedgpeth/chefkata) that follows these exercises. Each branch is a new day's kata.

## Resources, Recipes, Cookbooks
1. Create a `chef-training` repo on GitHub and clone it locally.

1. Create a new branch using today's date in the name to track it.
1. Ensure that you have a code editor with Chef Plugins installed. I recommend Visual Studio Code.
1. Generate a cookbook into the `chef-training` repo
1. Make your cookbook only support Ubuntu.
1. Set up test kitchen to run the `default` recipe of your cookbook using Vagrant and VirtualBox.
1. Ensure that Nano is installed (in an InSpec test and recipe). Run kitchen converge and verify to ensure this works.
1. For the rest of the lab, create a Test Kitchen workflow that uses the `kitchen create`, `kitchen converge`, `kitchen verify` and `kitchen destroy` commands. Also, use `kitchen login` to manually ssh into your Ubuntu machine.
1. Create `/var/website` directory.
1. Make sure `/var/old-website` directory does not exist.
1. Write a file `/var/website/directions.txt` with text "website goes here" in it.
1. Write a file `builder.txt` to `/var/website/builder.txt` containing the text "[Your Name] built this" where `[Your Name]` is a cookbook attribute with your actual name.
1. Download the Chef logo into `/var/website`: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSgQmQ0CYwU3cpFE6gEB82cp6TSIcBJSisax_HVvEfsgYHGBsO8kQ
1. When you run test kitchen, `builder.txt` should contain the text `Test Kitchen built this`.
1. Run the command `echo ran command > /var/website/command.txt`.
1. Don't run the command the second time chef converges (i.e. make it idempotent).
1. If the command *does* run, do a `git pull` of the architect repository into `/var/website/architect` (https://github.com/pages-themes/architect). It shouldn't pull the repository every time.
1. Refactor your command and pull into a custom resource called `chef_training_website`.
1. Make the git repo that you pull an attribute.
1. Write a `MyLogger` class with a `Log` method that prepends the message `CHEF TRAINING: ` and outputs that to the STDOUT (using puts).

## Chef Server
Now that we've had some practice with basic cookbook development, let's get connected to a Chef Server.

1. Create an Ubuntu virtual machine with VirtualBox.

1. Set up an account on manage.chef.io and ensure your keys and knife.rb are available to your `knife` command on your workstation.
1. Ensure your `chef-training` cookbook is uploaded to the Chef Server
1. Bootstrap your machine running 2 recipes: `chef-training` and the `os-hardening` cookbook. You'll need to ensure the other cookbook is uploaded to the Chef Server as well.
1. Run `chef-client` on the machine again, noticing that 0 resources are converged the second time.

## Search
1. On your workstation, search for all Ubuntu nodes.

1. On your workstation, search for all nodes that match the attribute used to create the `builder.txt` file above.
1. Create a data bag `website` with item `messages`. Inside of `messages`, have a `welcomeMessage` named `Welcome to Chef Learning!`
1. In your `chef-training` cookbook, write a file `/var/website/welcome.txt` with the welcome message from the data bag.
1. Push the updated cookbook to the Chef Server and reconverge, ensuring that the file is there on your Ubuntu VM.
1. Update the data bag to `Welcome to the BEST Chef Learning EVER!`
1. Reconverge and see that the file changed.

## Advanced Administration
1. Show the node's run list with `knife` and look it up in the UI.

1. Create a role named `security` which includes the `os-hardening` cookbook.
1. Read over the README of the `os-hardening` cookbook and find some attributes to set. Set those attributes in your `security` role.
1. Change the run list on the command line to remove the `os-hardening` cookbook and add the `security` role.
1. Reconverge and ensure that the behavior is the same.
1. Create a `development` environment that will be assigned to your existing node. It should:
   - Always run the latest `chef-training` cookbook on the chef server
   - Run the `1.4.1` version of `os-hardening` cookbook
   - Have the `builder.txt` saying `Development Built This`
1. Assign the `development` environment through the `client.rb` on your virtual machine
1. Create another virtual machine that will be your "production" machine
   - Run a specific version of the `chef-training` cookbook
   - The `builder.txt` should say `Production Built This`
   - Assign the `production` environment through the `client.rb`
   - Update your `chef-training` cookbook to change the text of `builder.txt`. After uploading it to the Chef Server notice that only your `development` node was updated, but not your `production` node.

## Chef Automate

For the first badge, you need to understand Automate itself, so this section won't be example driven but more idea driven.

Watch [this video](https://www.youtube.com/watch?v=ldY7KEOxCkM&index=1&list=PL11cZfNdwNyOPa_kLgCX0wDW3O00Sjydx) to get an overview of Chef Automate.

Now do this thought experiment:
* How would you solidify your cookbook deployment workflow? (if you don't know workflow enough, watch [this video](https://www.youtube.com/watch?v=OdoGu31EBU0))
* How would you see what happens on your nodes?
* How would you scan your nodes with inspec profiles?