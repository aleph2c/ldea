# ldea

ldea: Linux Development Environment via Ansible.

Install a trusted deployment machine:

  * A deployment machine with access to github
  * An ssh forwarding strategy to keep your keys off of your target machines
  * Access to an encrypted vault on the deployment machine (currently not used)

Perform the following on a target(s) from the deployment machine:

  * Tmux and a customized Tmux configuration
  * Build a full featured vim8.2 natively on the remote/local machine
  * Install a custom .vimrc
  * Use the Plugin manager to install vim plugins
  * Build YouCompleteMe for vim8.2
  * Customize the python environment
  * Customize the python debugger
  * Customize your .bashrc file
  * Install and then fix Umlet ('umlet' will work from command line)
  * Install a custom umlet template

# Initial setup of Deployment Machine

Login to your deployment machine:
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Copy public keys,  ``cat ~/.ssh/id_rsa.pub``, to your [github
keys](https://github.com/settings/keys).  Make a point of naming the key entry
after the contract/project you are working on, you will remove this access after you
have finished the work.

# Install your deployment machine using ansible
Clone this repo:
```
git clone git@github.com:aleph2c/ldea.git
cd ldea
```

Create a personal inventory from the personal_example:
```
cp personal_example personal
```

Customize your personal inventory:
```
[all]
10.0.0.21
10.0.0.22

[deployment_machine]
10.0.0.21

[targets]
10.0.0.22
# more ...
```

Setup the virtual environment:
```
# in ldea
python3 -m venv venv
source ./venv/bin/active
pip install --upgrade pip
pip install -r requirements.txt
```

Setup your deployment machine:
```
ansible-playbook -i personal deployment_machine.yml
```

# Quick Start

Copy your deployment machine's public keys to your target machines.  This will
let ansible ssh to the target without a password.

```
# on deployment machine 10.0.0.21
ssh-copy-id pi@10.0.0.22
# .. same command for other targets

```

Update your ``group_vars/all`` file with the correct user, group, vimrc repo,
tmux configuration repo and .pdb configuration repo.

Run the installation:
```
ansible-playbook -i personal site.yml
```

To only install tmux, vim, your .vimrc, it's plugins and YouCompleteMe:

```
ansible-playbook -i personal basic_development_env.yml
```

To only install the python basic environment

```
ansible-playbook -i personal python_env.yml

```
