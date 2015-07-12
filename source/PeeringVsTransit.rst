===================
Peering vs. Transit
===================

When you took your transit connections, you probably just accepted every prefix they sent you. You might have applied some policies to it such as reducing the local preference or accepting/ignoring MEDs, but for the most part you're just receiving BGP UPDATEs from them and installing them into your routers FIB. You trust your transit provider to only send you prefixes that not only they can reach, but also that they have validated as belonging to their customers -- or routes through to other transit providers that have done similar due diligence.

However, you're now looking to peer with another network. If you set it up identically to a transit provider, the traffic will flow out of that peering connection assuming the AS Path is shorter or some other locally set metric. Unfortunately, that peer may advertise (accidentally or nefariously) prefixes to you that they do not own, either as another AS along the path or even worse as the origin for those prefixes! Your router will blindly accept these prefixes and route your traffic to their AS which may end up with the traffic being dropped due to congestion or attack.

Let's compare this with any downstream customers you may have: You probably had a prefix list (manually edited) to only accept certain prefixes from them. If they do leak a full transit feed to you, you're just going to ignore them. However, although you're ignoring the prefix announcements they may all be stored in memory and therefore using precious router resources.

What's needed is a way to verify that not only are your peers sending you prefixes they have been delegated by the RIRs, but that any downstream customers they have are being advertised on with a valid AS Path.

This document seeks to show you how to generate (and hopefully automate) prefix lists, AS Path filters, and a few other more recent tools that will allow you to verify that these resources have been advertised to you with the consent of the RIRs involved. This will allow you not only to become a "good netizen" but also prevent you from being affected by many of the routing errors the community has seen in recent times.

In short: These processes will greatly help you avoid incidents of your peers causing outages on your network.
