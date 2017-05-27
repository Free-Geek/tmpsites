## What

This is a server-based platform that creates temporary, disposable web hosting accounts, with individual SFTP access.
This is very useful for a classroom situation for teaching the basics of web development.

The web accounts are automatically deleted after 3 hours.

This has only been tested with Amazon EC2, with an Ubuntu 14.04 "tiny" instance.

## How it works

1. `scripts/remote/provision` provisions a server with the necessary base software and configuration.
2. `scripts/remote/add_sites` creates user accounts and corresponding web configurations.
  Then it prints the usernames and their generated passwords for distribution.

The usernames of the accounts created will have the form "siteN", where N is a number
starting from 1.

The web virtual hosts will be configured with the hostname "siteN.example.com",
corresponding to each user account.

## Instructions

### Using Amazon EC2

Amazon EC2 is the only tested provider at this time.

Create a tiny instance with Ubuntu 14.04. By default, it will have very limited access,
based on your IP address at the time of creation. This will not work
for the purpose of Weblearn. Here's how to open access to the world:

* Add a new security group, named 'all-access'
* Grant TCP access to port 22 and 80 to all IP addresses
* Assign the instance to the new security group

### Other providers

Making this work with other providers should be no problem, as long as the
OS distribution is Ubuntu 14.04. Make sure that the default account on the server can run unlimited sudo commands without a password.

Unless your provider already does so for you, you may need to generate an
SSH keypair, and upload the public key to the server.

### DNS Configuration

You will need to configure DNS entries for every hostname you expect to use.
Let's say you have 6 students in a class, and you want to provide a hosting
account for each. You'll generate accounts and web configurations that will
support hostnames like this:

* site1.example.com
* site2.example.com
* site3.example.com
* site4.example.com
* site5.example.com
* site6.example.com

You'll need to log into your DNS provider, which will either be your domain provider,
or your web hosting provider, and configure the DNS entries for the domain name
you're going to use.

It's probably best set it up this way:

1. Create one 'A' record that acts as an anchor for all the sites, site1 through siteN:
  * Host: site.example.com
  * IP Address: (your server IP address)
2. For each of the site hostnames, create a CNAME record:
  * Host: siteN (where N is a number starting from 1)
  * Points to: site.example.com

### Configure

Copy the file `config.sample` to `config`:

```shell
cp config.sample config
```

Edit the file and change the settings:

`root_hostname`:

Let's say one of the sites generated is `site1.example.com`. The root hostname
for that would be `example.com`.

`ssh_remote`:

This is the server's admin username and server's default hostname,
joined with a '@'. This is used as part of an SSH command or Rsync.

### Create the private_key file

When the EC2 instance was created, it prompted you to download a PEM file.
This file is an SSH private key used to login to the server via SSH without
a password.

Write the private key to `private_key`. Change the permissions of it to 0600:

```shell
chmod 600 private_key
```

### Provision the server

Run the following:

```shell
./scripts/remote/provision
```

This installs the dependencies, such as nginx and mysecureshell,
and configures them.

It also installs the crontab that regularly checks for web hosting
accounts that have expired, and it deletes them.

### Add sites

Run the following:

```shell
./scripts/remote/add_sites 2
```

The above command adds 2 sites to the server.

Now visit: site1.example.com, or whatever your first site is called.

You should see a webpage with the words "Welcome to site1.example.com!"
