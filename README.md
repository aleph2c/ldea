# ldea

Linux Development Environment via Ansible, to have a single location from which
you can switch all of your keys.

I have come to depend upon a custom development environment while working in Linux.  It includes a:
* custom .vimrc
* build of vim from scratch
* build of YouCompleteMe from vim
* tmux and its customization
* UMLet drawing tools (Java support)
* custom scons builds for UMLet
* access to my git repos

# Quick Start

This deployment strategy assumes that the keys on the remote machine are the same as
this machine.  This way you won't have to manage multiple sets of keys in your lab, and
it will be an easy thing to switch your keys as you switch contracts. (leaving
behind harmless baby-borgs).

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
ansible-playbook -i personal lde.yml
```
