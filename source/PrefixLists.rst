======================================
Generating a Prefix List for your peer
======================================

A prefix list applied to a BGP peer allows your BGP speaker to only accept a certain set of prefixes on that BGP session. For instance, if you have a prefix list containing "192.0.2.0/24" and the peer tries to advertise "198.51.100.0/24" to you, it will reject that prefix. It will not tear down the BGP session however.

A potential "gotcha" is that if you are only accepting "192.0.2.0/24", you can't accept "192.0.2.0/25" because that is not an exact match. Your router operating system may have syntax so allow this, such as "192.0.2.0/24 ge 25 le 25" which will allow any prefix in "192.0.2.0/24" with CIDR mask "/25", i.e. "192.0.2.0/25" and "192.0.2.128/25". Note that this does NOT include "192.0.2.0/24" itself so be careful with your syntax.

There is an easy solution to this problem, however. A tool called `bgpq3<https://github.com/snar/bgpq3>` exists which automates all this for you.

Firstly, a couple of concepts you need to be aware of:

If you peer with AS65000, they may just be sending you prefixes from AS65000 only.

If you peer with AS64512, they may also be sending you prefixes from AS64512, and all their customers, AS65001 AS65002 AS65003 etc.

To make life easier, `RPSL<http://www.irr.net/docs/rfc4012.txt>` specifies an "as-set" -- a set of ASNs in one object. It looks like this:

.. code-block:: none

  as-set:         AS-EXAMPLE
  descr:          Example AS-SET
  members:        AS64512
  members:        AS65001
  members:        AS65002
  members:        AS65003
  tech-c:         EXAMPLE-RIPE
  admin-c:        EXAMPLE-RIPE
  mnt-by:         EXAMPLE-MNT
  changed:        noc@example.net 20150712
  created:        2015-07-12T14:45:58Z
  last-modified:  2015-07-12T20:20:01Z
  source:         RIPE

We can see here that if we expand "AS-EXAMPLE", we will get AS64512 plus all it's downstream customers. A "members:" entry may contain a further as-set, and can easily be recursed by bgpq3.

We can then find out which "route:" objects are stored in the IRRDB by performing an inverse lookup, such as:

.. code-block:: none

  whois -i origin AS65000

This gives you many route objects that this ASN has registered it is "origin" for. This is something you can parse and create a prefix list with -- do not do this. You will get it wrong and find a corner case that will break your system. Use bgpq3 -- it's really good at this!

Let's have a look at a sample bgpq3 run using Cisco IOS (bgpq3 currently supports IOS, IOS XR, JUNOS, BIRD -- more can be supported if demanded):

.. code-block:: none

  $ bgpq3 AS65000
  no ip prefix-list NN
  ip prefix-list NN permit 192.0.2.0/24
  ip prefix-list NN permit 192.0.2.0/26
  ip prefix-list NN permit 192.0.2.64/26
  ip prefix-list NN permit 192.0.2.128/26
  ip prefix-list NN permit 192.0.2.192/26

We can see here that AS65000 has registered it will be advertising three prefixes, and bgpq3 has spit out some configuration that will get rid of prefix-list "NN" and replace it with the three new prefixes. That's a bit wasteful though -- five entries. I'm sure we can summarise this... bgpq3 will do it for you!

.. code-block:: none

  $ bgpq3 -A AS65000
  no ip prefix-list NN
  ip prefix-list NN permit 192.0.2.0/24
  ip prefix-list NN permit 192.0.2.0/24
  ip prefix-list NN permit 192.0.2.0/24 ge 26 le 26

Excellent. However, if I now run it for AS65001 it will be over-written. Let's put a name in:

.. code-block:: none

  $ bgpq3 -A -l EXAMPLE-in AS65000
  no ip prefix-list EXAMPLE-in
  ip prefix-list EXAMPLE-in permit 192.0.2.0/24
  ip prefix-list EXAMPLE-in permit 192.0.2.0/24 ge 26 le 26

We don't want to accept any prefixes longer than a /24, as those are generally filtered on the internet:

.. code-block:: none

  $ bgpq3 -A -l EXAMPLE-in -m 24 AS65000
  no ip prefix-list EXAMPLE-in
  ip prefix-list EXAMPLE-in permit 192.0.2.0/24

Perfect. To summarise:

====== ========
Option Function
====== ========
-A     Aggregates prefixes as much as possible
-l     Labels the prefix list with a friendly name
-m     Maximum Size of prefix to be accepted
====== ========

With this output, you can install these prefix lists onto your router with your favourite tool (or do it by hand, but preferably do this at least once a week at a minimum -- most providers do this once per day)
