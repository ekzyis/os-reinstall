# OS reinstall steps: vps@delta-networks.de - ubuntu-18.04-x86_64

### 1. User setup


Login as root (Since perhaps your local config still uses a non-standard ssh port for the vps, specify the port manually):
```sh
ekzyis@local> ssh root@vps -p 22
```

Create user with home directory (-m):
```sh
root@vps> useradd -m ekzyis
```

Add user to sudo group
```sh
root@vps> usermod -aG sudo ekzyis
```

Login as user:
```sh
ekzyis@host> ssh vps -p 22
```

Set login shell with `chsh`.


### 2. Install git

Login as user:
```sh
ekzyis@local> ssh vps -p 22
```

Add personal package archive for git to get latest stable upstream version of git. We first need to install `software-properties-common` because the command `add-apt-repository` is included in it.
```sh
ekzyis@vps> sudo apt-get install software-properties-common
ekzyis@vps> sudo add-apt-repository ppa:git-core/ppa
```

Install git:
```sh
ekzyis@vps> sudo apt-get install git
```

### 3. Setup Github access with public key

Create ssh key pair:
```sh
ekzyis@vps> ssh-keygen -t rsa -b 4096
```

Add public key to your Github account.

### 4. Setup ssh

##### 4.1 Switch ssh port

Switch ssh port to 55680:
```sh
ekzyis@vps> sudo sed -ie 's/#Port 22/Port 55680/'
ekzyis@vps> sudo systemctl restart ssh
```

Check if ssh connection still works:
```sh
ekzyis@local> ssh vps -p 55680
```

Optional: Add/Update ssh config entry with new port.

Check if ssh connection works without specifying port:
```
ekzyis@local> ssh vps
```

For now, I will no longer specify the ssh port.

##### 4.2 Setup public key authentication with vps

```sh
ekzyis@local> ssh-copy-id vps
```

To disable pasword authentication, set following properties in /etc/ssh/sshd_config of vps and restart ssh daemon:

- PasswordAuthentication no
- ChallengeResponseAuthentication no
- UsePAM no

```sh
ekzyis@vps> sudo vim /etc/ssh/sshd_config
ekzyis@vps> sudo systemctl restart ssh
```

##### 4.3 Setup endlessh

Build endlessh:
```sh
ekzyis@vps> git clone git@github.com:skeeto/endlessh
ekzyis@vps> sudo apt-get install libc6-dev build-essential
ekzyis@vps> cd endlessh && make
```

Setup endlessh:
```sh
ekzyis@vps> sudo mv endlessh /usr/local/bin/
ekzyis@vps> which endlessh  # confirm it was installed
```

Setup endlessh service:
```sh
ekzyis@vps> sudo cp util/endlessh.service /etc/systemd/system/
ekzyis@vps> sudo systemctl enable endlessh
```

Open the endlessh.service file and follow instructions there to enable endlessh binding on ports < 1024.

Then create endlessh config:
```sh
ekzyis@vps> sudo mkdir -p /etc/endlessh
ekzyis@vps> echo "Port 22" | sudo tee config
```

And start it:
```sh
ekzyis@vps> sudo systemctl start endlessh
```

### 5. Install gitolite

Create git user and set its password:
```sh
ekzyis@vps> sudo useradd -m git
ekzyis@vps> sudo passwd git
```

Copy the public key of the gitolite admin to the server. Since most likely you want to be the gitolite admin, you need to copy your own public key.

```sh
ekzyis@local> scp ~/.ssh/id_rsa.pub vps:gitolite-admin.pub
ekzyis@local> ssh vps
ekzyis@vps> sudo mv gitolite-admin.pub /home/git/
ekzyis@vps> sudo chown git:git /home/git/gitolite-admin.pub
```

Install gitolite:

https://gitolite.com/gitolite/quick_install.html

```sh
ekzyis@vps> sudo su - git
git@vps> mkdir -p ~/bin
git@vps> git clone https://github.com/sitaramc/gitolite
git@vps> gitolite/install -ln ~/bin
git@vps> gitolite setup -pk gitolite-admin.pub
```

Check if gitolite setup worked. You should be greeting with "hello gitolite-admin":
```sh
ekzyis@local> ssh git@vps info
```

(Optional) Add old repositories:

https://gitolite.com/gitolite/basic-admin.html#appendix-1-bringing-existing-repos-into-gitolite

Move old, bare repositories into $HOME/repositories:
```sh
git@vps> mv <REPOSITORIES> $HOME/repositories
gite@vps> chown -R gitolite:gitolite $HOME/repositories  # make sure correct permissions are set
```

Then run these three commands:
```sh
git@vps> gitolite compile
git@vps> gitolite setup --hooks-only
git@vps> gitolite trigger POST_COMPILE
```

Check if you now have access to all previous repositories:
```sh
ekzyis@local> ssh git@vps info
```

If not, force push your backup of the gitolite-admin repo (you have one, right?):
```sh
ekzyis@local> cd gitolite-admin
ekzyis@local> git push -f
```

This should update the gitolite-admin on the remote thus the next command should show all repositories:
```sh
ekzyis@local> ssh git@vps info
```

### 6. Install docker

Docker installation: https://docs.docker.com/engine/install/ubuntu/
Docker access without sudo: https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user

Don't forget to configure docker to start on boot:
```sh
$ sudo systemctl enable docker
```
