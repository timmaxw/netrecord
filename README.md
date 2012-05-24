netrecord
=========

`netrecord` listens for TCP connections to a given port and forwards the traffic
to another address, but also records the traffic.

Installation
------------

    make netrecord
    make install

Usage
-----

    ./netrecord <server host> <server port> <proxy port> <log file root>

`netrecord` will listen on `<proxy port>`; it will forward traffic to
port `<server port>` on `<server host>`. For each connection, it will make a
file named `<log_file_root>` with a number after it; in that file, it will
record traffic in both ways on the connection.

Log file format
---------------

The log file consists of the following records:

* `c <timestamp> con\n` is written at the beginning of the log file.
    `<timestamp>` is the number of seconds since `netrecord` was started.

* `c <timestamp> <num>\n<bytes>` is written whenever the client sends something
    to the server. `<timestamp>` is as above. `<num>` is the number of bytes
    that the client sent. `<bytes>` is the data that the client sent. There is
    no newline after `<bytes>`. 

* `s <timestamp> <num>\n<bytes>` is the same as above, except for data that the
    server sent to the client.

* `c <timestamp> hup\n` is written when the client hangs up.

* `s <timestamp> hup\n` is written when the server hangs up.

* `c <timestamp> err\n` is written when `netrecord` gets an error from its TCP
    connection to the client.

* `s <timestamp> err\n` is written when `netrecord` gets an error from its TCP
    connection to the server.

Usually the log file will end with an `err` or `hup` record, but if the server
never accepts `netrecord`'s connection, there will be no end record.

Limitations
-----------

`netrecord` is implemented in terms of socket primitives (`listen()`,
`connect()`, `recv()`, `send()`, etc.) rather than by intercepting TCP traffic
in the kernel. That means it doesn't forward TCP traffic perfectly. For example:

* `netrecord` will affect the timing of TCP packets. This can lead to heisenbugs
    if you are trying to use `netrecord` to debug a program.

* `netrecord` accepts any connections it gets, and then tries to establish
    connections to the server; if the server rejects `netrecord`'s connection,
    then the client will see its connection accepted and then immediately
    closed.

History
-------

Tim Maxwell wrote `netrecord` in 2010 as an internal tool at RethinkDB.