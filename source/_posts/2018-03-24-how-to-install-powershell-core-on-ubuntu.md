title: How to install Powershell Core on Ubuntu
date: 2018-03-24 18:00:00
tags:
  - ubuntu
  - powershell
  - powershell-core
categories:
  - devops
---

To install Powershell Core on Ubuntu:

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list | sudo tee /etc/apt/sources.list.d/microsoft.list
sudo apt-get update
sudo apt-get install -y powershell
```

To verify the installation completed successfully:

1. Start Powershell by typing `pwsh`
2. Type `Get-` and press `TAB`

If things went well you should see a lot of familiar CmdLets ready for use on your Linux server ;)

{% asset_img running-powershell-core.png 'Powershell CmdLets on your Ubuntu Server' %}