jelectrum
=========

An Electrum Server written in Java

This is designed as replacement for the python electrum server:
https://github.com/spesmilo/electrum-server

Reasons to exist
----------------

1) The inital blockchain sync is multithreaded and a good bit faster than the python implementation
(2 - 3 days with a fast system and SSD)

2) It is good to have multiple implementations of things in general


DB Options
----------

Not sure what to use?  Use leveldb.  It is fast and has the smallest on disk footprint.

Make sure you are using a revent leveldb version.  1.17 and 1.18 seem to work well.


How to LevelDB
--------------

LevelDB seems cool and the C++ library seems good, so I've made a network layer to call into the C++ leveldb.

This is done because I've found things to be more reliable if the database is a separate process that doesn't
get restarted.  It is thus less likely to corrupt the datastore and be a problem.

Anyways, to run it, go into (assuming you are currently in jelectrum directory):
```
    $ screen -mdS LNET
    $ screen -r LNET
    $ cd cpp
    $ make
    $ ./levelnet /location/of/leveldb/here
```
(replace that path with where you want the leveldb to live)

This will run th leveldb network server on port 8844.  There is absolutely no security or checking.
Firewall it.

Then you can run jelectrum with db_type=leveldb and copy the other settings from the sample file

Note: there are a few cases which cause levelnet to exit because that is the only sane thing to do.
In those cases, just run it again.  Jelectrum will be retying operation till they work and things should
resume.


DB MongoDB
----------

MongoDB: Install and setup mongodb.  A single node instance is just fine.  Run mongodb on SSD if possible.
On startup, jelectrum will create tables as needed.


PostgreSQL
----------

PostgreSQL: Install and setup postgresql.
On startup, jelectrum will create tables and indexes needed.  You'll need to create the database and a user and put it in the config.


Lobstack
--------
Lobstack: Copy the lobstack config entries and modify them.  Lobstack uses a stupid amount of space and needs to occasionally have
a compress run.  See compressdb.sh.  For initial sync, the pattern is something like:

while true
do
  java -Xmx4g -Xss256k -jar jar/Jelectrum.jar jelly.conf
  ./compressdb.sh
done

It will exit when space gets low (configurable with 'lobstack_minfree_gb').  Then you run a compress, then run the service again.


How to run
----------

Make your SSL cert:
./makekey.sh

Build:
ant jar

Config:
cp jelly.sample.conf jelly.conf
edit jelly.conf as makes sense

Run:
java -Xmx4g -Xss256k -jar jar/Jelectrum.jar jelly.conf

My Instance
-----------

Feel free to connect to my instance which should be running the latest version.

```
b.1209k.com 50001 (tcp)
b.1209k.com 50002 (ssl)
h.1209k.com 50001 (tcp)
h.1209k.com 50002 (ssl)
```

What Doesn't Work
-----------------

1) HTTP/HTTPS.  I see no strong reason to support those over TCP and SSL+TCP.  I imagine it wouldn't be too hard
but I don't see the need.  If someone feels otherwise, let me know.

2) The following commands that clients don't seem to issue (yet):
```
blockchain.address.listunspent
blockchain.address.get_balance
blockchain.address.get_proof
blockchain.address.get_mempool
blockchain.utxo.get_address
```

(To check this list compare src/blockchain_processor.py from electrum-server to StratumConnection.java)

Monitoring
----------

If you are going to run it, monitor it!

I've made a monitoring tool that anyone to can use to monitor their electrum server (jelectrum or otherwise):

https://1209k.com/bitcoin-eye/ele.php





