# Q8. How to install postfix on ubuntu?

## Introduction:

Postfix is a popular open-source Mail Transfer Agent (MTA) that can be used to route and deliver email on a Linux system. It is estimated that around 25% of public mail servers on the internet run Postfix.

Originally written in 1997 by Wietse Venema at the IBM Thomas J. Watson Research Center in New York, and first released in December 1998[3], Postfix continues as of 2019 to be actively developed by its creator and other contributors. The software is also known by its former names VMailer and IBM Secure Mailer.


## Prerequisites:
	1. ubuntu 18.04 installed
	1. A fully qualified qualified domain name (FQDN)


##Installation:
Postfix repositories are available via the default package manager provided by ubuntu. So we can directly install using the following command.

	sudo apt install postfix

This command should trigger the installation process where you will be prompted to continue with the installation by pressing 'Y' or 'N' otherwise.

Once you continue, you will be prompted to select the postfix configuration which suits your needs.

The options specified and their meanings are.
	1. No Configuration
This specifies that no new configuration should be created or the existing configuration if any should be left intact.
	1. Internet Site
This option is used to send and receive email over the internet using Simple Mail Transfer Protocol (SMTP)
	1. Internet with Smarthost
This option receives email via SMTP but sends via another server. So our host acts as a relay.
	1. Satelite system
This option will configure postfix to send as well as receive email using a different server. 
	1. Local Only
This configuration is used when mail is required to be delivered to users on the same system only and no network is required.

For our setup we will be using Internet site so as to be able to send and receive email via internet from the host itself.

Next step requires setting up the domain name as the mail name which will act as the end point for identifying and routing our mail to this host server.
So, the domain should be pointing to our server. If example.com is the domain name we will set it up here as it is without any qualifiers. This will facilitate the building of email addresses for this domain as mail@example.com and so on.

For now, this is sufficient to complete the installation. Once the instalation is complete we can check if the postfix service is up and running using the below command

	netstat -ntlp

In the results, we will be able to see port 25 is open and listening. 

You can change the settings at any point of time by editing the setting at /etc/postfix/main.cf

We can create virtual email addresses and then map them to the local linux.
This can be created as below.
	1. Create file /etc/postfix/virtual
	1. Add email addresses followed by a space and then by the username of the local linux account. You can add multiple such lines for each email address.
	1. Run sudo postmap /etc/postfix/virtual to add the mapping file to postfix config
	1. Restart postfix server using sudo service postfix restart.

## Test Setup
To test you setup use the below command by replacing the mail address by the one specified in you /etc/postfix/virtual file.

	echo "test "  | mail -s 'Test email subject line' ubuntu@mail.mydomain.com

We can check the received email by using the "mail" command on  the terminal.
