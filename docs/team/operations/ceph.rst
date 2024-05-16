
=========================
Practical ceph operations
=========================

This document aims to describe ceph-related operations in NREC, without detracting from the magic of ceph.


It is always interesting to keep an eye on the state of ceph, use ``ceph -w`` or ``watch ceph status``.


Replace a borked drive
----------------------

At this stage we assume that the OSD is in the "down" state, puppet is disabled on the storage host, and the cluster is stable without any degraded PGs - all data is rebuilt on other OSDs.

We now need to find the /dev and the physical device matching the osd. We will use osd.443 as an example.

Find volume group and take notes:



Find /dev

Find serial number and location

.. code:: bash

  hdparm -I /dev/sdg | grep Serial

On iDrac, use the suearch functionality found in the storage overview to find the pysical location of the drive:

.. image:: images/idrac_search_serial.png
  :width: 400
  :alt: Alternative text


purge osd

.. code:: bash

  # not really necessary, but can prevent small accidents from developing into suboptimal and murky chain reactions
  ceph osd set noout
  # if the drive is marked as down and in the cluster, do not rebalance the cluster untill a new drive is ready
  ceph osd set norebalance
  # Did you remeber to disable puppet?
  ceph purge osd.443 --yes-i-really-mean-it

Check if ceph-osd-service still exists

.. code:: bash

  systemctrl unit list | 443
  # if it exists, remove it
  systemctrl disable ceph-osd@443

Check if the volume group still exists, and if so, remove it:

.. code:: bash

  vgscan | grep <volume groupe found earlier>
  vgremove <name of volume group>

Now it is time to physically replace the drive. Check youtube for your server.

If the new disk has a new /dev-name, typically the alphabetically latest of all your drives, then the server needs a reboot. *noout* must be set before reboot.

After a reboot, when the cluster is stable, or if no reboot was nessesserry:

.. code:: bash

  puppet agent --enable; puppet agent -t
  ceph osd unset norebalce
  ceph osd unset noout


backfill backfill_tofull
------------------------

To avoid this hopelessness in the long run – add more storage when possible and plan for balanced pools.

Hovewer, in the short term:

Find the fullest OSDs, and if the storage capacity on these is less than the average drive in the pool, try doing some rebalancing:

.. code:: bash

  ceph osd df
  ceph reweigt <value less than current> <full osd>


Some helpfull oneliners:

.. code:: bash
  
  # only look at ssd-drives
  ceph osd df| grep ssd | awk
