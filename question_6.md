# Question 6: How to send emails from php mail function using a SMTP server.

## PHP has a built in mail function which is used to send email from PHP scripts. This function comes very handy for setting up automated alerts or transactional emails.

The basic syntax of the mail function is as follows.

	mail(to, subject, message, headers, parameters);

| Parameters | Type | Description |
|---|---|---|
| to | Required | Email addresses for the receiver(s).
| subject | Required | Subject line for the email
| message | Required | Text or html message to be sent |
| headers | Optional | Specifies headers like From, Cc, Bcc seperated by CRLF (\r\n) |
| parameters | Optional | Additional parameters to the sendmail program like envelope sender address and others. |

You can include HTML into message parameter also by adding the following to the headers.
	$headers = 'MIME-Version: 1.0 '."\r\n";
	$headers .= 'Content-type: text/html; charset=iso-8859-1'."\r\n";


## Sending mail routed through a SMTP server
PHP mailer uses Simple Mail Transmission Protocol (SMTP) to send mail.
The SMTP mail settings can be configured from php.ini file in the installation folder. Locate the php.ini file and open it for editing.
Find the following in this file
	mail function

Locate the parameter SMTP and assign it the name of the smtp server.
	SMTP=smtp.example.com
Then, uncomment the line smtp_port and assign it the port number used for SMTP on the server.

	smtp_port=25

If the server requires authentication, then add the following lines
	auth_username=mail@example.com
	auth_password=example_password
	
After saving the changes to the ini file, restart apache server for the changes to take effect.

## A simple php mail example is given below.
```php
<?php

	$to="user@example.com";
	$subject="First php mail";
	$mesage="This is a test email sent from php mail function."
	$headers="From: noreply@example.com";
	mail($to, $subject, $message, $headers);
?>
```
