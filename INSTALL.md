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


### 2. Install `git`

Login as user:
```sh
ekzyis@local> ssh vps -p 22
```

Add personal package archive for `git` to get latest stable upstream version of `git`. We first need to install `software-properties-common` because the command `add-apt-repository` is included in it.
```sh
ekzyis@vps> sudo apt-get install software-properties-common
ekzyis@vps> sudo add-apt-repository ppa:git-core/ppa
```

Install git:
```sh
ekzyis@vps> sudo apt-get install git
```

