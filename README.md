# PPLive Network

PPLive is the largest chunk-driven multimedia streaming
p2p overlay in the world. As of May 2006, PPLive had
over 200 distinct online channels, a daily average of 400,000
aggregated users, and most of its channels had several thousands of users at their peaks [2]. The system is increasing
in popularity, especially in China and Asia. For instance,
during the Chinese New Year 2006 event, a particular PPLive
channel had over 200,000 simultaneous viewers [15]. In our
experiments during July 2006, we observed that there are about
400 online channels on the English version page of PPLive.

PPLive streams live TV and video data through overlays of
cooperative peers. The PPLive system has multiple channels,
each of which forms its own overlay. Each channel streams
either live audio-video feeds, or movies according to a preset
schedule. A human user may join any channel via her client
machine, but the client machine could also be used to relay
feeds for channels other than the subscribed one (by the
PPLive protocol).

Understanding how a large deployed system like PPLive
manages such a big network is essential for developing largescale IPTV applications in the future. There are several measurement studies about PPLive characteristics [15][18].
In these papers, authors focused on evaluating PPLive performance such as channel population, user arrivals and departures, user geographic distribution. They also evaluated
video trafc, video TCP connections, and user-perceived quality. These studies concentrated on measurement rather than
relationship between channel characteristics, user behavior,
and overlay characteristics. Nevertheless, there is no research
on how channel properties and user preference inuence the
system overlay. Our study attempts to ll this gap

Results obtained from our experiments indicate that PPLive
overlay characteristics differ from those of p2p le sharing.
Our major ndings are that: (1) Unlike p2p le sharing users,
PPLive peers are impatient, (2) Channel Size variations are
larger than in p2p le sharing networks, (3) Average degree
of a peer in the overlay (i.e., its out-degree) is independent
of channel size, (4) Smaller PPLive overlays are similar to
random graphs in structure, (5) The availability correlation
between PPLive peer pairs is bimodal, i.e., some pairs have
highly correlated availability, while others have no correlation.
All the above conclusions, except (3), are markedly different
from the well-known characteristics of p2p le sharing systems
[6][4][5]

Studying PPLive is challenging because (a) of its size and
dynamism of viewing content and user population, and (b)
it is a proprietary protocol with few publicly known design
decisions. As a result, our study follows two principles: I.
Careful design and thorough validation of our crawler; II.
Careful denition of measured metrics in order to derive unbiased results and draw correct conclusions. These are explained
in more detailed in Sections II and III. Our hypotheses are
validated by extensive experiments with data crawled during
a period of 4 months, stretching from April 2006 until the end
of July 2006.


# PPLive Crawler

The below figure shows the actions of a PPLive node to join the network: (1) retrieve a list of channels from channel management servers
via HT T P; (2) for the interested overlay, retrieve a small set of member nodes from the membership servers via UDP; (3) use this
seed partner list to harvest (learn about) other candidate partners by periodically probing existing partners (and sometimes membership
servers) via UDP. If a PPLive node is inside NAT or firewalls, UDP in the above steps may be replaced by TCP.

![testing](https://github.com/lhvu2/pplive_crawler/blob/main/images/pplive_membership_prototol.png)

