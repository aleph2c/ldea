# ldea

ldea: Linux Development Environment via Ansible.

Install a trusted deployment machine:

  * A deployment machine with access to github
  * An ssh forwarding strategy to keep your keys off of your target machines
  * Access to an encrypted vault on the deployment machine (currently not used)

Perform the following on a target(s) from the deployment machine:

  * Tmux and a customized Tmux configuration
  * Install a custom init.vim
  * Use the Plugin manager to install nvim plugins
  * Customize the python environment
  * Customize the python debugger
  * Customize your .bashrc file
  * Install and then fix Umlet ('umlet' will work from command line)
  * Install a custom umlet template

# Initial setup of Deployment Machine

We want to use ansible to automate the installation of our development tools
on other machines from your deployment machine.  To do this we must configure
ssh.

On the deployment machine ensure your sshd accepts passwords.
```
sudo apt-get install vim
sudo vim /etc/sshd/sshd_config
# uncomment PasswordAuthentication
```

Restart the sshd:
```
# in a debian based Linux machine:
sudo systemctl restart sshd

# or on the WSL:
sudo service ssh restart
```

Create your ssh keys if they don't exist (to check ``ls -l ~/.ssh/id_*``)
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Ensure you can ssh into this machine without keys:
```
# on deployment machine, ex: 10.0.0.21
ssh-copy-id pi@10.0.0.21
```

Confirm access
```
ssh <username>@127.0.0.1
exit
```

Copy your deployment machine's public keys,  ``cat ~/.ssh/id_rsa.pub``, to your
[github keys](https://github.com/settings/keys).  Make a point of naming the key
entry after the contract/project you are working on, you will remove this access
after you have finished the work.

# Install your Deployment machine using ansible

This procedure will configure ansible for "ssh key forwarding".  This is useful
if you want to pull down code on other machines, without leaving your github
accessible ssh keys on those machines.  Ansible is good at automating things, so
we use ansible to configure its own ansible environment to enable ssh key
forwarding to the other machines it will control.

Update your ``group_vars/all`` file with the correct user, group.

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
ansible-playbook -i personal deployment_machine.yml -K
```

Re-initialize bash and restart your venv.
```
source ~/.bashrc
source ./venv/bin/activate
```

When this is done, ansible will work using ssh key forwarding and it will be
able to work with your encrypted vault files if you choose to use them.

# Pre-deployment Work

On each machine you want to deploy to, ensure that sshd is installed and
running, and that it is accepting passwords and it will accept ssh key forwarding.

Access each machine and test to see if the sshd service is installed:
```
systemctl status sshd
# or on the WSL
service ssh status
```

On each machine you want to control, edit their sshd config and ensure that
PasswordAuthentication and AllowForwarding are enabled.  If not, enable them,
then restart their sshd daemon.

```
sudo nano /etc/sshd/sshd_config
# uncomment PasswordAuthentication and AllowForwarding
sudo systemctl restart sshd
# or on WSL
service ssh restart
```

From your deployment machine, copy your keys onto each machine you want to
deploy to.

```
# on deployment machine 10.0.0.21
ssh-copy-id pi@10.0.0.22
# .. same command for other targets

```

Confirm you can ssh forward to each machine you want to deploy to.

```
# do this to each machine you want to control
ssh -A pi@10.0.0.22
```

Once you have logged into the other machine you can see the ssh keys you are using with:

```
# do this on each machine you want to control
ssh-add -L
exit
```

You can now ssh from your deployment machine to each of your deployment machines
without needing a password and with the option of forwarding your deployment
machine's ssh keys.  Remember that ssh key forwarding is useful; you can pull
done code from github on the remote machine as if it were the deployment
machine.  This keeps important keys off those remote machines.

If you want to remove the ability for someone to ssh onto the remove machine
with a password, log back into it and:

```
sudo nano /etc/sshd/sshd_config
# uncomment PasswordAuthentication
sudo systemctl restart sshd
# or on WSL
service ssh restart
exit
```

# Deployment

Update your ``group_vars/all`` file with the correct user, group, vimrc repo,
nvim repo, tmux configuration repo and .pdb configuration repo.

To only install tmux, nvim, your init.vim, it's plugins:

```
ansible-playbook -i personal basic_development_env.yml
```

If you want to specify the user on the command line:

```
ansible-playbook -i personal basic_development_env.yml -K -e "user=bob"
```
