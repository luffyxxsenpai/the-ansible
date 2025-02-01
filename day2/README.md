# ansible-day2

# dynamic inventories
- as we talked about inventories in day1, dynamic inventories are used to automatically create the inventory file
- now we can do this in many cloud providers, here we will do with AWS EC2
- so lets see how we can do this -__-

### prequisites
- you need to have python3 and boto3 (pip install boto3)
- create a aws user with ec2 admin acess 
- install aws-cli and setup your access and secret keys
- `pip show boto3` to confirm your boto3 installation
- `ansible-galaxy collection install amazon.aws` to install the aws collection
- in your ansible config under inventory section add this
    - `enable_plugins = host_list, virtualbox, yaml, constructed, aws_ec2, ini, auto, toml`
    - in this we have enabled the aws and yaml support
    - you can run `ansible --version` and see where your ansible config is located
    - if theres no config, then either create it in `/etc/ansible/ansible.cfg` or in your current ansible working directory

- python3, boto3, aws-cli and an aws user with enough permission is all we need


## how to dynamic inv
- so first, we create a inventory file in yaml, we have to make sure name of file ends with aws_ec2.yml or aws_ec2.yaml
- first we define the plugin we are gonna use
- we can use access key and secret key but when we run `aws configure` it creates `~/.aws/credentials` which already have keys
- so its safe to use the profile opotion which by default is default. cat out the `~.aws/credentials` 
- we can lock the region or it takes a bit time to scan all the regions, so if your regions are fixed then you can fix it here too
- `hostnames` give us the details of ec2 ip. for public ip we use `ip-address` and for dns we can use `dns-name`
- we create groups ofc to group the servers. we have two ways `groups` and `keyed_groups`

- read the doc : 'https://docs.ansible.com/ansible/latest/collections/amazon/aws/docsite/aws_ec2_guide.html'
```yaml
plugin: aws_ec2
aws_profile: default
regions:
  - ap-south-1
hostnames:
  - ip-address

groups:
  redhat: "'amazon-linux' in tags.distro"
  ubuntu: "'ubuntu' in tags.distro"


keyed_groups:
  - prefix: arch
    key: architecture
```
