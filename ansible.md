# python virtual env
```
virtualenv venv
source venv/bin/activate
```

# install ansible
```
pip install ansible
```

# get ansible version
```
ansible --version
```

# generate SSH keygen
```
ssh-keygen -t rsa -b 4096 -C "alma@koretefa.hu" -f ./torkoly
```

# create ansible directory
```
mkdir ansible
cd ansible
nano dev
```

# create ansible directory and a dev file
```
[ipon]
52.166.119.116
104.45.7.170
```

```
nano ansible.cfg
# create ansible.cfg file in ansible directory
[defaults]
inventory = /home/mau/ansible/dev
remote_user = root
host_key_checking = False
private_key_file = /home/mau/torkoly
```
# run the first command
```
ansible -m ping ipon
```

# run the second command
```
ansible -m command -a 'w' ipon
```

# create roles directory
```
mkdir roles
cd roles
# create default ansible directories
ansible-galaxy init nginx
ansible-galaxy init tools # epel-release miatt
ansible-galaxy init firewalld
```

### tree of nginx directory
```
tree nginx
```

### create ansible playbook azure-server.yml
``` yaml
---
- hosts: azure
  vars:
    - course: Tigra
  become: true
  roles:
    - firewalld
    - tools
    - nginx
```
### roles/firewalld/tasks/main.yml
``` yaml
---
- name: install firewalld
  yum:
    name: firewalld
    state: present

- name: enable firewalld service
  service:
    name: firewalld
    state: started
    enabled: yes
```

### roles/tools/tasks/main.yml
``` yaml
---
- name: install tools
  yum:
    name: "{{ item }}"
    state: present
  with_items:
    - epel-release
    - net-tools
```

### roles/nginx/tasks/main.yml
``` yaml
---
- name: install nginx
  yum:
    name: nginx
    state: present

- name: enable nginx service
  service:
    name: nginx
    state: started
    enabled: yes

- name: allow http - firewalld
  firewalld:
    service: http
    state: enabled
    permanent: true
  notify:
    - restart firewalld

- name: copy index.html to /usr/share/nginx/html
  template:
    src: "index.html.j2"
    dest: "/usr/share/nginx/html/index.html"
```

### create roles/nginx/handlers/main.yml
``` yaml
---
- name: restart firewalld
  service:
    name: firewalld
    state: restarted
```

### create roles/nginx/vars/main.yml 
``` yaml
---
ipon:
  ipon_1: value_1
  ipon_2: value_2
```

### create roles/nginx/templates/index.html.j2
``` html
{{ inventory_hostname }} - Server -- {{ course }} -- {{ ipon.ipon_1 }} -- {{ ipon.ipon_2 }}
```

### inventory
```
{% for host in groups['azure'] %}
   {{ hostvars[host]['ansible_eth0']['ipv4']['address'] }}
{% endfor %}
```

### start ansible playbook
```
ansible-playbook azure-server.yml
```

### vault
```
echo 'tigrafzopassword' > secret_password
```

### create string
```
ansible-vault encrypt_string --vault-id secret_password 'meaning_of_life_42' --name 'life'
```

### encrypted variables start ansible playbook
```
ansible-playbook --vault-id secret_password azure-server.yml
```

### List all hosts
```
ansible --list-hosts all
ansible --list-hosts "*"
```

### List hosts of inventory file
```
ansible -i ./hosts --list-hosts all.
```

### You can list group
```
ansible --list-hosts webservers
```

### You can list host without for example loadbalancer (regexp)
```
ansible --list-hosts \!loadbalancer
```

### Return the first element of group
```
ansible --list-hosts "webservers[0]"
```
