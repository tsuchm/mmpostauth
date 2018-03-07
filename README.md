# Mailman PostAuth

This module provides an efficient spam protection method based on an
additional web-based confirmation for a open mailing list using
Mailman.

## Introduction

An open mailing list which accepts messages posted by non-members
still has several use cases.  For example, we keep an open mailing
list as the contact address of our office.  There are two reasons why
we employ a mailing list for the contact address instead of a simple
mail alias.  The first reason is necessity of the archive of the
messages posted to the contact address.  If a mail alias is employed
as the contact address, all messages posted to the contact address are
delivered into personal mailboxes of the office members, and a rookie
cannot see them.  At least, the archive of these messages is
necessarily for taking over duties between the office members.
The second reason is inter-operability between web-based non-mail
communication methods such as a bug tracking system, a chat system,
and a wiki system.  When you must refer a specific message posted to
the contact address from a non-mail communication method, the unique
URL of the message is necessary.  The Web-based archive of these
messages can provide it.

An open mailing list, however, has a big problem: it is weak against
mass-mailing of spammers.  In order to resolve this problem, this
module provides an additional web-based confirmation procedure to
protect an open mailing list using Mailman against messages posted by
bots.

This module mainly consists of two programs:

 * The controller CGI script: action
 * The moderation message upload script: mmpostauth-handler

The above two programs implements the following procedure:

 1. The upload script is employed to transfer a moderation message
	which is sent by Mailman when a non-member posts a message to the
	open mailing list.
 2. The controller CGI script generates a temporary confirmation URL
    and sends a mail which notifies it to the non-member who posted
    the original message.
 3. If someone accesses the confirmation URL, the non-member is
    considered as a real human.  After that, the controller CGI script
    sends an approval command message to Mailman, and asks it to
    deliver the original message.

## INSTALL

1) It is necessary to decide the base URL of this module.  If you can
access your own controller CGI as
https://www.example.net/mmpostauth/action, this URL is your base URL.

2) Install mod_fcgid and required perl modules.

    $ apt-get install libapache2-mod-fcgid libfcgi-perl libwww-perl libhtml-template-perl libcgi-session-perl libconfig-simple-perl

3) Confirm whether the controller CGI script works as a fcgid script.
If you see "OK" when you access the status URL (the base URL + `/status`),
the controller CGI script will work properly.

4) Confirm the regular expression which matches the server where the
moderation message upload script works.  The following example accepts
the servers whose IP addresses match 192.168.0.0/16.

    SetEnvIf Request_URI "/action/post$" post_action
    SetEnvIf Remote_Addr "^192\.168\." !post_action
    <Files action>
        SetHandler fcgid-script
        Order Allow,Deny
        Allow from all
        Deny from env=post_action
    </Files>

5) Put the moderator's password of the target open mailing list into
the configuration file `mmpostauth.ini` as follows:

    [testlist@lists.example.net]
    password = write_moderation_password_here

6) Add the moderation message upload script to the alias database `/etc/aliases` like:

    mmpostauth: |"/somewhere/mmpostauth-handler --url https://www.example.net/mailman/action/post"

This alias database entry means that all messages for `mmpostauth` are
transferred to the controller CGI script through the upload script.
You must give the right upload URL (the base URL + `/post`) to the
upload script with `--url` option.

7) Add the mail address which points the above alias database entry as
the moderator of the target open mailing list.

8) Send a test message to the target open mailing list.  You will
receive a notification message which contains the temporary
confirmation URL (the base URL + `/confirm/` + random sequence).
