MBS/IP Protocol Documentation
=============================

MBS/IP is a file transfer protocol used (mostly) by Austrian banks to transfer account statements and transactions from users to the banks and vice versa.

There are multiple versions of MBS/IP in use. This describes one of them, probably the 6.0 version, but this hasn't been verified yet.

General structure
-----------------

MBS/IP runs on port 3048, and is encrypted using SSLv3/TLSv1. Servers have been observed to support 3DES-SHA1.

The base protocol is text-based and resembles FTP to some degree. The client sends commands, the server responds with a confirmation string.

OK Reply
--------

The server confirms completion of a command by sending a line starting with `+OK` and ending with `\r\n`. There may be additional data between the `+OK` and the LF/CR. It has been observed that the server may send multiple "lines" before the final LF/CR if those lines are separated only by a LF character.

Startup
-------

After the SSL handshake, the server immediately sends an OK Reply identifying itself.
It has been observed that not all servers send the same amount of identification.

After the identification message, the client can start sending commands.

Base Commands
-------------

The client can send commands to the server. Each command line is terminated with `\r\n`.

Commands can take arguments. The command word and the arguments are separated by a single space character. It's not clear if the server treats everything following the command word as a single argument, or if indeed multiple arguments are supported. It has been observed that it may be beneficial to always send a space character after the command name, even if no arguments are provided.


### ELCONNECT

Purpose unclear.
No arguments.

Always sent by the client after receiving the server identification message.
Server responds with empty OK reply.

### ELCHECKSAMEPORT

It is assumed that this enables "same port" data streams.

Sent by the client after `ELCONNECT`.
Server responds with empty OK reply.

### ELCHECKCOMPRESS

Check support for gzip compression for data replies and retrieve minimum size after which compression will be used.

Sent by the client after `ELCHECKSAMEPORT`.
Server responds with an OK reply, argument indicates minimum size (bytes) for replies when compression will be applied. `0` is a valid minimum size (indicating that compression will always be used).

### ELPCPAKET

Client software identification. The argument is the name of the client software package.

A valid client software package appears to be `MBSIP Client GUI Edition (Version: 1.6.3-Release-2 (20131217))`. (This string is sent as a single argument with no quotes.)

Sent by the client after `ELCHECKCOMPRESS`.
Server responds with empty OK reply.

### ELPCVERSION

Client software version identification. The argument is the major version of the client software package.

A valid client software major version appears to be `1.6`.

Sent by the client after `ELPCPAKET`.
Server responds with empty OK reply.

### ELRZKENNUNG

Hostname the client was instructed to connect to (similar to the `Host:` header in HTTP/1.x). The argument is the name of the target server.

It is assumed that this is used for routing purposes when a single server provides service for multiple banks.

Sent by the client after `ELPCVERSION`.
Server responds with empty OK reply.

### ELKOMMBER

Username for authentication. Argument is the username.

Sent by the client after `ELKOMMBER`.
Server responds with empty OK reply.

### ELKOMMPW

Password for authentication. Argument is the plaintext password.

Sent by the client after `ELRZKENNUNG`.
Server responds with empty OK reply.

### MBS\_GETUPT\_FILENAMES

Purpose unclear. Possibly initiates a file transfer for the certificate update list.

Sent by the client after `ELKOMMPW`.
Server responds with empty OK reply.

### ELDISCONNECT

Terminate current session. No arguments.

Sent by the client to disconnect the session.
Server responds with empty OK reply.

File transfer commands
----------------------

As MBS/IP is a protocol for transfering files, it has commands to do so.

MBS/IP has two concepts of files, one of them is more like a request/reply stream (usually XML gets sent around, so we'll call this concept XML request/reply from now on), and the other one is like a real file.

Real files are not represented directly in the base protocol, instead all listing and manipulation of files is done using the XML request/reply commands.

### XML request/reply transfer flow

The client "puts" an XML request, the server processes it, and the client immediately fetches the response. The involved commands look a bit like multi-plexed FTP control and data commands.


Client > `ELNEWFILESAMEPORT`, argument is a number representing the length of the uncompressed data to be sent.

Server < `150 Start to open data connection\r\n`.

Client > compressed data (XML command request)

Server < empty OK reply

Client > `ELCHECK_SENDFILE`, argument again is uncompressed data length (from `ELNEWFILESAMEPORT` command)

Server < OK reply with a number (stream id?)

Client > `ELGETFILESAMEPORT`, argument is id from server.

Server < `150 NNN start to open data connection\r\n`, `NNN` is the uncompressed data length that will be sent

Server < compressed data (XML command reply)

Server < empty OK reply

Client > `ELCHECK_RECVFILE`, argument is the uncompressed data length that has been received

Server < OK reply with message  `Datei erfolgreich gesendet und als abgeholt markiert`

XML commands
------------

XML commands are XML documents with no XML header.

For requests (created by client), the root element is named `mbsip-request`, for replies (from server) the root element is named `mbsip-reply`.

### `info`

Retrieve some information from the server.

Request element: `info`. There are no attributes, value is empty.

Reply element: `info-reply`.

Known reply attributes:

* `deleteUser`: 0
* `version`: `1.6.3-Release-2 (20131217)`
* `groupID`: 0
* `createUser`: 5
* `homeDir`: `/`
* `resetAnyPwd`: 10

