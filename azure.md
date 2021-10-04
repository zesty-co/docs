# Zesty Disk installation on Azure

## Azure scale set
Azure scale set offers a way to quickly scale out a group of VMs by duplicating them based on different metrics.

There are three main approaches when it comes to integrating a scale set with Zesty disk:

## 1. Use user-data
Update the existing [user data](https://docs.microsoft.com/en-us/azure/virtual-machines/user-data) for a VMSS with [`az vmss update`](https://docs.microsoft.com/en-us/cli/azure/vmss?view=azure-cli-latest#az_vmss_update):
```bash
az vmss update ... --user-data "..."
```
Note(!) - user data should be **base64 encoded**

## 2. Use Cloud-Init
Using a [cloud-init](https://cloudinit.readthedocs.io/en/latest/) script to run on every new VM installing the Zesty collector.
In order to utilize cloud-init to pre-configure a VM with Zesty, the image has to be cloud-init-enabled and [deployed as such](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-cloud-init#deploying-a-cloud-init-enabled-virtual-machine).
Add to your `cloud-init.txt` the next `runcmd` command to install the zesty collector:
```yaml
runcmd:
  - curl -s https://static.zesty.co/ZX-InfraStructure-Agent-release/install.sh | sudo bash -s apikey=xxxxx
```

#### Updating a scale set custom data on Azure
This can only be done through the Azure API:
> "you can update VMSS custom data via REST API (not applicable for PS or AZ CLI clients)..." - [Azure Docs](https://docs.microsoft.com/en-us/azure/virtual-machines/custom-data#can-i-update-custom-data-after-the-vm-has-been-created)

Note(!) - When you update custom data in the VMSS model:

- Existing instances in the VMSS will not get the updated custom data, only until they are reimaged.
- Existing instances in the VMSS that are upgraded will not get the updated custom data.
- New instances will receive the new custom data.
- Custom data should be **base64 encoded**

The update operation is as follows:
```
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2021-07-01
```
[Further documentation](https://docs.microsoft.com/en-us/rest/api/compute/virtual-machine-scale-sets/update)

#### Creating a scale set with custom-data
A simpler operation is the creation of a brand new VMSS where the creation can be done right from the Azure CLI like so:
```bash
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.txt \
  --admin-username azureuser \
  --generate-ssh-keys
```

---

## 3. Use an external configuration management system to pre install the collector
CM systems can be [Chef](), [Puppet](), [Ansible]() and others. Below you can find examples for all three named systems

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

---

## 4. Create an image and update the VMSS
Another approach to automating the Zesty collector on a VMSS is creating an image that's pre-installed with the collector, then updating the scale set to use the new image.
Note that this opreation like updating custom-data above, will not roll the currently running VMs and they would have to be updated, either by installing the collector manually, or by being shut-down.
In order to create a new image:
Use one of the running instances and install the collector by running:
```bash
curl -s https://static.zesty.co/ZX-InfraStructure-Agent-release/install.sh | sudo bash -s apikey=xxxxx
```

Once the collector has been successfully installed, create an image:
1. Create an [image gallery](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/tutorial-use-custom-image-cli#create-an-image-gallery):
```
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```
Follow up by create an [image definition](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/tutorial-use-custom-image-cli#create-an-image-gallery) and an [image version](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/tutorial-use-custom-image-cli#create-an-image-gallery).

2. Next, bring the scale set [up to date](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) with the new image:
**Manually** by updating running instances with:
```bash
az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
```

