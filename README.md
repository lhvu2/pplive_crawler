# pplive_crawler

The below figure shows the actions of a PPLive node to join the network: (1) retrieve a list of channels from channel management servers
via HT T P; (2) for the interested overlay, retrieve a small set of member nodes from the membership servers via UDP; (3) use this
seed partner list to harvest (learn about) other candidate partners by periodically probing existing partners (and sometimes membership
servers) via UDP. If a PPLive node is inside NAT or firewalls, UDP in the above steps may be replaced by TCP.

![testing](https://github.com/lhvu2/pplive_crawler/blob/main/images/pplive_membership_prototol.png)

