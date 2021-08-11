# Vagrant-DebOps Project Template

This is a project template for running arbitrary code (whatever you create)
on a local Vagrant box configured using DebOps.

Although this limits you to Debian and Ubuntu operating systems
and the general capabilities of Vagrant, Ansible and DebOps,
this already provides everything you need to deploy a very wide array
of applications and computing environments.

[DebOps](https://docs.debops.org/) includes roles for
Python, PHP, Ruby, NodeJS, Java, Go, R, Neurodebian, WP-CLI, Elasticsearch, MariaDB,
PostgreSQL, Redis, Nginx, Apache, GUnicorn, Docker, (and more),
plus a full stack of systems-level configuration roles,
including your own PKI infrastructure, secure secrets storage, and over a dozen
ready-to-deploy applications (Gitlab, DokuWiki, Etherpad, Nextcloud...).

Plus, it is highly extensible: if your project requires something else,
you can write your own Ansible roles (or use third-party roles).

## Project Layout

`./config` contains the DebOps project. It is where you
define your dev server environment(s).
It gets mapped to `/home/vagrant/debops` on the guest VM,
which is where you will run `debops`.

`./src` contains your source code (to be written),
and an example `Vagrantfile` (to be modified).
It gets mapped to `/vagrant` on the guest VM,
which is where your project will be served from.

## Installation

To create a new project using this template, open a terminal and change to the parent
directory where you want to create the project, then

```
git clone https://github.com/adituriya/vagrant-debops.git
```

If you want the project to be named something other than `vagrant-debops`,
then add your preferred directory name (following a space)
at the end of the git clone command.

## Configuration

1. Edit `config/ansible/inventory/group_vars/all/netbase.yaml`.
The variable in this file should contain your project's top-level
domain name (TLD), which may be fictitious (e.g. `project.local`)
if you don't have a real one, since we will simply be mapping it
to a local IP address.

2. Edit `config/ansible/inventory/hosts`.
Edit the lines

```
[debops_all_hosts]
example ansible_host=example.project.local ansible_connection=local
```

so that `example` is changed to your preferred hostname
and `project.local` is changed to the TLD you entered in the
previous step.

This is also where you enable additional DebOps roles for
provisioning your project's various server dependencies (more on this
below - it does not need to be done now).

3. Edit `src/Vagrantfile`. The first few lines contain
variables that may need to be customized:

```
vagrant_box = 'debian/buster64'
```

can be any relatively recent Debian or Ubuntu box image from
https://app.vagrantup.com/boxes/search

```
hostname = 'example'
```

should match the hostname you specified in the Ansible inventory
file in the previous step;

```
local_domain = 'project.local'
```

should match the TLD that you specified in the previous step; and

```
local_ip = '192.168.11.11'
```

is any unused IP address on your local network.


## Usage

Once the three steps above are complete, commit your changes and
bring up the instance by navigating to the `src` directory and running

```
vagrant up
```

The first time, this will take around 10 minutes to run.
Subsequent runs will be faster. Once the instance is up,
if you need to re-run `debops` (say you updated your configuration), then

```
vagrant ssh
cd debops
```

From there you can apply your current configuration by running

```
debops
```

with any additional arguments needed (you can limit the run
to a given host, group or role, for example).

The intended workflow is to define your project's
server requirements by enabling DebOps roles in
`config/ansible/inventory/hosts` and specifying their
configuration parameters in `config/ansible/inventory/host_vars`
and `config/ansible/inventory/group_vars`. Then, apply them
by running `debops` on the Vagrant box.

See [DebOps roles documentation](https://docs.debops.org/en/master/ansible/roles/index.html)
for details on available roles and configuration variables.

Finally, when you are done with your session,

```
vagrant halt
```

to shut down the instance, or

```
vagrant destroy
```

to remove the instance from your computer.