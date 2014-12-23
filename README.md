tiered-aws_ha_chef Cookbook
===========================
This cookbook will install and configure a high-availability Chef server 
cluster with two back end servers and and multiple front-end servers.

NOTE: the recipes contained in this cookbook are NOT all idempotent, nor
should you be running the Chef client on your Chef server(s). Use it as a
one-off setup tool for building new Chef server clusters, or for restoring
after a failure.

Requirements
------------
TODO:  Add requirements here.

+ VPC with subnets in desired availability zones
+ Security groups for load balancer, front end, back end and VPC default
+ Internet gateway on Chef VPC network, so subnets can reach the internet
+ EBS storage device, recommended size 100GB and provisioned 3000 IOPS
+ IAM account with correct rights; see installation guide
+ Two back end instances, two or more front-end instances
  - Back end servers must both be in the same availability zone
  - Back end EBS storage must be in the same AZ as the back end servers
  - Front end servers can be in any availability zone in the same region
  - You must create VPC subnets in any AZ where you want to put servers
+ Either an internal or external load balancer to expose the Chef API
+ OPTIONAL: Routes and/or peering connections for any internal networks


Attributes
----------

```
node['aws_ha_chef']['api_fqdn']                 FQDN of your Amazon elastic load balancer

node['aws_ha_chef']['backend1']['fqdn']         FQDN of the primary back end
node['aws_ha_chef']['backend1']['ip_address']   IP address of the primary back end
node['aws_ha_chef']['backend2']['fqdn']         FQDN of the secondary back end 
node['aws_ha_chef']['backend2']['ip_address']   IP address of the secondary back end

node['aws_ha_chef']['frontends']['fe1']['fqdn']        FQDN of front end 1
node['aws_ha_chef']['frontends']['fe1']['ip_address']  IP address of front end 1
node['aws_ha_chef']['frontends']['fe2']['fqdn']        FQDN of front end 2
node['aws_ha_chef']['frontends']['fe2']['ip_address']  IP address of front end 2

# Want more frontends? Add as many as you need here:
# node['aws_ha_chef']['frontends']['fe3']['fqdn']        FQDN of front end 3
# node['aws_ha_chef']['frontends']['fe3']['ip_address']  IP address of front end 3
# node['aws_ha_chef']['frontends']['fe4']['fqdn']        FQDN of front end 4
# node['aws_ha_chef']['frontends']['fe4']['ip_address']  IP address of front end 4
```


```

Usage
-----

Usage is fairly simple and straightforward.  First configure all the attributes listed above via a role, *.json file, or .kitchen.yml file. Look in the included .kitchen.yml file to see which order the recipes are in. Note also that you will need to use the 'hostsfile' recipe for local testing if you don't have 'real' DNS for your test machines.  Also note that if you are installing the reporting or push_jobs addons, you must build and configure your frontend servers first, then configure the backend server last.  This is because the backend server generates certs and other config files that get pushed up to the frontend servers at the end of the backend.rb recipe. 

The NTP recipe is only required if you don't already have a way to configure NTP.

A typical run list for a frontend server might look like this:
      - recipe[tiered-aws_ha_chef::configfile]
      - recipe[tiered-aws_ha_chef::ntp]
      - recipe[tiered-aws_ha_chef::server]
      - recipe[tiered-aws_ha_chef::manage]
      - recipe[tiered-aws_ha_chef::reporting]
      - recipe[tiered-aws_ha_chef::push_jobs]
      - recipe[tiered-aws_ha_chef::frontend]

And a typical run list for a backend server might look like this:
      - recipe[tiered-aws_ha_chef::configfile]
      - recipe[tiered-aws_ha_chef::ntp]
      - recipe[tiered-aws_ha_chef::server]
      - recipe[tiered-aws_ha_chef::reporting]
      - recipe[tiered-aws_ha_chef::push_jobs]
      - recipe[tiered-aws_ha_chef::backend]
