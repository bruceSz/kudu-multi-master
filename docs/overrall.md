#
#
#

### intro 
This is just the overrall design doc of kudu-multi-master

### goal

This project provide support for kudu multi-master feature.
Specifically, we support master-slave model of mult-master.
Two masters are running at same time, one serve client, and
tablet-tserver, master replicate its data and log to slave.
The well-known "master-slave" mode. Master should switch to slave
quickly and slave should be ready to work quickly after switch to it.

We don't bother to design the master slave switch algorithm, plenty of works
has been proposed to do that.One feasible way would be using zoo keeper on top of different
master and master and slave would be considered equivalent.Another way to do it 
is using the raft protocol, which is more


### master-slave-switch

## zoo-keeper 
Zoo-keeper is a widely used coordination service. But i don't like it.

## self-made-one
Trivial and not interested.

## where are we
If master data is there. Starting a 

