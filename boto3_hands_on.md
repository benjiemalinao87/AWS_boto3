## boto3 quick hands-on

This documentation aims at being a _quick-straight-to-the-point-hands-on AWS_
resources manipulation with [boto3][0].

First of all, you'll need to install [boto3][0]. Installing it along with
[awscli][1] is probably a good idea as

- [awscli][1] is _boto-based_
- [awscli][1] usage is really close to _boto_'s
- [boto3][0] will use the same configuration files

A convenient method consists in installing them in a `python` [virtualenv][2]:

```
$ virtualenv awscli
$ . awscli/bin/activate
(awscli)$ pip install awscli boto3
```

From now on I will assume you already have an _AWS_ account setup, best
practices suggest that you must NOT use your master account but instead create
an account which might have adequate privileges.

```
$ aws configure
AWS Access Key ID [None]: your_key_id
AWS Secret Access Key [None]: your_access_key
Default region name [None]: eu-west-1
Default output format [None]: json
```

In order switch between regions more easily, I suggest you make aliases on the
`awscli` configuration files:

```
$ cat ~/.aws/credentials 
[default]
aws_access_key_id = your_key_id
aws_secret_access_key = your_access_key

[ireland]
aws_access_key_id = your_key_id
aws_secret_access_key = your_access_key

$ cat ~/.aws/config
[default]
output = json
region = eu-west-1

[profile ireland]
output=json
region=eu-west-1
```

You should then be able to summon `awscli` by region:

```
$ aws ec2 describe-instances --profile ireland
{
    [some json output]
}
```

We are now able to dig around with `boto3` itself. In order to learn its usage,
we'll use the fantastic [ipython][3]. First import `boto3`:

```
$ ipython3
In [1]: import boto3
```

Then initiate a `boto` session:

```
In [2]: s = boto3.Session(profile_name='ireland')

```

From there, initiate an object corresponding to the resource you want to
manipulate, for instance `ec2`:

```
In [3]: ec2 = s.resource('ec2')
```

And witness the methods at your disposal:

```python
In [4]: ec2.[press tab]
ec2.DhcpOptions                    ec2.create_snapshot
ec2.Image                          ec2.create_subnet
ec2.Instance                       ec2.create_tags
ec2.InternetGateway                ec2.create_volume
ec2.KeyPair                        ec2.create_vpc
ec2.NetworkAcl                     ec2.create_vpc_peering_connection
ec2.NetworkInterface               ec2.dhcp_options_sets
ec2.PlacementGroup                 ec2.disassociate_route_table
ec2.RouteTable                     ec2.images
ec2.RouteTableAssociation          ec2.import_key_pair
ec2.SecurityGroup                  ec2.instances
ec2.Snapshot                       ec2.internet_gateways
ec2.Subnet                         ec2.key_pairs
ec2.Tag                            ec2.meta
ec2.Volume                         ec2.network_acls
ec2.Vpc                            ec2.network_interfaces
ec2.VpcPeeringConnection           ec2.placement_groups
ec2.create_dhcp_options            ec2.register_image
ec2.create_instances               ec2.route_tables
ec2.create_internet_gateway        ec2.security_groups
ec2.create_key_pair                ec2.snapshots
ec2.create_network_acl             ec2.subnets
ec2.create_network_interface       ec2.volumes
ec2.create_placement_group         ec2.vpc_peering_connections
ec2.create_route_table             ec2.vpcs
ec2.create_security_group          
```

While those methods permit to execute operations, they also give access to
objects themselves, for example:

```
In [5]: for i in ec2.instances.all(): print(i)
ec2.Instance(id='i-ea39b240')
ec2.Instance(id='i-54c4fafe')
ec2.Instance(id='i-0d0533a7')
ec2.Instance(id='i-44c5fbee')
ec2.Instance(id='i-9405333e')
ec2.Instance(id='i-b105331b')
ec2.Instance(id='i-04fc68ae')
```

Let's pick one of those:

```
In [6]: i = ec2.Instance(id='i-ea39b240')
```

And discover its methods:

```
In [7]: i.[press tab]
i.ami_launch_index          i.private_ip_address
i.architecture              i.product_codes
i.attach_classic_link_vpc   i.public_dns_name
i.attach_volume             i.public_ip_address
i.block_device_mappings     i.ramdisk_id
i.client_token              i.reboot
i.console_output            i.reload
i.create_image              i.report_status
i.create_tags               i.reset_attribute
i.describe_attribute        i.reset_kernel
i.detach_classic_link_vpc   i.reset_ramdisk
i.detach_volume             i.reset_source_dest_check
i.ebs_optimized             i.root_device_name
i.hypervisor                i.root_device_type
i.iam_instance_profile      i.security_groups
i.id                        i.source_dest_check
i.image                     i.spot_instance_request_id
i.image_id                  i.sriov_net_support
i.instance_id               i.start
i.instance_lifecycle        i.state
i.instance_type             i.state_reason
i.kernel_id                 i.state_transition_reason
i.key_name                  i.stop
i.key_pair                  i.subnet
i.launch_time               i.subnet_id
i.load                      i.tags
i.meta                      i.terminate
i.modify_attribute          i.unmonitor
i.monitor                   i.virtualization_type
i.monitoring                i.volumes
i.network_interfaces        i.vpc
i.password_data             i.vpc_id
i.placement                 i.wait_until_exists
i.placement_group           i.wait_until_running
i.platform                  i.wait_until_stopped
i.private_dns_name          i.wait_until_terminated
```

For example:

```
In [8]: i.hypervisor
Out[8]: 'xen'
```

