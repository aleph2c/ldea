# ldea

ldea: Linux Development Environment via Ansible.

Install a trusted deployment machine:

  * A deployment machine with access to github
  * An ssh forwarding strategy to keep your keys off of your target machines
  * The vault plumbing the other ansible repos rely on: the deployment-machine
    playbook exports ``ANSIBLE_VAULT_PASSWORD_FILE=./.vault_pass`` (relative, so
    each repo's own committed script is used from its root); ``VAULT_PASSWORD``
    itself lives in ``~/.bashrc.secrets``. ldea itself has nothing
    encrypted; ``missionloops_ansible`` does, and its vaults open with that one
    password

Install the following on a target(s) from the deployment machine:

  * Tmux and a customized Tmux configuration
  * Neovim, a custom init.vim, and its plugins via vim-plug
  * The tools that init.vim expects: ripgrep and fzf (search and pickers),
    universal-ctags (jump to definition), glow (markdown preview), xclip
    (clipboard) and python3-pynvim (the Python provider)
  * A git template so every repo cloned or init'ed afterwards rebuilds its
    ctags index on commit, merge, checkout and rebase
  * Customize the python environment
  * Customize the python debugger
  * Customize your .bashrc file, and create the two files it sources when they
    are missing: ``~/.bashrc.local`` (this machine only) and ``~/.bashrc.secrets``
    (mode 600; holds ``VAULT_PASSWORD`` and any API keys; never committed or
    backed up)
  * Install and then fix Umlet ('umlet' will work from command line)
  * Install a custom umlet template

# Playbooks and roles

| Playbook | Runs on | Roles |
|---|---|---|
| ``basic_development_env.yml`` | targets | git, tmux, nvim, bashrc |
| ``python_env.yml`` | all | pdb, pip |
| ``umlet.yml`` | all | umlet (pulls in java and pip) |
| ``site.yml`` | | the three above, in that order |
| ``deployment_machine.yml`` | deployment_machine | git, ansible (pulls in ssh and redis), nvim |

# Initial setup of Deployment Machine

We want to use ansible to automate the installation of our development tools
on other machines from your deployment machine.  To do this we must configure
ssh.

On the deployment machine ensure your sshd accepts passwords.
```
sudo apt-get install vim
sudo vim /etc/ssh/sshd_config
# uncomment PasswordAuthentication
```

Restart the sshd daemon:
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

Confirm access:
```
ssh <username>@127.0.0.1
exit
```

Copy your deployment machine's public keys,  ``cat ~/.ssh/id_rsa.pub``, to your
[github keys](https://github.com/settings/keys).  Make a point of naming the key
entry after the contract/project you are working on. This makes it easy to
remove this access after you have finished the work.

# Setting up your Deployment Machine

This procedure will configure ansible for "ssh key forwarding".  This is useful
if you want to pull down code on other machines, without leaving your
github-accessible ssh keys on those machines.  Ansible is good at automating
things, so we use ansible to configure its own ansible environment to enable "ssh
key forwarding" to the other machines it will control.

Update your ``group_vars/all`` file with the correct user, group.

Clone this repo:
```
git clone git@github.com:aleph2c/ldea.git
cd ldea
```

Create a personal inventory from the personal_example (the personal file will be
ignored by git):
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
source ./venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Setup your deployment machine:
```
ansible-playbook -i personal deployment_machine.yml -K
```

The first run of this playbook reboots the machine if ``AllowAgentForwarding``
was not already enabled in its sshd configuration. Expect it, and re-run the
playbook afterwards. Later runs do not reboot.

Re-initialize bash and restart your venv.
```
source ~/.bashrc
source ./venv/bin/activate
```

When this is done, ansible will work using "ssh key forwarding" and it will have
access to your encrypted vault files if you choose to use them.

# Pre-Deployment Work

ldea configures an account; it does not create one. Getting a new target from
a fresh install to its first ldea run:

1. Install Debian or Ubuntu. The installer's account will do, or make one:
   ```
   sudo adduser scott
   sudo usermod -aG sudo scott
   ```
2. Make sure sshd is on the target: ``sudo apt install openssh-server``.
3. Follow the sshd steps below (password login on for now, agent forwarding
   on), then ``ssh-copy-id`` from the deployment machine.
4. Put the target in ``personal`` and name the account, unless it matches your
   login on the deployment machine:
   ```
   [targets]
   10.0.0.22 ansible_user=scott
   ```
5. Run ``ansible-playbook -i personal basic_development_env.yml -K``. The
   ``-K`` asks for that account's sudo password.
6. Optionally turn password login back off, as described at the end of this
   section.

On each machine you want to deploy to, ensure that sshd is installed and
running, and that it is accepting passwords and it will accept ssh key forwarding.

Access each machine and test to see if the sshd service is installed:
```
systemctl status sshd
# or on the WSL
service ssh status
```

For each machine you want to control, edit their sshd config and ensure that
``PasswordAuthentication`` and ``AllowForwarding`` are enabled.  If not, enable
them, then restart their sshd daemon.

```
sudo nano /etc/ssh/sshd_config
# uncomment PasswordAuthentication and AllowAgentForwarding
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

Confirm you can "ssh forward" to each machine.

```
# do this to each machine you want to control
ssh -A pi@10.0.0.22
```

Once you have logged in, confirm that you can see your forwarded ssh keys:

```
# do this on each machine you want to control
ssh-add -L
ssh git@github.com # say yes
exit
```

You can now ssh from your deployment machine to each of your other computers,
without needing a password, and with the option of forwarding your deployment
machine's ssh keys.  Remember that ssh key forwarding is useful; you can pull
down code from github on the remote machine as if it were the deployment
machine.  This keeps important keys off those other computers.

Now that you have set things up, you might want to remove the ability for
someone to ssh onto the other computers, with a password.  If this is the case,
log back into each machine and do the following:

```
sudo nano /etc/ssh/sshd_config
# uncomment PasswordAuthentication
sudo systemctl restart sshd
# or on WSL
service ssh restart
exit
```

# Deployment

``group_vars/all/vars.yml`` holds the git identity and the tmux and pdb
configuration repos. The account to configure defaults to whoever ansible logs
in as; set ``user=`` in the inventory or on the command line to override it.

To install tmux, nvim, its configuration and plugins, and the git hooks:

```
ansible-playbook -i personal basic_development_env.yml -K
```

To configure the machine you are sitting at, no sshd needed, give it a local
connection in the inventory (see ``personal_example``):

```
[targets]
127.0.0.1 ansible_connection=local
```

If you want to specify the user on the command line:

```
ansible-playbook -i personal basic_development_env.yml -K -e "user=bob"
```

Existing clones do not pick up the git template hooks automatically; run
``git init`` once inside each to copy them in.

# Checking a run

Every task is safe to repeat: packages, clones, symlinks and ``.bashrc`` lines
all converge on the same state. A second run is therefore harmless, but it will
not yet report ``changed=0``, because a few tasks are shell probes or ``touch``
operations that ansible cannot see through. Until those are converted, judge a
run by the ``failed`` count and by what the machine does afterwards:

```
nvim                     # opens with plugins, no startup messages
nvim +checkhealth        # Python 3 provider OK
tmux                     # custom configuration in force
git config --get init.templatedir   # ~/.git_template
```

The nvim and tmux roles pull their configuration repositories with
``update: yes``, which resets the checkout to what the remote has. Uncommitted
edits make the task refuse; commits that were never pushed would be silently
dropped, so both roles first count commits ahead of the remote and fail with a
message if there are any. Either way the rule is the same: commit **and push**
on the machine where you edited before running ldea there.
