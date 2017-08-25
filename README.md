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

We leverage group_vars here. There are three groups defined

 * local
 * loadbalancers
 * webservers

And the dev:children one at the bottom to encapsulate them all. Ansible will use that name, *dev* to look up group vars in our group_vars folder. In that folder, there are two more folders: dev and production. These tie back to the dev:children (and obviously, production:children) groups. 

Now that our environments exist, let's configure one of them.

    $ ansible-playbook site.yml -i inventories/dev

Shortly after, the play will finish. Hit the load balancer

    $ curl http://192.168.50.10

 You will see

 ```
 <!DOCTYPE html>
<html>
  <body>
    <h1>Enviroment</h1>
    <ul>
      <li>Node: 192.168.50.11
      <li>Environment: dev</li>
      <li>App name: webserver</li>
    </ul>
  </body>
</html>
```

That line Environment: dev is what we're interested in. Delve into the code a little. Look at roles/webserver/templates/index.htmnl.j2

This is the template for index.html on all the web servers. As you can see, it references a variable "env". Where does this come from?  Well, if you look at group_vars/dev/common.yml, you'll see it defined there. Similarly, production is in group_vars/production/common.yml. The inventory file defines which group vars are applicable, and the relevant config is pulled out of there. There's also an *all* group vars, for across-the-board config. Hint: that's where App name comes from.

Now configure the production ennvironment

    $ ansible-playbook site.yml -i inventories/production
    $ curl http://192.168.60.10

You'll see it's pulling the env variable from the production group vars. 