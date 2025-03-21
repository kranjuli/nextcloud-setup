Ansible: Create nextcloud collection

````
# create skeleton of collection
ansible-galaxy collection init <namespace>.<collection_name>
namespace/
    └── collection_name/
        ├── playbooks/
        │   └── playbook_1.yml
        ├── roles/
        │   ├── role_1/
        │       ├── role_1/
        │           ├── tasks/
        │               └── main.yml
        ├── galaxy.yml
# implements roles in the collection
# build collection 
ansible-galaxy build
# collection <namespace>-<collection>-<version>.tar.gz will be built
# install collection 
ansible-galaxy install <namespace>-<collection>-<version>.tar.gz
# run playbook
ansible-playbook <namespace>.<collection>.playbook_1.yml 
````
