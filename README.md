# Ansible multi-environment demo

This project demonstrates how to manage multiple environments with one set of Ansible code.

Using vagrant, we configure two separate environments, dev and production. Start by bringing these all up

    $ vagrant up

This brings up two environments, each consisting of a load-balancer and a number of web servers. The aim of this project is to show how to separate out environment-specific configuration.


An environment is described by an *inventory* file. You can see two of them in the inventories directory, a dev and a prod one. The dev inventory looks like this

```
[local]
127.0.0.1 ansible_connection=local

[loadbalancers]
192.168.50.10

[webservers]
192.168.50.11
192.168.50.12

[dev:children]
local
loadbalancers
webservers
```