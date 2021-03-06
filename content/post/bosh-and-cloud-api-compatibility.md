+++
title = "bosh and cloud api compatibility"
date = "2013-08-02"
Categories = ["cloudfoundry", "cloud", "bosh", "amazon", "aws", "openstack"]
+++

The gauntlet has again been dropped in the world of cloud interoperability. The dueling factions include those asserting that competitors to Amazon's web services (principally OpenStack) must adopt AWS's API's in order to remain viable, and those that believe such "API cloning" will do nothing more than stunt innovation. If you were to ask me, I'd say that we've seen this play out before. Remember the "Clone Wars" that began in the late 1980's and that persisted for the better part of two decades? A huge cast of competitors battled for the title of "best PC that's not manufactured by IBM." How did that play out? For a relatively short period of time, having the best PC "designed for Microsoft Windows," along with the leanest supply chain (see Dell), paved a golden path to victory. And then Steve Jobs returns to Apple, and now better than 50% of the laptops running in the Starbucks in which I'm writing this blog have a shiny white fruit on their lids. As it turns out, "going your own way" can work out awfully well.

But that's not the angle I want to take in this discussion. Let's dig deeper into what the two sides have to say.

The battle was first renewed with Cloud Scaling CTO Randy Bias' [Open Letter to the OpenStack Community](http://www.cloudscaling.com/blog/cloud-computing/openstack-aws). Randy adopts the position that full-compatibility with the AWS API's is necessary for OpenStack's survival. The gist of his argument is that Amazon currently dominates public cloud, supporting this via a comparison between Amazon's and Rackspace's growth rates since 2009, and that they also "control the innovation curve" as they push "new features into production at a breathtaking pace." Furthermore, he asserts that any hope for survival with respect to competing cloud platforms is limited to the hybrid cloud space, providing enterprises with the capability to seamlessly migrate workloads between the public cloud and private, on-premises clouds. Therefore, OpenStack must adopt API compatibility with AWS in order to become the enterprise choice for hybrid cloud.

A few days later, Rackspace's "Startup Liaison Officer" Robert Scoble responded with his own [Open Letter](https://plus.google.com/+Scobleizer/posts/HQ7Wi4WCQse). Scoble makes some interesting counterpoints, most notably the argument that customers don't adopt cloud platforms because of API compatibility with Amazon, but because of the promise of a "10x improvement" to their own business. In order to provide such improvements, cloud platform competitors must not shackle themselves to a "de facto standard" API, but rather must focus their limited resources on driving those 10x improvements in infrastructure capability.

So by now you must be wondering, whose side am I on? I'm on the side of innovation. But that doesn't necessarily put me in either camp. I think the end goals of both parties are things that we want:

- **Freedom:** the ability to migrate workloads between cloud infrastructure providers without needing to significantly alter the behavior of the workload itself.
- **Innovation:** the ability to realize capabilities that don't exist today that will solve emerging problems (particularly those related to the explosion of generated and archived data).

Spending development cycles on API compatibility will certainly slow anyone's ability to innovate. And what is API compatibility anyway? I believe that much of the concern rests on the large investment many enterprises have (or believe they will need to create) in bespoke automation written to a particular vendor's API. Having recently left a large-scale project that generated thousands of lines of such automation to drive consumption of a particular vendor's infrastucture platform, and that was in the near term planning to migrate to another platform, I can tell you that this concern is very real. But simply stating that "your existing code will work when you target our API" does not compatibility make. As Amazon continues to deploy new features at their breathtaking pace, how will OpenStack and other platforms keep up?

For API compatibility to be *real*, a "technology compatibility kit" (TCK) is needed. For those in the Java world, TCK's are near and dear. Java itself is not a particular implementation, but a standard API that invites competing implementations and innovation. But for any competing implementation to call itself "Java," it must pass the tests contained within the TCK. An AWS TCK is really the only true way to ensure API compatibility. But I think it's hard to argue that Amazon has any real business interest in creating and sharing one. 

