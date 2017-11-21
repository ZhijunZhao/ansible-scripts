# How to use Dynamic Inventory for Azure with Ansible

If you don't know the Dynamic Inventory concept, checkout out [Intro to Dynamic Inventory](http://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html).

## Prerequisites

- Ansible and the Azure Python SDK modules installed on your host system.

    ```bash
    pip install ansible[azure]
    ```

- Azure credentials, and Ansible configured to use them.
    - [Create Azure credentials and configure Ansible](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/ansible-install-configure#create-azure-credentials)


## Dynamic Inventory Script

The dynamic inventory script for Azure resource management is called azure_rm.py. You can download it from [here](https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/azure_rm.py). Please also download the config file [azure_rm.ini]((https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/azure_rm.ini) to the same directory. 

Run below command to add executable permission to the script.

```bash
chmod +x azure_rm.py
```

The dynamic inventory script can be executed directly, passed as a parameter to the ansible command, or passed directly to ansible-playbook using the -i option. No matter how it is executed the script produces JSON representing all of the hosts found in your Azure subscription. You can narrow this down to just hosts found in a specific set of Azure resource groups, or even down to a specific host.

```bash
./azure_rm.py --help
```

Blow command lists all the hosts under your current subscription. 

```bash
./azure_rm.py --list --pretty
```

## Host Groups

By default hosts are grouped by:

- azure (all hosts)
- location name
- resource group name
- security group name
- tag key
- tag key_value

You can control host groupings and host selection by either defining environment variables or editing the `azure_rm.ini` file.

NOTE: The .ini file will take precedence over environment variables.

Control grouping using the following variables defined in the environment.

```
AZURE_GROUP_BY_RESOURCE_GROUP=yes
AZURE_GROUP_BY_LOCATION=yes
AZURE_GROUP_BY_SECURITY_GROUP=yes
AZURE_GROUP_BY_TAG=yes
```

Select hosts within specific resource groups by assigning a comma separated list.

```
AZURE_RESOURCE_GROUPS=resource_group_a,resource_group_b
```

Select hosts for specific tag key by assigning a comma separated list of tag keys.

```
AZURE_TAGS=key1,key2,key3
```

Select hosts for specific locations by assigning a comma separated list of locations.

```
AZURE_LOCATIONS=eastus,eastus2,westus
```

Or, select hosts for specific tag key:value pairs by assigning a comma separated list key:value pairs.

```
AZURE_TAGS=key1:value1,key2:value2
```

If you don’t need the powerstate, you can improve performance by turning off powerstate fetching.

```
AZURE_INCLUDE_POWERSTATE=no
```


## Examples
Here are some examples using the inventory script:

Execute /bin/uname on all instances in the `Testing` resource group

```bash
ansible -i ./azure_rm.py Testing -m shell -a "/bin/uname -a"
```

Use the inventory script to print instance specific information

```bash
./azure_rm.py --host my_instance_host_name --resource-groups=Testing --pretty
```

Use the inventory script with ansible-playbook

```bash
ansible-playbook -i ./azure_rm.py test_playbook.yml
```

Here is a simple playbook to exercise the Azure inventory script:

```yml
- name: Test the inventory script
  hosts: azure
  connection: local
  gather_facts: no
  tasks:
    - debug: msg="{{ inventory_hostname }} has powerstate {{ powerstate }}"
```

You can execute the playbook with something like:

```bash
ansible-playbook -i ./azure_rm.py test_azure_inventory.yml
```