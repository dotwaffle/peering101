# So, You've Decided To Peer -- What Next?

This project seeks to make an easy to follow, concise publication on how to be a "good BGP citizen". This should include examples for as many "big vendor" routing platforms as possible, and be applicable to all BGP peering sessions (with the possible exception of transit).

It aims to cover:

* How to use max-prefix safely.
* How to generate / use prefix lists.
* How to generate / use as-path filters.
* How to implement BCP38 on your edge ports.
* How to implement RPKI, where applicable.

It will not cover:

* Edge cases.
* What BGP is, or what iBGP is.
* What peering is, or how to peer effectively.
* The history of peering.
* Good / Bad networks to peer with.
* BGP Path selection.
* BGP Communities, except with an IXP Route Server.

It should remain, where possible, without overly vendor specific features or mentions of particular products in isolation.

The expected deliverables are threefold:

1. An HTML guide, on readthedocs.org or similar.
2. A PDF (preferably in A4, A5, and US Letter sizes) for download.
3. A slide-deck for inclusion into Network Operator Group conferences.

It is likely that #3 will be created at a later date.

All submitted contributions to the main upstream source repository must be in source (preferably textual) form. If you choose to submit a binary (such as an image) it must be absent of all branding (commercial or otherwise) and be provided free of charge and subject only to the terms of the license of this documentation. You must have rights to the submitted material, and must be able to release into the Public Domain. This does not infringe on your right to fork this repository and produce your own derivative work.
