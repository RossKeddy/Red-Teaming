https://github.com/jetmore/swaks
https://serversmtp.com/smtp-error/
---
The `Simple Mail Transfer Protocol` (`SMTP`) is a protocol for sending emails in an IP network. It can be used between an email client and an outgoing mail server or between two SMTP servers. SMTP is often combined with the IMAP or POP3 protocols, which can fetch emails and send emails. In principle, it is a client-server-based protocol, although SMTP can be used between a client and a server and between two SMTP servers. In this case, a server effectively acts as a client.

By default, SMTP servers accept connection requests on port `25`. However, newer SMTP servers also use other ports such as TCP port `587`. This port is used to receive mail from authenticated users/servers, usually using the STARTTLS command to switch the existing plaintext connection to an encrypted connection. The authentication data is protected and no longer visible in plaintext over the network. At the beginning of the connection, authentication occurs when the client confirms its identity with a user name and password. The emails can then be transmitted. For this purpose, the client sends the server sender and recipient addresses, the email's content, and other information and parameters. After the email has been transmitted, the connection is terminated again. The email server then starts sending the email to another SMTP server.

SMTP works unencrypted without further measures and transmits all commands, data, or authentication information in plain text. To prevent unauthorized reading of data, the SMTP is used in conjunction with SSL/TLS encryption. Under certain circumstances, a server uses a port other than the standard TCP port `25` for the encrypted connection, for example, TCP port `465`.

An essential function of an SMTP server is preventing spam using authentication mechanisms that allow only authorized users to send e-mails. For this purpose, most modern SMTP servers support the protocol extension ESMTP with SMTP-Auth. After sending his e-mail, the SMTP client, also known as `Mail User Agent` (`MUA`), converts it into a header and a body and uploads both to the SMTP server. This has a so-called `Mail Transfer Agent` (`MTA`), the software basis for sending and receiving e-mails. The MTA checks the e-mail for size and spam and then stores it. To relieve the MTA, it is occasionally preceded by a `Mail Submission Agent` (`MSA`), which checks the validity, i.e., the origin of the e-mail. This `MSA` is also called `Relay` server. These are very important later on, as the so-called `Open Relay Attack` can be carried out on many SMTP servers due to incorrect configuration. We will discuss this attack and how to identify the weak point for it a little later. The MTA then searches the DNS for the IP address of the recipient mail server.

On arrival at the destination SMTP server, the data packets are reassembled to form a complete e-mail. From there, the `Mail delivery agent` (`MDA`) transfers it to the recipient's mailbox.

|Client (`MUA`)|`➞`|Submission Agent (`MSA`)|`➞`|Open Relay (`MTA`)|`➞`|Mail Delivery Agent (`MDA`)|`➞`|Mailbox (`POP3`/`IMAP`)|
|---|---|---|---|---|---|---|---|---|

But SMTP has two disadvantages inherent to the network protocol.

1. The first is that sending an email using SMTP does not return a usable delivery confirmation. Although the specifications of the protocol provide for this type of notification, its formatting is not specified by default, so that usually only an English-language error message, including the header of the undelivered message, is returned.
    
2. Users are not authenticated when a connection is established, and the sender of an email is therefore unreliable. As a result, open SMTP relays are often misused to send spam en masse. The originators use arbitrary fake sender addresses for this purpose to not be traced (mail spoofing). Today, many different security techniques are used to prevent the misuse of SMTP servers. For example, suspicious emails are rejected or moved to quarantine (spam folder). For example, responsible for this are the identification protocol [DomainKeys](http://dkim.org/) (`DKIM`), the [Sender Policy Framework](https://dmarcian.com/what-is-spf/) (`SPF`).
    

For this purpose, an extension for SMTP has been developed called `Extended SMTP` (`ESMTP`). When people talk about SMTP in general, they usually mean ESMTP. ESMTP uses TLS, which is done after the `EHLO` command by sending `STARTTLS`. This initializes the SSL-protected SMTP connection, and from this moment on, the entire connection is encrypted, and therefore more or less secure. Now [AUTH PLAIN](https://www.samlogic.net/articles/smtp-commands-reference-auth.htm) extension for authentication can also be used safely.

```bash
#Version Detection
nc -nv <IP> 25 

# -M means mode, it can be RCPT, VRFY, EXPN
smtp-user-enum -M VRFY -U username.txt -t <IP> 

# Open relay test
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v

sudo nmap -p 25 --script smtp-enum-users 10.129.195.130
msf > use auxiliary/scanner/smtp/smtp_enum

#Sending emain with valid credentials, the below is an example for Phishing mail attack
sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server 192.168.50.242 --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
```

The sending and communication are also done by special commands that cause the SMTP server to do what the user requires.

| **Command**  | **Description**                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------ |
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client.                                     |
| `HELO`       | The client logs in with its computer name and thus starts the session.                           |
| `MAIL FROM`  | The client names the email sender.                                                               |
| `RCPT TO`    | The client names the email recipient.                                                            |
| `DATA`       | The client initiates the transmission of the email.                                              |
| `RSET`       | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY`       | The client checks if a mailbox is available for message transfer.                                |
| `EXPN`       | The client also checks if a mailbox is available for messaging with this command.                |
| `NOOP`       | The client requests a response from the server to prevent disconnection due to time-out.         |
| `QUIT`       | The client terminates the session.                                                               |

To interact with the SMTP server, we can use the `telnet` tool to initialize a TCP connection with the SMTP server. The actual initialization of the session is done with the command mentioned above, `HELO` or `EHLO`.