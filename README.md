# lde
Linux Development Environment via Ansible

I have come to dependend upon a custom development environment while working in Linux.  It includes a:
* custom vimrv
* build of vim from scratch
* build of YouCompleteMe from vim
* tmux and a custom configuration
* UMLet drawing tools (Java support)
* custom scons builds for UMLet

# Quick Start
python3 -m venv venv
. ./venv/bin/activate
pip install ansible
ansible-playbook -i personal lde.yml
