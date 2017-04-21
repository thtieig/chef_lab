# chef_lab

I have created this repo to store some notes and make easier to rebuild my Chef lab without the need to go through again the [learn Chef website](https://learn.chef.io).

So, what this repo does?


It creates a structure for a Chef lab, including some configuration files.

The lab will be made by:
1. *Chef-server*
2. *node1-centos*
3. *node2-centos*

To use this you need to have the following installed on your machine (I'm running this on Ubuntu 16.04):
1. [Chef DK](https://downloads.chef.io/chefdk)
2. Vagrant - `apt install vagrant`
3. Virtualbox - `apt install virtualbox`

Once all is installed, clone the repository, get into *chef_exercise* folder and run the following:
```
cd chef-server
vagrant up
```
This will take a while and it will create the 3 virtual machine on Virtualbox, headless. Also, it will create a folder called *secrets* where you will find the RSA key for the Chef server.

Once completed, move the *admin.pem* to the *.chef* directory.
```
cp secrets/admin.pem ../.chef/
```

Now, get the SSL certs from the Chef Server:
```
knife ssl fetch
knife ssl verify
knife ssl check
```

You can now use my other repository for testing Chef:
```
cd ../cookbooks/
git clone git@github.com:thtieig/chef_exercise.git
mv chef_exercise/ lamp_centos7
```

Now, you can upload the cookbook to the Chef server:
```
knife cookbook upload lamp_centos7
knife cookbook list
```

And bootstrap the 2 nodes from your workstation using these commands:
```
knife bootstrap localhost --ssh-port 2200 --ssh-user vagrant --sudo --identity-file /home/user/chef_exercise/chef-server/.vagrant/machines/node1-centos/virtualbox/private_key --node-name node1-centos

knife bootstrap localhost --ssh-port 2201 --ssh-user vagrant --sudo --identity-file /home/user/chef_exercise/chef-server/.vagrant/machines/node2-centos/virtualbox/private_key --node-name node2-centos
```
And verify:
```
knife node list

knife node show node1-centos
knife node show node2-centos
```

Time now to upload the roles as well:
```
cd lamp_centos7/
knife role from file roles/web.json
knife role from file roles/db.json
```
...and apply web to node1, and db to node2:
```
knife node run_list set node1-centos "role[web]"
knife node run_list set node2-centos "role[db]"
```

and verify - as usual:
```
knife node show node1-centos
knife node show node2-centos
```
Now that the roles are set, we need to run *chef-client* to actually apply the roles to the nodes:
```
knife ssh localhost --ssh-port 2200 'sudo chef-client' --manual-list --ssh-user vagrant --identity-file /home/user/chef_exercise/chef-server/.vagrant/machines/node1-centos/virtualbox/private_key

knife ssh localhost --ssh-port 2201 'sudo chef-client' --manual-list --ssh-user vagrant --identity-file /home/user/chef_exercise/chef-server/.vagrant/machines/node2-centos/virtualbox/private_key
```

In theory, all should work ;-)

For further information, check  [learn Chef website](https://learn.chef.io) and [my blog post here](https://blog.tian.it/chef-notes/).

Happy Chef'ing!

