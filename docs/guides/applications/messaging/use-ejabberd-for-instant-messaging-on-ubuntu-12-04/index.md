---
slug: use-ejabberd-for-instant-messaging-on-ubuntu-12-04
deprecated: true
author:
  name: Linode
  email: docs@linode.com
description: 'This guide shows how to use Ejabberd, a Jabber daemon written in the Erlang language, as an instant messaging service on the Ubuntu 12.04 Operating System.'
keywords: ["ejabberd", "ejabberd ubuntu", "ejabberd ubuntu 12.04", "ejabberd on linux", "real-time messaging", "xmpp", "collaboration software", "chat software", "linux jabber server", "instant messaging", "jabber daemon", "erlang"]
tags: ["ubuntu"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/communications/xmpp/ejabberd/ubuntu-12-04-precise-pangolin/','/applications/messaging/use-ejabberd-for-instant-messaging-on-ubuntu-12-04/','/applications/messaging/instant-messaging-services-with-ejabberd-on-ubuntu-12-04-precise-pangolin/']
modified: 2012-10-31
modified_by:
  name: Linode
published: 2012-10-31
title: 'Use ejabberd for Instant Messaging on Ubuntu-12-04'
external_resources:
 - '[Ejabberd Community Site](http://www.ejabberd.im/)'
 - '[XMPP Standards Foundation](http://xmpp.org/)'
 - '[XMPP Client Software](http://xmpp.org/software/clients.shtml)'
relations:
    platform:
        key: how-to-install-ejabberd
        keywords:
            - distribution: Ubuntu 12.04
---

Ejabberd is a Jabber daemon written in the Erlang programming language. It is extensible, flexible and very high performance. With a web-based interface and broad support for [XMPP standards](http://xmpp.org/), ejabberd is a great choice for a multi-purpose XMPP server. Ejabberd can be considered "heavyweight" by critics because of the requirements of the Erlang run-times. However, it is incredibly robust and can scale to support incredibly heavy loads. Ejabberd servers are believed to be the backbone for some of the largest Jabber servers running now.

This installation process assumes that you have a working installation of Ubuntu 12.04 (Precise Pangolin), have followed the steps in the [getting started](/docs/getting-started/) guide, and now have an up-to-date instance of the Ubuntu Precise Pangolin operating system. We also assume you are connected to your Linode via SSH as root. Once you've completed these requirements, we can begin with the installation process.

## XMPP/Jabber Basics

Although you can successfully run an XMPP server with only a passing familiarity of the way the XMPP network and system works, understanding the following basic concepts will be helpful:

-   The *JID* (or "Jabber ID"), is the unique identifier for a user in the XMPP network. It often looks like an email and contains the username that identifies a specific user on a server, the hostname that identifies the server, and a resource that identifies from where a given user is logged in. The resource is optional and is often safely omitted or ignored by most users. In the following example, "username" is the username, "example.com" is the hostname, and "/office" is the resource.

        username@example/office

    Again, the resource is optional; although XMPP allows a single JID to be connected to the server from multiple machines (i.e. resources), the resource adds a useful amount of specificity.

-   The XMPP system is federated by nature. Users with accounts on one server - if the server administrators allow it - can communicate with users on other servers. Without a centralized server, each XMPP server maintains the accounts and serves as the communication gateway for its own users. In the XMPP system there is no single point of failure; however, each server administrator can decide how his server is going to participate in the federated network. For instance, to federate with Google's "GTalk" XMPP network, server administrators need to have server-to-server (s2s) SSL/TLS encryption enabled, while other servers don't always require this.
-   XMPP takes advantage of ["SRV" DNS records](/docs/dns-guides/introduction-to-dns) to support the resolution of domains to the servers which provide DNS records.

## Install ejabberd

To install ejabberd and its required dependencies, issue the following command:

    apt-get install ejabberd

The default installation is complete and functional. The installation process creates a self-signed SSL certificate. If you want to use a commercially signed certificate, place the certificate file at `/etc/ejabberd/ejabberd.pem`. Most of the time a self-signed certificate is sufficient for many jabber applications.

If you have not already configured your `/etc/hosts` as follows, please do so before you continue. This will allow your Linode to associate its hostname with the public IP. Your file should have an excerpt that looks something like this (use your Linode's public IP address instead of 12.34.56.78):

{{< file "/etc/hosts" >}}
127.0.0.1    localhost.localdomain   localhost
12.34.56.78  username.example.com  username

{{< /file >}}


With the hostname configured, you're ready to begin configuring ejabberd.

## Configure ejabberd

Ejabberd's configuration files are written in Erlang syntax, which might be difficult to comprehend. Thankfully, the modifications we need to make are relatively minor and straightforward. The main ejabberd configuration file is located at `/etc/ejabberd/ejabberd.cfg`. We'll cover each relevant option in turn.

### Administrative Users

Some users will need the ability to administer the XMPP server remotely. By default this block of the config file looks like this:

{{< file "/etc/ejabberd/ejabberd.cfg" >}}
%% Admin user {acl, admin, {user, "", "localhost"}}.

{{< /file >}}


In Erlang, comments begin with the `%` sign, and the Access Control list segment contains information in the following form: `{user, "USERNAME", "HOSTNAME"}`. The following examples correspond to the users with the JIDs of `admin@example.com` and `username@example.com`. You only need to specify one administrator, but you can add more than one administrator simply by adding more lines, as shown below:

{{< file "/etc/ejabberd/ejabberd.cfg" >}}
{acl, admin, {user, "admin", "example.com"}}.
{acl, admin, {user, "username", "example.com"}}.

{{< /file >}}


All users specified in this manner have full administrative access to the server through both the XMPP and web-based interfaces. You will have to create your administrative users (as described below) before they can log in.

### Hostnames and Virtual Hosting

A single ejabberd instance can provide XMPP services for multiple domains at once, as long as those domains (or subdomains) are hosted by the server. To add a hostname for virtual hosting in ejabberd, modify the `hosts` option. By default, ejabberd is only configured to host the "localhost" domain:

{{< file "/etc/ejabberd/ejabberd.cfg" >}}
{hosts, ["localhost"]}.

{{< /file >}}


In the following example, ejabberd has been configured to host a number of additional domains. In this case, "username.example.com," "example.com," and "example.com."

{{< file "/etc/ejabberd/ejabberd.cfg" >}}
{hosts, ["username.example.com", "example.com", "example.com"]}.

{{< /file >}}


You can specify any number of hostnames in the host list, but you should be careful to avoid inserting a line break, which will cause ejabberd to fail.

### Listening Ports

TCP port number 5222 is the conventional "XMPP" port. If you want to change the port, this is the section of the configuration that needs to be modified.

Additionally, you may want to enable SSL access for client-to-server (c2s) SSL/TLS connections if you or the other users of you are using a client that requires legacy support for secured connections on port 5223. Uncomment the following stanza.

{{< file "/etc/ejabberd/ejabberd.cfg" >}}
{5223, ejabberd_c2s, [
    {access, c2s},
    {shaper, c2s_shaper},
    {max_stanza_size, 65536},
    tls, {certfile, "/etc/ejabberd/ejabberd.pem"}
 ]},

{{< /file >}}


### Additional Functionality

The `ejabberd.cfg` file is complete and well commented; from this point forward your server should run. However, you should take the time to become familiar with the options provided in this file.

By default, MUCs or Multi-User-Chats (chatrooms) are accessible on the "conference.[hostname]" sub-domain. If you want the public to be able to access MUCs on your domain, you need to create an "A Record" pointing the `conference` hostname (e.g. subdomain) to the IP address where the ejabberd instance is running.

## Using Ejabberd

Once installed, the use and configuration of ejabberd is uncomplicated. To start, stop or restart the server, issue the appropriate command from the following:

    /etc/init.d/ejabberd start
    /etc/init.d/ejabberd stop
    /etc/init.d/ejabberd restart

By default, ejabberd is configured to disallow "in-band-registrations," which prevents Internet users from getting accounts on your server without your consent. To register a new user, issue a command in the following form:

    ejabberdctl register username example.com man

In this example, `username` is the username, `example.com` is the domain, and `man` is the password. This will create a JID for `username@example.com` with the password of "man." Use this form to create the administrative users specified above.

To remove a user from your server, issue a command in the following form:

    ejabberdctl unregister username example.com

The above command would unregister the `username@example.com` account from the server.

To set or reset the password for a user, issue the following command:

    ejabberdctl set-password username example.com morris

This command changes the password for the `username@example.com` user to `morris`.

To back up ejabberd's database, issue the following command:

    ejabberdctl dump ejabberd-backup.db

This command dumps the contents of the internal ejabberd database into a file located in the "/var/lib/ejabberd/" directory. To restore from the backup, issue the following command:

    ejabberdctl load ejabberd-backup.db

For more information about the `ejabberdctl` command, issue `ejabberdctl help` or `man ejabberdctl`.

If you would prefer to administer your ejabberd instance via the web-based interface, log in to `http://example.com:5280/admin/`, where "example.com" is the domain where ejabberd is running. Log in with the full JID using the username and the password of one of the administrators specified in the `/etc/ejabberd/ejabberd.cfg` file.

## XMPP Federation and DNS

To ensure that your ejabberd instance will federate properly with the rest of the XMPP network, particularly with Google's "GTalk" service (i.e. the ["@gmail.com](mailto:"@gmail.com)" chat tool), we must set the SRV records for the domain to point to the server where the ejabberd instance is running. We need three records, which can be created in the DNS Management tool of your choice:

1.  Service: `_xmpp-server` Protocol: TCP Port: 5269
2.  Service: `_xmpp-client` Protocol: TCP Port: 5222
3.  Service: `_jabber` Protocol: TCP Port: 5269

The "target" of the SRV record should point to the publicly routable hostname for that machine (e.g. "username.example.com"). The priority and weight should both be set to `0`.

## Troubleshoot

The logs for ejabberd are located in the `/var/log/ejabberd/` directory. If you're getting error messages look in these files, particularly `ejabberd.log` and `sasl.log`. Additionally, if ejabberd crashes, the "image dump" of Erlang will be saved in this directory. Begin your investigations for error messages in these files.

Furthermore, ejabberd's "Mnesia" database is stored in the `/var/lib/ejabberd/` directory. If you think the database has become corrupted, delete the files in this directory (e.g. `rm /var/lib/ejabberd/*`) and reload from a backup, if necessary. This is sometimes required if the hostname of the local machine changes.
