---
layout: post
title:  "Preparing a Debian server for Kamal deployments"
tags:   kamal rails debian
---

This is not so much a _TIL_, but more a _TI summarized what I've learned from months of Kamal deployments_.

I usually deploy to Debian-based servers, that's why this note works on the premise of a Debian server.

It should be noted that I don't frequently deploy to new servers, I'm doing it occasionally for side projects. If the frequency was higher I'd probably look at something like [Ansible](https://docs.ansible.com/) or [Terraform](https://developer.hashicorp.com/terraform) for automating such a process. For my current needs, this little guide is more than enough when I need to prepare a new server.

## First steps

For accessing the server I want to use an SSH key, instead of the default root login/password that often comes when renting out a server. I copy my public SSH key to the server:

{% highlight bash %}
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@demo-server-ip
{% endhighlight %}

That should enable me to access the server via:

{% highlight bash %}
ssh root@demo-server-ip
{% endhighlight %}

Once there, it doesn't hurt to immediately update all installed packages:

{% highlight bash %}
sudo apt-get update && sudo apt-get upgrade
sudo apt dist-upgrade
{% endhighlight %}

Kamal needs Docker, so I'm installing it right away, following the [official guide for Debian](https://docs.docker.com/engine/install/debian/).

## Changing to a dedicated user

It's a good idea to create a new user other than root.

First, I create a new user (replace name with desired name):

{% highlight bash %}
adduser deploy
{% endhighlight %}

All additional details can be left blank, I just make sure I store the new chosen password safely, e.g. in a password manager.

Next, the new user should be added to the sudo and the docker group:

{% highlight bash %}
usermod -aG sudo deploy
usermod -aG docker deploy
{% endhighlight %}

In a new terminal window, I copy the public SSH key for my new user as well. That should enable me to SSH into the server as the deploy user.

{% highlight bash %}
ssh-copy-id -i ~/.ssh/id_ed25519.pub deploy@demo-server-ip
{% endhighlight %}

## Changing the SSH config

Before making any changes to my SSH config, I save a backup of the config, in case I need to return to the default state:

{% highlight bash %}
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
{% endhighlight %}

I then change my SSH config via:

{% highlight bash %}
sudo nano /etc/ssh/sshd_config
{% endhighlight %}

The important bits to add/change:

{% highlight plain %}
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
{% endhighlight %}

Before restarting the SSH service, I make sure I'm  logged into the server in another terminal, in case the config causes issues. Then I restart the SSH service:

{% highlight bash %}
sudo systemctl restart ssh
{% endhighlight %}

On a sidenote, it's possible to tail SSH logs via:

{% highlight bash %}
sudo journalctl -fu ssh
{% endhighlight %}

After this change it should still be possible to get SSH access with the deploy user's key. It shouldn't be possible to a) log in as the root user and b) log in with a password authentication.

## Security updates

Next, I want to make sure the server stays up-to-date and receives all relevant security updates.

If not yet installed, I install `unattended-upgrades`:

{% highlight bash %}
sudo apt install unattended-upgrades
{% endhighlight %}

I create a file that will hold my configs (I avoid overwriting existing config files, as they may be overriden by updates):

{% highlight bash %}
sudo touch /etc/apt/apt.conf.d/51myunattended-upgrades
{% endhighlight %}

I like to use the content from [this example](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server?tab=readme-ov-file#debian-based-systems) and copy it to my config file. That makes sure that packages are kept up-to-date without manual work.

## That's all

... for now. I'm not adding stuff like [UFW](https://en.wikipedia.org/wiki/Uncomplicated_Firewall) or [Fail2Ban](https://en.wikipedia.org/wiki/Fail2ban) because my assumption is that I'm only opening ports that are supposed to be open and therefore UFW is not the right direction. Also, UFW gives kind of a false feeling of security [when used together with Docker](https://docs.docker.com/engine/network/packet-filtering-firewalls/#docker-and-ufw) (which is the case for Kamal deployments).

## Helpful resources

I obviously didn't come up with these ideas myself. Instead I compiled what works for my needs from the following resources:

- https://community.netcup.com/en/tutorials/first-steps-to-protect-your-linux-server-against-common-attacks
- https://github.com/imthenachoman/How-To-Secure-A-Linux-Server
- https://kamalmanual.com/provisioning/