Here's an example of the `filter` method that many objects use to match only
certain criterias:

```
In [9]: filter = {'Name': 'name', 'Values' : ['debian*amd64*ebs']}
In [10]: for i in ec2.images.filter(Filters = [filter]): print(i)
ec2.Image(id='ami-61e56916')
ec2.Image(id='ami-879e4ff0')
ec2.Image(id='ami-8bf29ffc')
ec2.Image(id='ami-971a65e0')
ec2.Image(id='ami-99f39eee')
ec2.Image(id='ami-c935cbbe')
ec2.Image(id='ami-e31a6594')
ec2.Image(id='ami-e7e66a90')
```

Accessing services through `resource` is called the _high level_ method, and not
everything is yet reachable through this consistent interface. The other,
_low-level_ method is `client`:

```
In [11]: ec2c = s.client('ec2')
In [12]: ec2c.[press tab]
Display all 188 possibilities? (y or n)
ec2c.accept_vpc_peering_connection
ec2c.allocate_address
ec2c.assign_private_ip_addresses
ec2c.associate_address
ec2c.associate_dhcp_options
ec2c.associate_route_table
ec2c.attach_classic_link_vpc
ec2c.attach_internet_gateway
ec2c.attach_network_interface
ec2c.attach_volume
ec2c.attach_vpn_gateway
ec2c.authorize_security_group_egress
ec2c.authorize_security_group_ingress
ec2c.bundle_instance
ec2c.can_paginate
[...]
```

As a usage example, no high-level resource is available to list a region
Availability Zones, but a `client` can achieve this:

```
In [13]: ec2c.describe_availability_zones()
Out[13]:
{'AvailabilityZones': [{'Messages': [],
   'RegionName': 'eu-west-1',
   'State': 'available',
   'ZoneName': 'eu-west-1a'},
  {'Messages': [],
   'RegionName': 'eu-west-1',
   'State': 'available',
   'ZoneName': 'eu-west-1b'},
  {'Messages': [],
   'RegionName': 'eu-west-1',
   'State': 'available',
   'ZoneName': 'eu-west-1c'}],
 'ResponseMetadata': {'HTTPStatusCode': 200,
  'RequestId': '46e474a5-7e77-4716-a9a1-010990d066ee'}}
```

Every single `resource` and `client` methods are documented in
[boto3's documentation][4].

Last but not least, I wrote a convenient little module in order to ease `boto3`
usage. It is available in [my GitHub repository][5].

Its usage is pretty straightforward:

```
In [14]: from session import Aws
In [15]: ec2 = Aws('ireland', 'ec2')
```

The instanciated `ec2` objects holds both `resource` and `client` methods along
with helper functions and variables:

```
In [16]: ec2.[press tab]
ec2.change_nsrecord      ec2.getamis              ec2.profile
ec2.client               ec2.getinst              ec2.region
ec2.create_tag           ec2.gettagval            ec2.resource
ec2.dmesg                ec2.lsinstances          ec2.session
ec2.get_id_from_nametag  ec2.lsinstnames          ec2.tags2dict
ec2.getall               ec2.mktags               
ec2.getami               ec2.mkuserdata
```

Every helper is nicely documented:

```
In [17]: ec2.change_nsrecord??
[...]
    def change_nsrecord(self, action, dnsrecord):
        '''Create, delete or modify a DNS record

        :param str action: One of ``CREATE``, ``DELETE`` or ``UPSERT``
        :param dict dnsrecord: A dict describing the DNS record to change
[...]
```

And uses `boto3` functions behind the scenes:

```
In [18]: ec2.dmesg('i-ea39b240').split('\n')[0]
Out[18]: u'ian.net/debian/ wheezy/main debconf-utils all 1.5.49 [55.8 kB]\r'
```

Let's finish this quick hands on with a very basic _EC2_ instance creation and
termination using `boto3`:

```
In [19]: rc = ec2.resource.create_instances(
    ImageId = ec2.getami('NetBSD*64*6.1.5*'),
    MinCount = 1,
    MaxCount = 1,
    KeyName = 'mysshpemkey',
    InstanceType = 'm3.medium',
    PrivateIpAddress = '10.10.0.1',
    SubnetId = ec2.get_id_from_nametag('subnets', 'examplesubnet')
)
In [20]: print(rc[0].id)
i-b1774f1b
In [21]: rc[0].terminate()
Out[21]: 
{'ResponseMetadata': {'HTTPStatusCode': 200,
  'RequestId': '45738798-812c-4ea4-9c15-edb7ddcd9950'},
 u'TerminatingInstances': [{u'CurrentState': {u'Code': 32,
    u'Name': 'shutting-down'},
   u'InstanceId': 'i-b1774f1b',
   u'PreviousState': {u'Code': 16, u'Name': 'running'}}]}

```

In this example, we retrieve the _AMI_ id through my module's `getami()`
function, and the `SubnetId` using my helper function `get_id_from_nametag()`
which supposes that you gave a tag, `examplesubnet` here, to the subnet you
intend to spawn this instance in.

The `create_instances` function returns an array of instances objects that are
immediately ready for usage, and as a matter of fact, in this example, we show
the instance `id` and terminate it.

Hope that quick hands on has been informative, do not hesitate to comment and
share it!

Emile `iMil` Heitor -- imil@NetBSD.org

[0]: https://github.com/boto/boto3
[1]: http://aws.amazon.com/cli/
[2]: https://virtualenv.pypa.io/en/latest/
[3]: http://ipython.org/
[4]: http://boto3.readthedocs.org/en/latest/
[5]: https://github.com/iMilnb/awstools/blob/master/mods/session.py
