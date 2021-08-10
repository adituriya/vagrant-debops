# Vagrant-DebOps Starter Project

This is a barebones starter project that runs
on a local Vagrant box functioning as its own
DebOps controller, ready to configure as needed
for your new development project.

Almost anything is possible, you can use any language, any framework,
any tech stack&mdash;anything that runs on Debian (or Ubuntu) and is either
supported directly by [DebOps](https://docs.debops.org/) or configurable as a custom [Ansible](https://docs.ansible.com/ansible/latest/index.html) role.

In the end, you will have a clearly defined set of server requirements,
providing anyone who clones your project a consistent, reproducible
dev environment while also specifying unambiguously what is required of
staging and production servers (whether or not they get provisioned using DebOps).

See [Provisioning a Local Dev Box using Debops and Vagrant](https://code.turiya.dev/provisioning-a-local-dev-box-using-debops-and-vagrant) for a
walkthrough on how to get this running on your local machine.

## Project Layout

`./config` contains the DebOps project. It is where you
will define your dev server environment(s). This folder
will be mounted to the guest VM as `/home/vagrant/debops`
which is consequently where you will run `debops`.

`./src` contains your source code (to be written),
and an example `Vagrantfile` (to be modified).
This folder gets mounted to the guest VM as `/vagrant`,
which is where your project will be hosted (bear this
in mind when configuring your application).

If all goes well (is well configured), DebOps will run as part of the initial
provisioning, the first time you `vagrant up`. Subsequently,
you will use `vagrant ssh` to connect to the instance,
then `cd debops && debops` (details below).

## Installation

To set up a new project,

```
git clone https://www.github.com/adituriya/vagrant-debops example
```

changing `example` to your preferred project directory name.

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

Once all three steps above are complete, commit your changes and
bring up the instance by navigating to the `src` directory

```
cd example/src
```

and running

```
vagrant up
```

The first time around, it will take around 10 minutes to run.
Subsequent runs will be significantly faster. Once the instance is up,
if you need to re-run `debops` (say you updated your configuration), then

```
vagrant ssh
cd debops
```

From there you can run

```
debops
```

with any additional arguments needed (you can limit the run
to a given host, group or tag, for example). This will apply
your current configuration to the Vagrant box.

The intended workflow is that you define your project's
server requirements by enabling DebOps roles in
`config/ansible/inventory/hosts` and specifying their
configuration parameters in `config/ansible/inventory/host_vars`
and `config/ansible/inventory/group_vars`. Then, apply them
by running `debops` on the Vagrant box.

See (DebOps documentation)[https://debops.org] for full
details on available roles and configuration variables.

Finally, when you are done for the day,

```
vagrant halt
```

to shut down the instance, or

```
vagrant destroy
```

to completely remove the instance from your computer.