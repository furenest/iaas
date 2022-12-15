=========================
OpenStack Designate (DNS)
=========================

.. WARNING::
   This document is under construction. Designate is not yet in
   production.

Managing top-level domains (TLDs)
=================================

.. _Internet Assigned Numbers Authority (IANA): https://www.iana.org/

We manage a list of legal top-level domains, in which the users can
create their zones. With such a list in place, zones may only be
created with the TLDs listed there. Without such a list, any TLD would
be legal, and users could also register zones such as "com", thus
preventing any other user to create a domain under the ".com" TLD.

A list of all valid top-level domains is maintained by the `Internet
Assigned Numbers Authority (IANA)`_ and is updated from time to
time. The list is available here:

* http://data.iana.org/TLD/tlds-alpha-by-domain.txt

This list can be imported into Designate via the himlarcli
command **dns.py**::

  # download the TLD list from IANA
  curl http://data.iana.org/TLD/tlds-alpha-by-domain.txt -o /tmp/tlds-alpha-by-domain.txt

  # import the TLD list into designate
  ./dns.py tld_import --file /tmp/tlds-alpha-by-domain.txt

The **tld_import** action will add any TLDs which aren't already
registered, with a special comment that marks the TLD as being bulk
imported. Any registered TLDs that aren't on the import list, and also
have the special bulk import comment, will be deleted. This means that
we can add our own TLDs if needed, as they will not be deleted by the
bulk import. The bulk import as show above is designed to run
automatically by cron etc.

The himlarcli command also have actions to create, delete and update a
TLD, and to view the list or a specific TLD.


Managing blacklists
===================

Blacklisting is done to prevent users from creating domains that we
don't want them to create. This is mostly done to protect a domain
from eventual future use. In order to prevent any users from creating
the domain **foo.com** and any subdomains::

  ./dns.py blacklist_create \
      --comment 'Protect domain foo.com including any subdomains' \
      --pattern '^([A-Za-z0-9_\-]+\.)*foo\.com\.$'

If we want to only protect the domain itself, but allow users to
create subdomains in the domain, we can use a simpler pattern::

  ./dns.py blacklist_create \
      --comment 'Protect domain foo.com while allowing subdomains' \
      --pattern '^foo\.com\.$'

Listing the blacklists is done via the **blacklist_list** action (the
option **--pretty** formats the output in a table). Example::

  $ ./dns.py blacklist_list --pretty
  +----------------------------------+--------------------------------------------------+--------------------------------------+
  | pattern                          | description                                      |                  id                  |
  +----------------------------------+--------------------------------------------------+--------------------------------------+
  | ^([A-Za-z0-9_\-]+\.)*foo\.com\.$ | Protect domain foo.com including any subdomains  | 8958bf52-8e64-4a86-87ea-2087b7bc6d60 |
  | ^bar\.net\.$                     | Protect domain bar.net while allowing subdomains | b3f7fc9f-67a8-4d07-aabc-444f0e4d67c4 |
  +----------------------------------+--------------------------------------------------+--------------------------------------+

Updating a blacklist entry is done via the **blacklist_update**
action. Updating the comment (example)::

  $ ./dns.py blacklist_update --comment 'new comment' \
       --id b3f7fc9f-67a8-4d07-aabc-444f0e4d67c4

And updating the pattern (example)::

  $ ./dns.py blacklist_update --new-pattern '^tralala\.org\.$' \
       --id b3f7fc9f-67a8-4d07-aabc-444f0e4d67c4

Deleting a blacklist entry is done via the **blacklist_delete**
action::

  ./dns.py blacklist_delete --id <ID>

Blacklisted domains
-------------------

The following domains should be blacklisted in production:

+------------------+------------------------------------------+-------------------------+
| DOMAIN           | PATTERN                                  | COMMENT                 |
+==================+==========================================+=========================+
| uh-iaas.no       | ``^([A-Za-z0-9_\-]+\.)*uh-iaas\.no\.$``  |Official UH-IaaS domain, |
|                  |                                          |managed outside of       |
|                  |                                          |Openstack Designate      |
+------------------+------------------------------------------+-------------------------+
| uhdc.no          | ``^([A-Za-z0-9_\-]+\.)*uhdc\.no\.$``     |Official UH-IaaS domain, |
|                  |                                          |managed outside of       |
|                  |                                          |Openstack Designate      |
+------------------+------------------------------------------+-------------------------+
| uio.no           | ``^([A-Za-z0-9_\-]+\.)*uio\.no\.$``      |Domain belonging to      |
|                  |                                          |UiO. Instances in UH-IaaS|
|                  |                                          |are not allowed to have  |
|                  |                                          |UiO addresses            |
+------------------+------------------------------------------+-------------------------+
| uib.no           | ``^uib\.no\.$``                          |Domain belonging to      |
|                  |                                          |UiB. The domain itself is|
|                  |                                          |forbidden, but subdomains|
|                  |                                          |are allowed              |
+------------------+------------------------------------------+-------------------------+
| uiocloud.no      | ``^uiocloud\.no\.$``                     |Domain belonging to      |
|                  |                                          |UiO. The domain itself is|
|                  |                                          |forbidden, but subdomains|
|                  |                                          |are allowed              |
+------------------+------------------------------------------+-------------------------+
| uninett.no       | ``^([A-Za-z0-9_\-]+\.)*uninett\.no\.$``  |Domain belonging to      |
|                  |                                          |Uninett. Reserved for    |
|                  |                                          |possible future use      |
|                  |                                          |                         |
+------------------+------------------------------------------+-------------------------+
| ntnu.no          | ``^([A-Za-z0-9_\-]+\.)*ntnu\.no\.$``     |Domain belonging to      |
|                  |                                          |NTNU. Reserved for       |
|                  |                                          |possible future use      |
|                  |                                          |                         |
+------------------+------------------------------------------+-------------------------+
| uia.no           | ``^([A-Za-z0-9_\-]+\.)*uia\.no\.$``      |Domain belonging to      |
|                  |                                          |UiA. Reserved for        |
|                  |                                          |possible future use      |
|                  |                                          |                         |
+------------------+------------------------------------------+-------------------------+

