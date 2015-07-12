============================
Setting a "max-prefix" limit
============================

Setting a limit on the number of prefixes you'll accept from a peer is one of the simplest things you can do. What this does is say "if you send me more than X prefixes, I will forcibly bring down my peering with you".

You will see recommendations of various algorithms to set these prefix limits -- normally 10% of the number of prefixes you expect from them. The problem is that when you set these limits it is easy to forget they are in place and when they take a new customer on you may see a sudden jump in the number of prefixes, causing your session to reset.

The purpose of this limit is a "final" failsafe. If your prefix lists fail, you want this to bring down the session which will hopefully alert both your NOC and the peer that something "incorrect" has happened. Therefore, it wants to be high to prevent accidentally tripping it but also low enough to not blindly accept them accidentally sending you all their peering routes etc.

The suggestion here is to multiply the number of routes they will be sending you and multiply it by 10:

========  =======
Received  Limit
========  =======
1         10
35        350
950       9500
75000     500000
========  =======

One peer is sending us 75000 prefixes, yet the limit table says 500000 -- at the time of writing the DFZ is reaching 600000 prefixes. If they send more than 500000 prefixes we are almost certainly receiving a full table from them so we need to drop it before we reach that number. Very few peers will ever send you this many prefixes, and indeed your routing platform may not accept this many prefixes in a prefix list so you may have to rely on just this rudimentary way of detecting a routing mistake.

The max-prefix limit is usually set per-neighbor. A Cisco IOS router will be configured in the following way:

.. code::
  neighbor 10.1.1.1 maximum-prefix 350

This will drop the peer 10.1.1.1 when more than 350 prefixes are received.

There are some other options you can consider:

.. code::
  neighbor 10.1.1.1 maximum-prefix 350 warning-only

This will only warn when the peer sends more than 350 prefixes, the session will remain Established.

.. code::
  neighbor 10.1.1.1 maximum-prefix 350 100

This will warn at 100 prefixes (remember, you're expecting 35) and drop at 350.

.. code::
  neighbor 10.1.1.1 maximum-prefix 350 100 warning-only

This will warn at 100 prefixes as before, but send a different warning at 350. The session will remain Established.

Similar functionality is present in other network operating systems. Full examples for your device may be posted at the end of this document -- if it is missing and you're unsure how to proceed, please do feel free to ask and a later revision of the document may include your router syntax!

