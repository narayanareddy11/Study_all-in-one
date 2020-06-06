### 
-  sudo yum install ansible
- useradd ansible
- passwd ansible
- ssh-keygen 
- ssh-copy-id ansible@TARGET_HOST
- /etc/sudoers : ansible ALL=(ALL) NOPASSWD: ALL
- /etc/ansible/ansible.cfg
```
Inventory file:
mail.example.com ansible_port=5556 ansible_host=192.168.0.10
[webservers]
httpd1.example.com
httpd2.example.com
[labservers]
lab[01:99]
```
