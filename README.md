# Ansible_automation
ansible automation
Ansible Installation steps on Ubuntu Instance

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04


#### Ansible windows setup ref:


**Link**:  https://www.devopsschool.com/tutorial/ansible/ansible-windows-adhoc-commands.html#Program1

Set Up Windows Inventory File Correctly
Following:


[win]
172.16.2.5 
172.16.2.6 

[win:vars]
ansible_user=LocalUsername
ansible_password=Password
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore
Ansible Windows Adhoc Commands
Following:


$ ansible [host_group_name_in_inventory_file] -i hosts -m win_ping

$ ansible win -i inventory -m win_ping 
$ ansible win -i inventory -m setup
$ ansible win -i inventory -m raw -a "dir"
$ ansible win -i inventory -m win_service -a "name=spooler"
$ ansible win -i inventory -m win_service -a "name=spooler state=stopped"
$ ansible win -i inventory -m win_service -a "name=spooler"
$ ansible win -i inventory -m win_feature -a "name=Telnet-Client state=present"    
