+++
date = '2025-10-22'
draft = false
title = 'Secure Unraid Offsite Backup With Restic, Backrest, and Tailscale.'
description = 'Learn how to set up secure, encrypted offsite backups between two Unraid servers using Restic, Backrest, and Tailscale — all without exposing any services to the web.'
categories = ['Homelab']
tags = ['Backup', 'Security', 'Tailscale', 'Unraid', 'Docker', 'Tutorial', 'Self Hosting']
+++
{{< lead >}}
No exposed services needed!
{{< /lead >}}

I'm finally adhering to the **3-2-1 backup rule** by implementing the "1" part, which is the offsite backup!

My friend and I, who both run Unraid servers, have managed to do this using Restic + Tailscale which gives us encrypted, incremental offsite backups on each other's servers, **without exposing anything to the web**.

It was also surprisingly easy to implement thanks to Unraid 7.0+'s support for **adding tailscale directly to your containers** - which means that you can share individual containers with others via tunnelling.

---

## Setup

All containers below can be found as templates in Unraid's Community Applications.

### [Restic rest-server](https://github.com/restic/rest-server)

A HTTP server implementing restic's backend API. Each of us has their own instance of this running, and we send our restic backups here.

#### Docker and Tailscale Configuration

Install the container, map its `/data` directory to a share where you want to store the incoming restic repositories, and ensure the `--private-repos` option flag is set.

By setting the flag above, you ensure that each user connecting to the server can only access repos in their dedicated folder.

Enable Tailscale for the container (use the Tailscale toggle in update container menu, Unraid 7.0 and above) and add it to your tailnet. 

Share this container via tailscale with your friend, and add theirs to your tailnet.

![Container Settings](restic_rest_server_settings.png "Container Settings")

#### Authentication

The container will create a `.htpasswd` file in the data directory.
Inside, you should specify a username and a password in the following format:

```
<username>:<bcrypt-hashed-password>
```
<sup>Omitting < and ></sup>

Create a secure plaintext password, run it through bcrypt, and send the bcrypt hash output to your friend. 

Save your plaintext password securely for later use.

{{< alert >}}
**Warning!** Do not use any special characters that backrest may confuse for URL escape sequences.
{{< /alert >}}

---

### [Backrest](https://github.com/garethgeorge/backrest)

Backrest is a wrapper for restic, which gives you a nice GUI and handles:
- Repo creation and management including health checks and pruning.
- Backup scheduling.
- Restoring.

It also has notification agents such as a Discord webhook message if a backup goes wrong.

![Backrest](featured.png "Backrest GUI")


#### Docker and Tailscale Configuration
Configure the container by mapping all the directories defined in the app template to your preference.

The most important step here is to also enable Tailscale for this container so it can talk to the restic rest-server that has been shared with you. 

---

### Bringing It All Together

So by now you and your backup buddy should each have one of:

- A **restic rest-server**, shared to each other's tailnets.
- A **.htpasswd** file with your friend's username and hashed password used by the rest-server.
- An instance of **backrest** added to your own tailnet only.

Because your backrest container is in your tailnet, and your friend's restic server has also been shared in — they can see and connect to each other!

You can now create and add a repo in backrest with the following URL:

```
rest:http://<username>:<password>@<tailscale-name/IP-of-rest-server>:8000/<username>/<repo_name>
```
<sup>Omitting < and ></sup>

Let's break this down:
- `http` will be used unless you wish to provision certs for the rest-server. This however should be safe enough as: 
  - You are using tailscale as a VPN tunnel, so this data is never on the web.
  - If, and you should have, set a repository password, restic will encrypt the data before sending it.

- `<username>` should match what your friend put in their **.htpasswd** file.

- `<password>` is your plaintext password you created earlier. **Not the bcrypted hash.**

- `<tailscale-name/IP-of-rest-server>` self explanatory. If you have issues with the Magic DNS domain name, just use the tailscale container IP.

- `<repo_name>` name of the repo folder which will be created on your friend's drive.

As an example:
```
rest:http://michal_testing:plaintext_pass123@restic-rest-server.random-words.ts.net:8000/michal_testing/my_repo
```

Will create a repository in the folder that is mapped to the rest-server's `/data` directory, with the full path being `/data/michal_testing/my_repo`

And that's it! Safe and incremental offsite backups!