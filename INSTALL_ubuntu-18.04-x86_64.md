# OS reinstall steps: ubuntu-18.04-x86_64

### 1. User setup

Login as root:
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

##### 4.2 Setup public key authentication with vps

```sh
ekzyis@local> ssh-copy-id -p 55680 vps
```

To disable pasword authentication, set following properties in /etc/ssh/sshd_config of vps and restart ssh daemon:

PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

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
