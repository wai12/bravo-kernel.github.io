title: How to install Watchman on Debian
tags:
  - debian
  - ember
categories:
  - Debian
date: 2016-11-22 12:54:38
---

To globally install Watchman on Debian:

```bash
sudo apt-get install build-essential
sudo apt-get install python-dev
sudo apt-get install automake
sudo apt-get install autoconf

cd /tmp
git clone https://github.com/facebook/watchman.git
cd watchman
git checkout v4.7.0  # or whatever is now the latest stable release

./autogen.sh
./configure --enable-statedir=/tmp
sudo make
sudo make install
sudo mv watchman /usr/local/bin/watchman
```

> Please note that the `--enable-statedir` option must be set on
> Debian systems to prevent Watchman startup failures due to
> non-existing default location `/usr/local/var/run/watchman`.

To make sure things went well:

```bash
cd ~
which watchman
# should display `/usr/local/bin/watchman`

watchman -v
# should display the installed version e.g. `4.7.0`

watchman watch .
# should display something similar to:
# {
#    "version": "4.7.0",
#    "watch": "/home/vagrant",
#    "watcher": "inotify"
# }

watchman watch-del-all
watchman shutdown-server
```

Closing notes:

- Watchman state can be found in `/tmp/<username>-state/`
- Ember users should no longer see annoying Watchman related messages
