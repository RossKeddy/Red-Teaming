By default, ports `110` and `995` are used for POP3, and ports `143` and `993` are used for IMAP. The higher ports (`993` and `995`) use TLS/SSL to encrypt the communication between the client and server. Using Nmap, we can scan the server for these ports. The scan will return the corresponding information (as seen below) if the server uses an embedded certificate.

With the help of the `Internet Message Access Protocol` (`IMAP`), access to emails from a mail server is possible. Unlike the `Post Office Protocol` (`POP3`), IMAP allows online management of emails directly on the server and supports folder structures. Thus, it is a network protocol for the online management of emails on a remote server. The protocol is client-server-based and allows synchronization of a local email client with the mailbox on the server, providing a kind of network file system for emails, allowing problem-free synchronization across several independent clients. POP3, on the other hand, does not have the same functionality as IMAP, and it only provides listing, retrieving, and deleting emails as functions at the email server. Therefore, protocols such as IMAP must be used for additional functionalities such as hierarchical mailboxes directly at the mail server, access to multiple mailboxes during a session, and preselection of emails.

Clients access these structures online and can create local copies. Even across several clients, this results in a uniform database. Emails remain on the server until they are deleted. IMAP is text-based and has extended functions, such as browsing emails directly on the server. It is also possible for several users to access the email server simultaneously. Without an active connection to the server, managing emails is impossible. However, some clients offer an offline mode with a local copy of the mailbox. The client synchronizes all offline local changes when a connection is reestablished.

The client establishes the connection to the server via port `143`. For communication, it uses text-based commands in `ASCII` format. Several commands can be sent in succession without waiting for confirmation from the server. Later confirmations from the server can be assigned to the individual commands using the identifiers sent along with the commands. Immediately after the connection is established, the user is authenticated by user name and password to the server. Access to the desired mailbox is only possible after successful authentication.

SMTP is usually used to send emails. By copying sent emails into an IMAP folder, all clients have access to all sent mails, regardless of the computer from which they were sent. Another advantage of the Internet Message Access Protocol is creating personal folders and folder structures in the mailbox. This feature makes the mailbox clearer and easier to manage. However, the storage space requirement on the email server increases.

Without further measures, IMAP works unencrypted and transmits commands, emails, or usernames and passwords in plain text. Many email servers require establishing an encrypted IMAP session to ensure greater security in email traffic and prevent unauthorized access to mailboxes. SSL/TLS is usually used for this purpose. Depending on the method and implementation used, the encrypted connection uses the standard port `143` or an alternative port such as `993`.

#### IMAP Commands

|**Command**|**Description**|
|---|---|
|`1 LOGIN username password`|User's login.|
|`1 LIST "" *`|Lists all directories.|
|`1 CREATE "INBOX"`|Creates a mailbox with a specified name.|
|`1 DELETE "INBOX"`|Deletes a mailbox.|
|`1 RENAME "ToRead" "Important"`|Renames a mailbox.|
|`1 LSUB "" *`|Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`.|
|`1 SELECT INBOX`|Selects a mailbox so that messages in the mailbox can be accessed.|
|`1 UNSELECT INBOX`|Exits the selected mailbox.|
|`1 FETCH <ID> all`|Retrieves data associated with a message in the mailbox.|
|`1 CLOSE`|Removes all messages with the `Deleted` flag set.|
|`1 LOGOUT`|Closes the connection with the IMAP server.|

#### POP3 Commands

|**Command**|**Description**|
|---|---|
|`USER username`|Identifies the user.|
|`PASS password`|Authentication of the user using its password.|
|`STAT`|Requests the number of saved emails from the server.|
|`LIST`|Requests from the server the number and size of all emails.|
|`RETR id`|Requests the server to deliver the requested email by ID.|
|`DELE id`|Requests the server to delete the requested email by ID.|
|`CAPA`|Requests the server to display the server capabilities.|
|`RSET`|Requests the server to reset the transmitted information.|
|`QUIT`|Closes the connection with the POP3 server.|