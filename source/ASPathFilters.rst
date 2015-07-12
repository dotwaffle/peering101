=============================
Validating AS-PATH attributes
=============================

Verifying the prefix origin is registered through this peer is a good start, but there are other attributes we can validate also -- the major one being the AS Path.

With bgpq3, this is as simple as using the "-f" option which generates an as-path filter for you for a given AS-SET:

.. code::

  $ bgpq3 -f 65000 AS65000
  no ip as-path access-list NN
  ip as-path access-list NN permit ^65000(_65000)*$

You will now only accept prefixes with the AS Path attribute starting "65000" and ending "65000", with any number of "65000" in between (this is in case the network is prepending to you, there may be multiples of their ASN).

However, if we try and do this for an as-set, we'll see more data:

.. code::

  $ bgpq3 -f 64512 AS-EXAMPLE
  no ip as-path access-list NN
  ip as-path access-list NN permit ^64512(_64512)*$
  ip as-path access-list NN permit ^64512(_[0-9]+)*_(65001|65002|65003)$

Here we can see that we'll accept anything with the next ASN in the path being any number of "64512" ASNs, followed by a path terminating in AS65001, AS65002, or AS65003. This was expanded form the AS-SET "AS-EXAMPLE" which has these ASNs as members.

This access-list can then be referred to as usual within your route-map:

.. code::

  router bgp 65000
    neighbor 10.1.1.1 remote-as 64512
    neighbor 10.1.1.1 route-map EXAMPLE-in in

  route-map EXAMPLE-in permit 10
    match as-path NN

Using Regular Expressions such as those in the as-path access-lists above may cause strain on your router. While they may be of benefit, many networks opt not to implement them because it greatly increases convergence times on older platforms. Remember that these filters are only evaluated on the prefix being selected for consideration -- you should have a backup route to this prefix, usually transit. Therefore, while your router may take a little time to flip traffic onto the peering path it is probably worth the wait to validate the IRRDB approves of the path.

Using as-path access-lists is NOT RECOMMENDED if you do not have a 32-bit ASN capable router, or the peer is likely to have "AS23456" (the 4 byte transition ASN) in the path. This was more of a problem 5-10 years ago, but certain platforms still do not have good 32-bit ASN support. It is an exercise left to the reader to determine if their router operating system has sufficiently new code to perform this operation. Anything from 2015 or later is likely to be a direct "OK to use" answer from your support partner.
