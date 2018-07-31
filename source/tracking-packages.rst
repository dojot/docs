Tracking packages
=================

This tutorial will show how to develop new pluggable modules to dojot to
perform specific tasks. These modules are:

- New nodes in flow palette (stateless procedures)
- New modules in dojot (stateful procedures)

The scenario is: we have a courier delivery services company and we need to
track individual packages. The current package tracking is based only on
manually scanning each device's bar code at departure and arrival in
distribution centers. What we want is a more fine-grained system that allows
checking where a particular package is (or it should be) in the meantime.

Each package is equipped with a RFID and all courier vehicules have RFID
scanners, LTE connection and GPS.


.. NOTE::
   - Who is this for: developers, solution architects
   - Level: advanced
   - Reading time: X m


Implementation details
----------------------

The idea behind this is: associate each package to a vehicule and, whenever an
update from the vehicule is received, this solution also updates all associated
packages. The association could be done by the vehicule itself - each time a
new device is registered by its RFID scanners, it sends a message to dojot
indicating that the device is now associated to it. This message would be
processed by a new software component (to be developed in this tutorial) that
performs this association, as currently there is no module responsible for
"device associations". When a vehicule sends a new location, this update is
also processed by the association module, which in turn will update the
location from all associated packages.


