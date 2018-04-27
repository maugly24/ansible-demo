## Working with ansible-vault

```
roles/aliens
	├── tasks
	│   └── main.yml
	└── vars
		    └── spoilers.yml
First, put your spoiler text in a roles/aliens/vars/spoilers.yml:
```

``` yaml
---
spoiler_text: | 
  people run into some space aliens
  and they end up fighting them
(Note the pipe, followed by the new line with text indented by two spaces. This allows you to easily put multi-line text into a variable.)
```
Then, reference your spoiler_text variable in your task:

``` yaml
---
- include_vars: spoilers.yml

- name: Put the spoiler text in the tmp directory on the remote server.
  copy:
    content="{{spoiler_text}}"
    dest=/tmp/spoiler_text.txt
```

Encrypt your spoilers file using your vault password file on the command line:

```
$ ansible-vault encrypt roles/aliens/vars/spoilers.yml --vault-password-file ~/.vault_pass.txt
```

Encryption successful
You can now safely put this file in your source control without spoiling the movie for everyone.

```
$ head -n3 aliens/vars/spoilers.yml
$ANSIBLE_VAULT;1.1;AES256
61616366326131636131323230613333356361333737356566646133343062623061313931666462
3933316533346664393430643963646533663737343434320a613862353665663862393939383336
...
```
Then, given a playbook that looks like:

``` yaml
---
# file: movies.yml
- hosts: all

  roles:
    - { role: aliens }
```

You can now run this against your server:

```
$ ansible-playbook -i inventory/development.hosts playbooks/movies.yml --vault-password-file ~/.vault_pass.txt
```
That's it! Hop on the server and you can see that the decrypted content is there on disk:

```
remote_server$ cat /tmp/spoiler_text.txt 
```