# #! Admin Tools

Ansible playbooks and other admin tools/docs for maintaining the #! network.


## Requirements

  * Recent version of Ansible
  * Local [#! pass database](https://github.com/hashbang/password-store)
  * User with sudo access on all servers


### Git configuration

You might also want to use the following snippet in `~/.gitconfig`:

	[diff "gpg"]
		textconv = gpg --no-tty --decrypt
		cachetextconv = false
	[diff "ansible-vault"]
		textconv = ansible-vault view
		cachetextconv = false


### SSH configuration

All the “service servers” (as opposed to shell servers) listen for SSH
on port 8993 (ASCII-encoding of `#!`), and the user is `core`, with
the following exceptions:
- `lon1.irc.hashbang.sh` and `sfo1.irc.hashbang.sh`
  do not yet follow that convention;
- `git-infra.hashbang.sh` is a service hosted on `nyc3.apps.hashbang.sh`
  which uses port 22.

This is expressed in the following `.ssh/config` snippet:

	Host da1.hashbang.sh ny1.hashbang.sh sf1.hashbang.sh to1.hashbang.sh
	     User your_nick

	Host git-infra.hashbang.sh
	     User git

	Host sfo1.irc.hashbang.sh lon1.irc.hashbang.sh
	     User core

	Host *.hashbang.sh hashbang.sh
	     User core
	     Port 8993


## Playbooks

There are 3 playbooks present here:
- `shell.yml` is used to synchronise the configuration (incl. installed packages)
  across the shell servers.
- `credentials.yml` is used to deploy the admin's SSH keys across all servers:
  - admins can login as `root` on the shell servers;
  - they can login as `core` on the CoreOS servers.
- `coreos.yml` performs CoreOS-specific tasks.  Currently, it only bootstraps
  tha Ansible agent's dependencies.


## Usage

### Install a package

  1. Connect to any shell server as yourself

      ```bash
      ssh you@ny1.hashbang.sh
      ```

  2. Install Package

      ```bash
      sudo apt-get install some-package
      ```

  3. Deploy changes on other servers

      The configuration changes (including `packages.txt`) should have been
      auto-pushed to `shell-etc`.  Now, you only need to re-sync the servers:

      ```bash
      ansible-playbook --ask-become-pass sync.yml
      ```


### Making a configuration change

  1. Connect to any shell server as yourself

      ```bash
      ssh you@ny1.hashbang.sh
      ```

  2. Make and test any desired changes to files in /etc

      ```bash
      sudo vim /etc/some-config/file
      ```

  3. Commit changes via etckeeper

      ```bash
      sudo etckeeper commit -m 'updated some-config with some change'
      ```


### Sync packages/config across all servers

  1. Run Ansible playbook "sync"

      ```bash
      ansible-playbook --ask-become-pass -u your-sudo-user sync.yml
      ```