There is another way. Perhaps we should stop creating bespoke automation and rally around a common standard toolkit for managing large-scale cloud application deployments. This toolkit could provide mechanisms for configuration management, orchestration, health management, and rolling upgrades. It could further, as part of its architecture, build an adapter layer between its core components and the underlying infrastructure provider. Plugins could then be developed to provide the toolkit with the ability to manage all of the common infrastructure providers.

Enter BOSH and it's Cloud Provider Interface (CPI) layer. BOSH was initially developed as the means of deploying and managing the Cloud Foundry PaaS platform, but it's much more generally applicable. BOSH can today deploy any distributed system, *unchanged*, to any of several popular IaaS providers: VMware vSphere, VMware vCloud Director, Amazon Web Services, and OpenStack. Heresy you say! Not so. Just ask Colin Humphreys of CloudCredo, who recently [described their wildly successful deployment](http://blog.cloudfoundry.com/2013/04/30/uk-charity-raises-record-donations-powered-by-cloud-foundry) of Cloud Foundry to a hybrid composition of vSphere and AWS-based clouds. He recently presented a technical deep dive in Pivotal's offices in which he made the statement (paraphrasing) "I took the same Cloud Foundry bits that were running on AWS and deployed them unchanged to vSphere using BOSH." As anyone can tell, this isn't just theory, it's production.

So how then does BOSH make this happen? A trip [into the code](https://github.com/cloudfoundry/bosh/blob/master/bosh_cpi/lib/cloud.rb) for the BOSH CPI "interface" will show a list of core infrastructure capabilities that BOSH requires:

- `current_vm_id`
- `create_stemcell`
- `delete_stemcell`
- `create_vm`
- `delete_vm`
- `has_vm?`
- `reboot_vm`
- `set_vm_metadata`
- `configure_networks`
- `create_disk`
- `delete_disk`
- `attach_disk`
- `snapshot_disk`
- `delete_snapshot`
- `detach_disk`
- `get_disks`

All interactions between BOSH and the underlying infrastructure provider pass through these core methods. As long as a CPI exists that exposes these capabilities to BOSH, BOSH can deploy and manage the lifecycle of Cloud Foundry (or any other distributed system described by a BOSH release) on an infrastructure provider.

So how hard is it to provide the CPI's for both AWS and OpenStack? If you choose simple metrics like number of classes (NOC) and lines of code (LOC), not that hard.

You can find the CPI implementations at these links:

- [https://github.com/cloudfoundry/bosh/tree/master/bosh_aws_cpi](https://github.com/cloudfoundry/bosh/tree/master/bosh_aws_cpi)
- [https://github.com/cloudfoundry/bosh/tree/master/bosh_openstack_cpi](https://github.com/cloudfoundry/bosh/tree/master/bosh_openstack_cpi)

First we'll generate the metrics for AWS:

``` bash
$ find ./bosh_aws_cpi/lib -name "*.rb" -exec wc -l {} \;
       2 ./bosh_aws_cpi/lib/bosh_aws_cpi.rb
      68 ./bosh_aws_cpi/lib/cloud/aws/aki_picker.rb
      39 ./bosh_aws_cpi/lib/cloud/aws/availability_zone_selector.rb
     651 ./bosh_aws_cpi/lib/cloud/aws/cloud.rb
      22 ./bosh_aws_cpi/lib/cloud/aws/dynamic_network.rb
      30 ./bosh_aws_cpi/lib/cloud/aws/helpers.rb
     171 ./bosh_aws_cpi/lib/cloud/aws/instance_manager.rb
      25 ./bosh_aws_cpi/lib/cloud/aws/manual_network.rb
      37 ./bosh_aws_cpi/lib/cloud/aws/network.rb
      89 ./bosh_aws_cpi/lib/cloud/aws/network_configurator.rb
     189 ./bosh_aws_cpi/lib/cloud/aws/resource_wait.rb
      68 ./bosh_aws_cpi/lib/cloud/aws/stemcell.rb
     114 ./bosh_aws_cpi/lib/cloud/aws/stemcell_creator.rb
      30 ./bosh_aws_cpi/lib/cloud/aws/tag_manager.rb
       7 ./bosh_aws_cpi/lib/cloud/aws/version.rb
      44 ./bosh_aws_cpi/lib/cloud/aws/vip_network.rb
      43 ./bosh_aws_cpi/lib/cloud/aws.rb
```
We'll also want the total LOC:

``` bash
$ find ./bosh_aws_cpi/lib -name "*.rb" -exec wc -l {} \; | awk '{ sum += $1 } END { print sum }'
1629
```

And now let's do the same for OpenStack:

``` bash
$ find ./bosh_openstack_cpi/lib -name "*.rb" -exec wc -l {} \;
       4 ./bosh_openstack_cpi/lib/bosh_openstack_cpi.rb
     867 ./bosh_openstack_cpi/lib/cloud/openstack/cloud.rb
      28 ./bosh_openstack_cpi/lib/cloud/openstack/dynamic_network.rb
     131 ./bosh_openstack_cpi/lib/cloud/openstack/helpers.rb
      34 ./bosh_openstack_cpi/lib/cloud/openstack/manual_network.rb
      37 ./bosh_openstack_cpi/lib/cloud/openstack/network.rb
     159 ./bosh_openstack_cpi/lib/cloud/openstack/network_configurator.rb
      16 ./bosh_openstack_cpi/lib/cloud/openstack/tag_manager.rb
       8 ./bosh_openstack_cpi/lib/cloud/openstack/version.rb
      50 ./bosh_openstack_cpi/lib/cloud/openstack/vip_network.rb
      39 ./bosh_openstack_cpi/lib/cloud/openstack.rb
$ find ./bosh_openstack_cpi/lib -name "*.rb" -exec wc -l {} \; | awk '{ sum += $1 } END { print sum }'
1373
```

So, summarizing:

CPI        | Number of Classes (NOC) | Lines of Code (LOC)
---------- | -----------------------: | -------------------:
Amazon AWS | 17                      | 1629
OpenStack  | 11                      | 1373
<br/>

Let's make a couple of points about these metrics. First of all, the two CPI's do not use a common foundation. The AWS CPI uses the [AWS SDK for Ruby](http://aws.amazon.com/sdkforruby) while the OpenStack CPI uses [Fog](http://fog.io). Fog could also have been used as the foundation for the AWS CPI, but the CPI authors presumably thought it better to stick with the toolkit provided by Amazon. This is a minor point, however, as both of these toolkits essentially provide simple wrappers around the infrastructure providers' REST API's. It's doubtful that using a common API wrapper for both CPI's would have substantially changed the metrics presented here.

Second, obviously NOC and LOC are rather naive metrics. It's incredibly possible to write terse code that is opaque, buggy, and hard to maintain or enhance. In fact, according to Code Climate, both of the top-level implementation classes for these CPI's have quite a lot of room for improvement:

* [https://codeclimate.com/github/cloudfoundry/bosh/Bosh::AwsCloud::Cloud](https://codeclimate.com/github/cloudfoundry/bosh/Bosh::AwsCloud::Cloud)
* [https://codeclimate.com/github/cloudfoundry/bosh/Bosh::OpenStackCloud::Cloud](https://codeclimate.com/github/cloudfoundry/bosh/Bosh::OpenStackCloud::Cloud)

With that said, it is rather amazing that one could encapuslate all of the infrastructure-specific implementation necessary to deploy and manage a distributed system as powerful as Cloud Foundry in *less than twenty classes and 1700 lines of code*.

So, to summarize where we've been, while there's an impressive war of words out there regarding API compatibility with Amazon AWS, Cloud Foundry and BOSH don't necessarily need to take sides. If OpenStack chooses to embrace the AWS API's, the BOSH AWS CPI will be there waiting. However, if OpenStack chooses to follow in the footsteps of Apple and take the road less traveled, the OpenStack CPI is ready and waiting to evolve with it. Should Google Compute Engine or Microsoft's Azure gain a foodhold on the innovation curve, they are presumably a relatively simple CPI away from joining the BOSH ecosystem. So if you really want "cloud freedom," BOSH is leading the charge.
