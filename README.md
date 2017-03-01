# Ansible Role: Hound

I'm automating my entire fleet using ansible. This role installs [hound](https://github.com/etsy/hound).

## Requirements

This role is intended for linux boxes on systemd, with a working golang and nginx
install. If you are provisioning a bare box, the following roles might be of interest:

- [gantsign.golang](https://github.com/gantsign/ansible-role-golang)
- [geerlingguy.nginx](https://galaxy.ansible.com/geerlingguy/nginx/)

You will need to pick a user for the service to run under. Seeing as hound needs
to be able to clone private repos, this user should have a home folder set with a
`.ssh` folder present with the appropriate config.

## Role Variables

```
go_root: /usr/local/go # Location of go installation
hound_port: 8301
hound_server_name: hound # used in nginx server_name
hound_user: # your dev user should be set here.
hound_data_dir: /var/hound # stores config and data index.
hound_config:
  max-concurrent-indexers: 2
  dbpath: data
  repos:
    HoundRole:
      url: https://github.com/issmirnov/ansible-role-hound.git
```

For your `hound_config`, create one as per [etsy's sample](https://github.com/etsy/hound/blob/master/config-example.json)
and then feed it into a [YAML cruncher](https://www.json2yaml.com/)

## Example Playbook

```
- name: Install hound.
  hosts: your_server
  vars:
    hound_user: dev
    hound_config:
      max-concurrent-indexers: 2
      dbpath: data
      repos:
        HoundRole:
          url: https://github.com/issmirnov/ansible-role-hound.git

  roles:
    - issmirnov.hound

```

If you are provisioning a bare bones server, you might try:

```
- name: install ansible deps on bare server
  remote_user: root
  hosts: your_server
  gather_facts: no
  pre_tasks:
    - name: 'install python2 and json support'
      raw: sudo apt-get -y install python-simplejson
    - name: 'create dev user'
      raw: sudo adduser --system --group dev

- name: Install nginx, golang, hound
  hosts: your_server
  vars:
    golang_gopath: '$HOME/go'
    hound_user: dev
    hound_server_name: hound
    hound_config:
      max-concurrent-indexers: 2
      dbpath: data
      repos:
        HoundRole:
          url: https://github.com/issmirnov/ansible-role-hound.git

  roles:
    - geerlingguy.nginx
    - gantsign.golang
    - role: issmirnov.hound
      go_root: "{{ansible_local.golang.general.home}}"
```

## License

MIT

## Author Information

Ivan Smirnov, http://ivansmirnov.name
