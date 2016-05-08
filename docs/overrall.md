#
#
#

### intro 
This is just the overrall design doc of kudu-multi-master

### goal

This project provide support for kudu multi-master feature.
Specifically, we support master-slave model of mult-master.
Two masters are running at same time, one serve client, and
serve tablet-tserver, master replicate its data and log to slave
(The well-known "master-slave" mode). Master should switch to slave
quickly and slave should be ready to work quickly after switch to it.

We don't bother to design the master slave switch algorithm, plenty of works
has been proposed to do that.One feasible way would be using zoo keeper on top of different
masters and master and slave would be considered equivalent.Another way to do it 
is using the raft protocol, which has been used in kudu rpc code. The simplest 
way to do master-slave mode multi-master can also refer that of mysql: one master just 
replicate its log to slave, where slave is cold standby , while the switch is done through 
external program


### master-slave-switch

## zoo-keeper 
Zoo-keeper is a widely used coordination service. But i don't like it.

## raft
Kudu community is working on this.

## self-made-one
Trivial and not interested.

## where are we
If master data is there. Starting a kudu-master using existing data and recover
a kudu-master is easy. According This, using cold standby or read only standby + 
master-slave log copy seems to be simplest way. Also manual switch is also accept
in some case.


### master-slave-copy

## prepare
Like mysql we plan to add flag to kudu-master when starting.Master should have slave
ip addresses to where replicate log .Slave should have master ip to register itself.
As kudu has both persistent data and wal , slave should have some base data from master .
otherwise it should take a while for slave to be ready to accpet read/write as a master
after switch to it(log replay).
To send correct number of wal log to slave during slave start up. We must make a wal log 
comparation: first anchor master's wal log current-head equals to replicate-head.
and mark the diff-head where wal logs on master and slave begin to diff.And transfer data
between replicate-head and diff-head(also mark current-replicate-head as replication is 
under going).


## copy
We only copy wal log to slave.To make whole cluster recover asap, we should also make
it very quick to replicate wal from master to slave, and to replay the wal log on slave
(cache on master will be lost, we don't consider it currently).The total time of cluster 
become usable is the sum of log_replicate_time + wal_reply_time(not include the switch).
In one-write-multi-read mode. this is also True.
We chose a new master from different slaves, to guarantee all slave have same data, master should
return success after replicate log to all slaves.

## after witch and switch-back
If switch happened,slave should have the data recovered at the time of last wal-log-replicate.
There is possible data gap between old master and new master.If switch-back happens, we may consider
replay that part of data. To do that we should always anchor a tag in wal log, the current head,
last replicate-head during replicate, after switch ops between current head and last replicate-head.
should be replayed.