These are added with these commands::

  ./dns.py blacklist_create --pattern '^([A-Za-z0-9_\-]+\.)*uh-iaas\.no\.$' --comment 'Official UH-IaaS domain, managed outside of Openstack Designate'
  ./dns.py blacklist_create --pattern '^([A-Za-z0-9_\-]+\.)*uhdc\.no\.$' --comment 'Official UH-IaaS domain, managed outside of Openstack Designate'
  ./dns.py blacklist_create --pattern '^([A-Za-z0-9_\-]+\.)*uio\.no\.$' --comment 'Domain belonging to UiO. Instances in UH-IaaS are not allowed to have UiO addresses'
  ./dns.py blacklist_create --pattern '^uib\.no\.$' --comment 'Domain belonging to UiB. The domain itself is forbidden, but subdomains are allowed'
  ./dns.py blacklist_create --pattern '^uiocloud\.no\.$' --comment 'Domain belonging to UiO. The domain itself is forbidden, but subdomains are allowed'
  ./dns.py blacklist_create --pattern '^([A-Za-z0-9_\-]+\.)*uninett\.no\.$' --comment 'Domain belonging to Uninett. Reserved for possible future use'
  ./dns.py blacklist_create --pattern '^([A-Za-z0-9_\-]+\.)*ntnu\.no\.$' --comment 'Domain belonging to NTNU. Reserved for possible future use'
  ./dns.py blacklist_create --pattern '^([A-Za-z0-9_\-]+\.)*uia\.no\.$' --comment 'Domain belonging to UiA. Reserved for possible future use'



Reinstall exisiting name server
-------------------------------

*Designate* constantly check existing zone data on the name servers (`slaves`).
If any of the existing servers are reinstalled, they lose the data concerning
the Designate administered domains! Designate itself will, when the
discrepancies are uncovered, attempt to **modify** the zones on the server. Due
to the zone not being configured in any of the views of the new system, it will
reject any of these attempts. Unfortunately it seems that Designate will only
create a zone on the slaves during its initial definition process (i.e. a user
running the ``openstack zone create`` command). From then on the zone is
expected to exist. For this reason, all previously created zones must be
recreated on the new server.

This may be done by quering Designate for any active zones, and then execute a
suitable *rndc* command. Fortunately Designate creates a zone in this manner
using a deterministic pattern, so it is trivial to define a suitable command for
this issue.

When the new server has finished its installation, follow this procedure:

1) Block incoming DNS traffic (forcing all requests over to ns2)

   *NB*: Do not block traffic from the admin node!
2) Ensure `puppet` has run several times

   Check the existence of zone files not administered by Designate in */var/named/pz*
3) On the *dns* node, execute this (altering the data inside square brackets):

   .. code:: bash

      . openrc
      OLDIFS="$IFS"
      export IFS=$'\t\n'
      for LINE in $(openstack zone list -f value -c name -c id --all); do
        IFS=$' \t\n' read -r ID ZONE <<<$(echo $LINE); rndc -s [rndc-servers IP-adresse] -p 953 -c /etc/rndc.conf -k /etc/rndc-mdns.key "addzone ${ZONE}  { type slave; masters {[designate-servers IP-adresse] port 5354;}; file \"slave.${ZONE}${ID}\"; };";
      done
      IFS="$OLDIFS"

   Afterwards check the existence of slave files in */var/named*

4) Perform various sanity checks, for instance:

   - There should be one slave zone file for every existing **ACTIVE** zone

     Note that if comparing with other slave name servers, they may have several
     additional files. These are from previously created but now deleted zones.
     The files are not always instantly deleted.
   - Check zone status with the command `rndc zonestatus [zone]` (locally!)
   - Query Designate for currently active zones and compare with existing slave
     zone files
   - Dump the local nameserver database (`rndc dumpdb -all`) and validate
     zone data from the dump file (*/var/named/data/named_dump.db*)

5) Remove traffic block
6) Monitor logs:

   - `journalctl -fu named`
   - check/follow */var/named/data/querylog*


