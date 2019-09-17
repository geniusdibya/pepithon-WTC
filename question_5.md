# Question 5: How to Set up SPF and DKIM with Postfix on Ubuntu 18.04?

## Introduction to DKIM:
Domain Key Identified Mail (DKIM) is a method of signing electronic emails using public-private key pair. DKIM is used by receiving mail server for identifying authenticity, that they are sent by authorized mail servers on behalf of the domain. It also minimizes the possibility of emails getting marked as SPAM.

# Prerequisites:
1. Ubuntu Server 16.04 and above.
1. Postfix server installed and configured.

## Installation
We will need two pakages to be installed which can be installed with a single command as below.
	sudo apt-get install opendkim opendkim-tools

## DKIM Configuration
1. Generate key pair
We can create DKIM key-pair using the *opendkim-genkey* command. Execute the following commands in order to create the proper folder structure for the keys to be stored.

	$ MYDOMAIN=example.com
	$ mkdir -p /etc/opendkim/keys/$MYDOMAIN
	$ cd /etc/mail/opendkim/keys/$MYDOMAIN
	$ opendkim-genkey -t -s mail -d $MYDOMAIN

This will generate two files, viz., mail.private & mail.txt

1. Add postfix user to opendkim group
	sudo gpasswd -a postfix opendkim

1. Edit OpenDKIM main config file /etc/opendkim.conf and make the following changes.
	1. Uncomment the following lines
		Canonicalization   simple
		Mode               sv
		SubDomains         no
	1. Replace _simple_ with _relaxed/simple_
	1. Add the following lines if not present otherwise uncomment them and update values as given below.
		AutoRestart         yes
		AutoRestartRate     10/1M
		Background          yes
		DNSTimeout          5
		SignatureAlgorithm  rsa-sha256
	1. Append the following lines to the end of the file
		#OpenDKIM user
		# Remember to add user postfix to group opendkim
		UserID             opendkim

		# Map domains in From addresses to keys used to sign messages
		KeyTable           refile:/etc/opendkim/key.table
		SigningTable       refile:/etc/opendkim/signing.table

		# Hosts to ignore when verifying signatures
		ExternalIgnoreList  /etc/opendkim/trusted.hosts

		# A set of internal hosts whose mail should be signed
		InternalHosts       /etc/opendkim/trusted.hosts
1. Create key table, signing table & trusted host file as mentioned in the config above.
	sudo mkdir /etc/opendkim
	sudo mkdir /etc/opendkim/keys

Change owner for these files from root to opendkim so that opendkim user can read/write to this directory

	sudo chown -R opendkim:opendkim /etc/opendkim
	sudo chmod go-rw /etc/opendkim/keys

Create the signing table 
	sudo nano /etc/opendkim/signing.table
Add the below line to this file.
	*@example.com		default._domainkey.example.com

Create the key table
	sudo nano /etc/opendkim/key.table
Add the following line to the key table
	default._domainkey.example.com    example.com:default:/etc/opendkim/keys/example.com/default.private

Next, create the trusted host file.
	sudo nano /etc/opendkim/trusted.hosts
Add the following files to this file.
	127.0.0.1 
	localhost 
	*.example.com

This specifes that message coming from the above will be signed and trusted.

1. Add public key to you DNS record
	sudo cat /etc/opendkim/keys/example.com/default.txt

The string in p parameter without the quotes is the public key. Add this string to the DNS record of example.com by creating a TXT record

1. Test your configuration
Enter the following command to test your key.
	sudo opendkim-testkey -d your-domain.com -s default -vvv

If everything is ok, you will get the below message.
	key OK

1. Connecting postfix to opendkim
Postfix can talk to OpenDKIM via a Unix socket file. 
Create a directory to hold the OpenDKIM socket file and only allow opendkim user and postfix group to access it.

	sudo mkdir /var/spool/postfix/opendkim

	sudo chown opendkim:postfix /var/spool/postfix/opendkim
 
Then edit the socket configuration file.

	sudo nano /etc/default/opendkim


Find the following line:
	SOCKET="local:/var/run/opendkim/opendkim.sock"
and replace it with

	SOCKET="local:/var/spool/postfix/opendkim/opendkim.sock"
 
Next, we need to edit Postfix main configuration file /etc/postfix/main.cf and add the following lines smtpd_recipient_restriction section.

	# Milter configuration
	milter_default_action = accept
	milter_protocol = 6
	smtpd_milters = local:/opendkim/opendkim.sock
	non_smtpd_milters = $smtpd_milters
 
Now restart postfix and opendkim
	sudo service postfix restart
	sudo service opendkim restart
 
## Introduction to SPF
Sender Policy Framework (SPF) record specifies which hosts or IP addresses are allowed to send emails on behalf of a domain. You should allow only your own email server or your ISP’s server to send emails for your domain.

## Create a SPF record in your DNS
In your DNS management interface, create a new TXT record like below.
	TXT  @   v=spf1 mx ~all

Here, v=spf1 stands for SPF record version 1 and mx ~all means that all hosts listed in mx record are allowed to send email on behalf of example.com and all other hpsts are disallowed from sending. 

## Testing SPF & DKIM setup
Try sending an email from this server to your gmail account to check if SPF & DKIM checks are passed.

Open the email and on the right side click on Show original button from the dropdown menu to see the authentication details.
