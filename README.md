# Using Packer to generate base Vagrant boxes / AWS AMI images

This weekend I decided that I would investigate [HashiCorp's](https://www.hashicorp.com/) [Packer](https://www.terraform.io/).

Downloaded Packer from [here](https://releases.hashicorp.com/packer/0.10.0/packer_0.10.0_linux_amd64.zip).

Extracted and install Packer.

```bash
mkdir packer-bin
cd packer-bin
unzip /tmp/packer_0.10.0_linux_amd64.zip
cd ../
export PATH=$PATH:packer-bin
```

I then followed the (Intro guide)[https://www.packer.io/intro].

The first divergance came from removing the AWS credentials from the example.json file. The files listed looked like the following:

```json
{
  "variables": {
    "aws_access_key": "",
    "aws_secret_key": ""
  },
  "builders": [{
    "type": "amazon-ebs",
    "access_key": "{{user `aws_access_key`}}",
    "secret_key": "{{user `aws_secret_key`}}",
    "region": "us-east-1",
    ...
  }]
}
```

And are executed by `packer build -var 'aws_access_key=YOUR ACCESS KEY' -var 'aws_secret_key=YOUR SECRET KEY' example.json`. This is great, you can pass in the variables on the command line, which at least keeps your secrets from being committed even if they will show in in `ps`...

I use AWS a lot I have the `~/.aws/config` and `~/.aws/credentials` files set up with profiles as documented in the [AWS Documentation](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-multiple-profilesa) and using the `AWS_DEFAULT_PROFILE` evironmental variable. This seems even cleaner to me so instead I decided to use this flavour of configuration.

The thing is the transition to `AWS_DEFAULT_PROFILE` from `AWS_PROFILE`, which is what it was called before is relatively recent and packer still uses `AWS_PROFILE`. In any case I ended up with a configuraiton file that looks like the following instead:

{
  "builders": [{
    "type": "amazon-ebs",
    "region": "us-east-1",
    ...
  }]
}

After making this change a `packer build example.json` just worked! Awesome!

The modifications in the [provision section](https://www.packer.io/intro/getting-started/provision.html) worked exactly like they should. I skipped the [Digital Ocean](https://www.digitalocean.com/) section as right now, being much more interested in the [Vagrant Boxes](https://www.vagrantup.com/).

The [Vagrant](https://www.packer.io/intro/getting-started/vagrant.html) `post-processor` section __seemed__ to work perfectly while building, except it created a Vagrant Box for running on AWS... I am not sure that's exactly what I had in mind :-(

```
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box './packer_amazon-ebs_aws.box' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Box file was not detected as metadata. Adding it directly...
==> default: Adding box './packer_amazon-ebs_aws.box' (v0) for provider: virtualbox
    default: Unpacking necessary files from: file:///home/fozz/Projects/hashicorp-packer-1/packer_amazon-ebs_aws.box
    default: Progress: 0% (Rate: 0/s, Estimated time remaining: --:--:--The box you attempted to add doesn't match the provider you specified.

Provider expected: virtualbox
Provider of box: aws
```

Firstly I tried and failed to create a `*.ova` file in VirtualBox using the `File` __>__ `Export Appliance` which took ages and in the end Packer could not connect to it via SSH. Not sure what i did wrong but as it took so long I didn't want to try again.

Instead I found these two magical lines at [https://github.com/flynn/flynn/tree/master/util/packer](https://github.com/flynn/flynn/tree/master/util/packer):

```
$ BOX_URL=https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box
$ curl $BOX_URL | tar --delete Vagrantfile > ubuntu.ova
```

So I ran them and they did output the ubuntu.ova file! Using the modifications in the [./example-basic-provisioner-vagrant.json](./example-basic-provisioner-vagrant.json) ended up creating a file named [./packer_virtualbox-ovf_virtualbox.box](`./packer_virtualbox-ovf_virtualbox.box`) which ran perfectly with [this Vagrantfile](./Vagrantfile). So eventually I have a somewhat working process for outputting Vagrant box files, which makes it a pretty successful weekend project.
