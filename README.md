# PPLive Network

PPLive is the largest chunk-driven multimedia streaming
p2p overlay in the world. As of May 2006, PPLive had
over 200 distinct online channels, a daily average of 400,000
aggregated users, and most of its channels had several thousands of users at their peaks. The system was increasing
in popularity, especially in China and Asia. For instance,
during the Chinese New Year 2006 event, a particular PPLive
channel had over 200,000 simultaneous viewers. In our
experiments during July 2006, we observed that there are about
400 online channels on the English version page of PPLive. Understanding how a large deployed system like PPLive
manages such a big network is essential for developing largescale IPTV applications in the future.

PPLive streams live TV and video data through overlays of
cooperative peers. The PPLive system has multiple channels,
each of which forms its own overlay. Each channel streams
either live audio-video feeds, or movies according to a preset
schedule. A human user may join any channel via her client
machine, but the client machine could also be used to relay
feeds for channels other than the subscribed one (by the
PPLive protocol).

PPLive divides video
streams into chunks and distributes them via overlays of
cooperative peers. Each channel streams
either live content or a repeating prexed program, and the feed from the channel originates from a server. A user can join
at most one channel. When she does so, her client machine is
not only as a consumer of feeds from that channel, but may
also be chosen by the protocol to act as a relay for feeds from
other channels. By default, each PPLive client has a pair of
TCP and UDP ports to communicate with PPLive servers
and partners. A number of other TCP ports can be used by
the client to exchange video chunks during its sessions.

We face several challenges studying PPLive network since it is very difcult to
distinguish between the notion of user and client machine when studying PPLive overlay networks.
There are reasons for this: (1) PPLive users are free
to join, leave, and switch channels by accessing the PPLive
web interface or PPLive Net TV. (2) Due to NAT boxes and
firewalls, a user's client machine may change its IP or UDP
port number or both. (3) The proprietary PPLive system is
rumored to use the idea of inter-overlay optimizations;
as a result, a client machine may appear as a participant
in multiple overlays, including ones that the user is not
subscribed to. Moreover, studying PPLive is challenging because (a) of its size and
dynamism of viewing content and user population, and (b)
it is a proprietary protocol with few publicly known design
decisions. As a result, our study follows two principles: I.
Careful design and thorough validation of our crawler; II.
Careful denition of measured metrics in order to derive unbiased results and draw correct conclusions.


# PPLive Crawler Design Principles

## PPLive Protocols

Although PPLive is not opensource,
a little of its internal design decisions are known. Each PPLive
node executes two protocols, for (1) registration and harvesting
of partners, and (2) p2p video distribution. A PPLive node
maintains two kinds of partners: candidates and real
partners. The latter are used for exchanging video streams
via TCP connections, while the former is used to replace real
partners that have become unresponsive. For our study, we use
a capability where a node can be queried for its candidate and
real partner lists, and it returns this partner list in a message.
One difculty is that it is not known whether this returned
list is the full partner list, or a subset of it. Hence, we need
to define a notion of node degree and partner list that is
generic and covers both possibilities as discussed below.

