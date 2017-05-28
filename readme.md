## What

`tmpsites` is a tool for creating temporary, disposable web hosting accounts.
It is useful for a classroom situation, such as when teaching the basics of web development.

Each web account has a hostname, SFTP access, and a generated password.
Accounts are automatically deleted after 3 hours or a configurable timeframe.

The tool consists of client-side commands and server code.
Everything is controlled from the client-side commands.
The client commands have been tested on macOS 10.12.
The server code has been tested on an Amazon EC2 "micro" instance with Ubuntu 16.04.

## How it works

1. The `client/provision` command provisions a server with the necessary base software and configuration.
2. The `client/add_accounts` command creates user accounts and corresponding web configurations.
  Then it prints the usernames and their generated passwords for distribution.

The usernames of the accounts created will have the form "siteN", where N is a number
starting from 1. For example: `site1`, `site2`, `site3`.

The web virtual hosts will be configured with the hostname "siteN.example.com",
corresponding to each user account. For example: `site1.example.com`, `site2.example.com`,
`site3.example.com`.

## Instructions

### Clone this repo

```shell
git clone https://github.com/moxley/tmpsites.git
cd tmpsites
```

### Using Amazon EC2

Amazon EC2 is the only tested provider at this time. Other providers may work too (see below).

Create a micro instance with Ubuntu 16.04. Complete all the
steps of the creation process, rather than skipping to
the end.

By default, it will have very limited network access,
based on your IP address at the time of creation. If you use the creation wizard
to create the instance, you will have the opportunity near the end of the wizard
to open ports 22 and 80 to all address. If you didn't do that, after the fact:

* In the EC2 console left navigation menu, find "Network & Security", and select "Security Groups".
* Add a new security group, named 'all-access'
* Grant TCP access to port 22 and 80 to all IP addresses
* Go back to the EC2 instances list, and select your instance. Under the "Actions" dropdown,
  select "Networking", and then "Change Security Groups".
* Assign the instance to the new security group

At the end of the instance creation, you will be prompted to either provide an SSH public key or
download a generated PEM file. If you're not sure, just download the PEM file.

### Other providers

Making this work with other providers shouldn't be much problem, as long as the
OS distribution is Ubuntu 16.04 or similar.
Make sure that the default account on the server can run unlimited sudo commands without a password.

You will also need to have an SSH key-pair in place,
as described [here](http://www.linuxproblem.org/art_9.html).

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

The advantage to using CNAME records is if you switch to
a different server with a different IP address, you only
need to change the one 'A' record.

### Configure

Copy the file `config.sample` to `config`:

```shell
cp config.sample config
```

Edit the file and change the settings:

`root_hostname`: Let's say the site hostnames are `site1.example.com`, `site2.example.com`, `site3.example.com`, etc. The `root_hostname`
would be `example.com`. Of course, you will need to have `example.com` registered.

`ssh_remote`: This is the server's admin username and server's default hostname,
joined with a '@'. This is used as part of an SSH command or Rsync.

`num_accounts`: This is the number of web accounts to create.

`accounts_ttl`: Amount of time the website accounts have to live. The format of this setting
is exactly the format used in the `date` command to calculate date arithmetic.

### Create the private_key file

When the EC2 instance was created, it prompted you to download a PEM file.
This file is an SSH private key used to login to the server via SSH without
a password.

Write the private key to a file called `private_key`, in this directory. Change the permissions of it to 0400:

```shell
chmod 400 private_key
```

### Provision the server

Run the following:

```shell
client/provision
```

This installs the dependencies, such as
[nginx](https://www.nginx.com/resources/wiki/)
and [mysecureshell](https://mysecureshell.readthedocs.io/en/latest/),
and configures them.

It also installs a [crontab](https://www.computerhope.com/unix/ucrontab.htm) command that regularly checks for web hosting
accounts that have expired, and it deletes them.

### Add web accounts

Run the following:

```shell
client/add_accounts
```

The above command adds sites to the server. The number of sites is
dependent on `num_accounts`, in the `config` file.

Now visit: `site1.example.com`, or whatever your first site is called.

You should see a webpage with the words "Welcome to site1.example.com!"

If there were sites already created before, they will be deleted to make
way for the new ones.

### Remove web accounts

The web accounts will be removed automatically, after 3 hours, or after whatever `accounts_ttl` is set to, but if you
would like to remove them right away, run this:

```shell
client/remove_accounts
```
