# ldea

ldea: Linux Development Environment via Ansible.

Install:

  * Tmux and a customized Tmux configuration
  * Build a full featured vim8.2 natively on remote machine
  * Install custom .vimrc
  * Use the Plugin manager to install vim plugins
  * Build YouCompleteMe for vim8.2
  * Customize the python environment
  * Customize the python debugger
  * Customize the .bashrc file
  * Install and then fix Umlet ('umlet' will work from command line)
  * Install a custom umlet template


# Initial setup of keys on deployment machine

Login to your deployment machine:
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Copy public keys to your [github
keys](https://github.com/settings/keys).  Make a point of naming the key entry
after the contract/project you are working on, you will remove this access after you
have finished the work.


# Quick Start

Ensure the remote has a .ssh directory, and then copy your public and private
key onto the machine, so that that machine will have the same access to github that
your deployment computer has.

```
cat ~/.ssh/id_rsa.pub | ssh <remote_user>@<ip> "mdir -p ~/.ssh && chmod 700 && cat >> ~/.ssh/authorized_keys"
scp ~/.ssh/id_rsa <remote_user>@<ip>:~/.ssh
scp ~/.ssh/id_rsa.pub <remote_user>@<ip>:~/.ssh

```

Install ansible on the machine you will be deploying from

```
python3 -m venv venv
. ./venv/bin/activate
pip install ansible
```

Create an inventory file ('personal' example below):
```
[all]
10.0.0.23 ansible_connection=ssh ansible_user=<remote_user>
```

Update your ``group_vars/all`` file with the correct user, group, vimrc repo,
tmux configuration repo and .pdb configuration repo.


Run the installation:
```
ansible-playbook -i personal site.yml
```

To only install tmux, vim, your vimrc, it's plugins and YouCompleteMe:

```
ansible-playbook -i personal basic_development_env.yml
```

To only install a non-broken version (works headlessly) of UMLet:

```
ansible-playbook -i personal umlet.yml

```