![testing](https://github.com/lhvu2/pplive_crawler/blob/main/images/pplive_membership_prototol.png)

The above figure shows the actions of a PPLive node to join
the network: (1) retrieve a list of channels from channel
management servers via HTTP; (2) for the interested overlay,
retrieve a small set of member nodes from the membership
servers via UDP; (3) use this seed partner list to harvest
(learn about) other candidate partners by periodically probing
existing partners (and sometimes membership servers) via
UDP. If a PPLive node is inside NAT or firewalls, UDP in
the above steps may be replaced by TCP. Since PPLive a closed source system, to understand its protocols, we develop a multi-threading C/C++ program to work in socket level to communicate with PPLive membership server as well as peers in the same channels. The source code is available at [6].

## PPLive Overlay

We formally dene the PPLive overlay as a graph G = (V, E). Recall that each PPLive overlay
corresponds to an individual PPLive channel. Each node (or
peer) is dened as a given <IP, port> tuple and belongs to
V. Each partner (or neighbor) of this node, appearing in its partner list, then corresponds to an edge (or link) in E.

### k response degree

We are interested only in the outdegree of a node in the overlay, and henceforth call this simply
as degree. As discussed previously, we need to dene degree in a manner that is generic. The k response degree of a node is
dened as the aggregated set of partners returned in the rst k responses from a node that is repeatedly (once a second) sent
a query for its partner list. We use a default setting of k = 15, however we verify the generality of these results for smaller
values of k as well (Section VI-B). Henceforth in this paper, the term node degree stands for k response degree.

### Active Peer

The next challenge is to clearly dene when a peer is considered to be a part of a given overlay. This is
complicated by the fact that a user may not be subscribed to a channel that the user's client machine is participating in.
Further, some clients may be behind NATs or rewalls, and may not respond to a direct probe message.
Given an overlay G and a peer v, v is considered to be an active peer in G if either v appears in the membership list
for G at one of the membership servers, or v is present in the partner list of some other peer u that is also an active peer.
Notice that the denition is recursive. Formally, we define the predicate:

ACTIVE(v, G) = {v ??? Membership Server List for G} OR
{???u : ACTIVE(u, G) AND v ??? u.PartnerList(G)}

Note that this denition includes silent peers that may behind
firewalls and previous studies simply ignore these peers - that to our knowledge, is not correct. To justify the above denition, we perform two experiments below.

First, we measured the fraction of peers that were captured
by our crawler using
the above denition, but that did not respond to a direct ping.
We observed that that over 50% of the captured peers are non-responsive: it is
important to consider the characteristics of these peers as a
part of the overlay, and our denition does this

Secondly, we verify that the p2p membership services of the
PPLive protocol do in fact spread updates rather quickly. We
chose a large channel, and ran 10 globally-distributed instances
of our crawler, each at a randomly chosen PlanetLab node.
Under this, we first had a PPLive node join a particular channel
and ran the crawlers 15 seconds after the join. All the 10
crawlers captured the new node. Then we killed the PPLive
node, and 15 seconds later, ran the crawlers again. None of
the crawlers returned the node in its membership list. Thus we
are reasonably condent that the p2p membership services of
the PPLive protocol spread membership updates quite rapidly.

# PPLive Crawler Implementation

## Snapshot Operation

We are interested in measuring the following metrics: (1) session length of peers, (2) channel
size, (3) peer degree, (4) overlay clustering coefcient, and (5) availability correlations. In order to support this, we use our
crawler to develop a Snapshot Operation. This Snapshot is different from the notion of Chandy and Lamport's consistent snapshots.
Intuitively, the Snapshot Operation for a PPLive overlay attempts to capture the set of nodes present in the overlay over
a short period of time (O(minutes)), along with their partnerlists. It works by repeatedly fetching partner lists and querying
returned entries. Specically, the snapshot operation works as follows:

The initiator (either our Linux node or one of our PlanetLab nodes) first requests the initial peer list from one of the PPLive membership servers, and uses this to initialize a local round-robin list denoted as L. Next, the initiator then continuously scans the list L in a
round-robin fashion, by sending a request for partner list to each entry, and appending to L new peers (i.e.,
ones that it has not heard about before) received in the partner list replies. Finally the Snapshot Operation terminates when the initiator has
received fewer than k new peers among the last ??? peers
received as partner lists. We use k = 8, ??? = 1000 in all of our experiments; with this setting, the Snapshot
Operation typically takes between 3 to 8 minutes. To avoid ooding network with our messages, new snapshot operations are initiated only once every 10 minutes

To increase the coverage of our snapshot, we run it in parallel on multiple machines. We
observe that 10 machines cover about 98% of peers covered by 20 machines. Hence, we decided to use 10
PlanetLab nodes to run simultaneous Snapshot Operations.

## Partner Discovery
This operation is to obtain the k response degree of a node discussed above. For this, we repeatedly
request a peer (approximately once every second) to send its partner list. The first k received responses are aggregated to
create the k response degree (and k response partner list).


# Results

Results obtained from our experiments indicate that PPLive
overlay characteristics differ from those of p2p le sharing.
Our major findings are that: (1) Unlike p2p le sharing users, PPLive peers are impatient, (2) Channel Size variations are
larger than in p2p le sharing networks, (3) Average degree of a peer in the overlay (i.e., its out-degree) is independent
of channel size, (4) Smaller PPLive overlays are similar to random graphs in structure, (5) The availability correlation
between PPLive peer pairs is bimodal, i.e., some pairs have highly correlated availability, while others have no correlation.

The source code of PPLive crawler can be found at [6], its Snapshot Operation and Partner Discovery traces can be found at [7][8]. The crawler source code and the traces were downloaded more than 3700 times worldwide. We have published several scientific
papers [1][2][3][4][5] in this topic, using our PPLive crawler, which received more than 350 citations. 

# Reference

1. Measurement and modeling of a large-scale overlay for multimedia streaming, L Vu et al., QShine, 2007, 106 citations (https://scholar.google.com/citations?view_op=view_citation&hl=en&user=DTm6snUAAAAJ&citation_for_view=DTm6snUAAAAJ:u-x6o8ySG0sC)
2. Understanding overlay characteristics of a large-scale peer-to-peer IPTV system, L Vu et al., ACM TOMCCAP, 2010, 91 citations (https://scholar.google.com/citations?view_op=view_citation&hl=en&user=DTm6snUAAAAJ&citation_for_view=DTm6snUAAAAJ:qjMakFHDy7sC)
3. Measurement of a large-scale overlay for multimedia streaming, L Vu et al., HPDC, 2007, 54 citations (https://scholar.google.com/citations?view_op=view_citation&hl=en&user=DTm6snUAAAAJ&citation_for_view=DTm6snUAAAAJ:9yKSN-GCB0IC)
4. MIS: Malicious nodes identification scheme in network-coding-based peer-to-peer streaming, Q Wang, L Vu, K Nahrstedt, H Khurana, INFOCOM 2010, 71 citations (https://scholar.google.com/citations?view_op=view_citation&hl=en&user=DTm6snUAAAAJ&citation_for_view=DTm6snUAAAAJ:eQOLeE2rZwMC)
5. Identifying malicious nodes in network-coding-based peer-to-peer streaming networks, Q Wang, L Vu, K Nahrstedt, H Khurana, 2010, 36 citations, (https://scholar.google.com/citations?view_op=view_citation&hl=en&user=DTm6snUAAAAJ&citation_for_view=DTm6snUAAAAJ:NaGl4SEjCO4C)
6. PPLive Crawler source code, http://dprg.cs.uiuc.edu/traces/download_details.php?details=4, L Vu, 1340 downloads
7. PPLive Snapshot Operation trace, http://dprg.cs.uiuc.edu/traces/download_details.php?details=3, L Vu, 1306 downloads
8. PPLive Partner trace, http://dprg.cs.uiuc.edu/traces/download_details.php?details=2, L Vu, 1158 downloads
