# Zesty Disk Installation
Zesty Disk installation involves two logical steps:
1. Install the `ZestyDisk` agent so that the system can gain visibility over disks and I/O activity
2. Create a `ZestyDisk` volume that's identify-able by the system and therefor can start being automatically managed

Let's quickly go through both steps:

### ZestyDisk Agent
In order to install the agent on any machine, simply run:
```bash
curl -s https://static.zesty.co/ZX-InfraStructure-Agent-release/install.sh | sudo bash -s apikey=xxxxx
```
* Note that the installation requires an `apikey` for the volume assignment
* If you don't have the key already saved, get it from: [Zesty app](https://app.zesty.co) --> Pick "Zesty Disk" on the left hand-side menu --> Hit "Install Collector"

---

### ZestyDisk Volume
A volume would be identified and automatically managed based on its [snapshot](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html) ID. This is nothing but a logical identifier with no additional dependencies.

The list of snapshot-ids for `ZestyDisk`
```json
  snapshot_ids = {
    af-south-1     = "snap-0e2cb5c972c820f07"
    eu-north-1     = "snap-0ff78c28ef9594115"
    ap-south-1     = "snap-0fb2053077c169af7"
    eu-west-3      = "snap-0465d456347efae6c"
    eu-west-2      = "snap-0f19038a86bde248f"
    eu-south-1     = "snap-078edd8014efe790a"
    eu-west-1      = "snap-02ff3ee6bc8a91872"
    ap-northeast-2 = "snap-08d18682aaeaf33be"
    me-south-1     = "snap-087bd48f60e1122ce"
    ap-northeast-1 = "snap-0bbfd407d10372b58"
    sa-east-1      = "snap-033c6ad75f5c89b46"
    ca-central-1   = "snap-02600418e30fb5849"
    ap-east-1      = "snap-01794850217168ace"
    ap-southeast-1 = "snap-05fb9c3e745c46ed9"
    ap-southeast-2 = "snap-0bf3208e700b734a7"
    eu-central-1   = "snap-0c334e7bfb7a8a31a"
    us-east-1      = "snap-0712202f76b8585c4"
    us-east-2      = "snap-099178a8d55156d8c"
    us-west-1      = "snap-019c04f137eb4c2d9"
    us-west-2      = "snap-0fdd4d38f36be4da0"
  }
  ```

* Another way of fetching the latest snaphot-id for a specific region can be done by searching for `Public Snapshots`, where `Description: Zesty Disk` **and** `Owner: 297421580991`
* Here's the CLI equivalent of running the same query:
```bash
aws ec2 describe-snapshots \
  --owner-ids 297421580991 \
  --filters Name=description,Values="Zesty Disk"
```

---

# Mass Installations
In order to mass install Zesty Disk agents across a logical set of resources, an organization can utilize their configuration management system of choice, such as Puppet, Chef, Ansible, or, leverage the use of an Infrastructure as Code tool like Cloudformation or Terraform.

### Ansible
```yaml
- name: Install ZestyDisk Agent
  ansible.builtin.shell: curl -s https://static.zesty.co/ZX-InfraStructure-Agent-release/install.sh | sudo bash -s apikey=xxxxx
```

### Chef
```ruby
bash 'extract_module' do
  cwd ::File.dirname(src_filepath)
  code <<-EOH
    curl -s https://static.zesty.co/ZX-InfraStructure-Agent-release/install.sh | sudo bash -s apikey=xxxxx
  EOH
  not_if { ::File.exist?(extract_path) }
end
```

### Puppet
```ruby
exec {
  'ZestyDiskInstallation':
    command => 'curl -s https://static.zesty.co/ZX-InfraStructure-Agent-release/install.sh | sudo bash -s apikey=xxxxx
',
    refreshonly => true
}
```

* Here's the [Zesty Terraform module ](https://github.com/zesty-co/terraform-zesty-disk-config) implementing both the agent installation, together with the provisioning of a ZestyDisk-ready volume

---

## AWS Mass installation with Systems Manager
* If systems manager is already in use, the [`run command`](https://docs.aws.amazon.com/systems-manager/latest/userguide/execute-remote-commands.html) can be utilized to deploy the agent across a subset of instances
* If SSM isn't yet implemented here's [a quick, five-steps tutorial](https://aws.amazon.com/getting-started/hands-on/remotely-run-commands-ec2-instance-systems-manager/) to quickly configure the system and run a command
