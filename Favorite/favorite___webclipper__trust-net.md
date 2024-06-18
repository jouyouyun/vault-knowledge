---
title: TrustNet
data: 2024-06-18T17:12:13+08:00
categories:
  - webclipper
tags:
  - webclipper
  - security
  - trust
  - network
draft: false
origin-link: https://cblgh.org/trustnet/
---
_preface_

-   _for more information, [read the full 103-page Master’s Thesis report (~4MB)](https://cblgh.org/dl/trustnet-cblgh.pdf)_
-   _if you are able to support independent research, see [cblgh.org/support](https://cblgh.org/support.html)_
-   _code & repository links at the bottom of this article_

## Trust-based Moderation Systems

How do you remove malicious participants from a chat? For a set of participants, what are the steps needed such that the malicious participant is no longer visible by anyone in the set?

In the centralized context, removing a malicious participant is the action of a moderator. Usually it is one or two clicks, and the malicious participant has been removed for all other participants.

In a distributed context, there are many possible answers to this problem. The first and naive solution is to delegate the responsibility of removing the malicious participant to each individual participant. Thus everyone participating has to individually hide offenders. Viewed as an isolated case it works, but repeated instances will risk causing an outsize burden on the participants.

Traditional moderation systems grant a special privilege to the initiator of the chat group. This can become problematic for many reasons. The initiator may disappear, leaving the group moderation-less. Similarly, the initiator may be adept at starting new contexts, but lacking in skill concerning matters of moderation (e.g. assigning new moderators). Finally, issues may arise where previously good moderators have a falling out and start banning people, as demonstrated in increasing frequency by large community-driven Mastodon instances.

A subjective system, where participants can themselves decide who moderates on their behalf, sidesteps the mentioned problems. Additionally, the mechanism of freely allowing multiple people to moderate also spreads out the invisible care-giving labour required to keep a community free from abuse.

This work explores an approach to implementing a subjective, trust-based system.

What if participants could automatically block the malicious peer, if they discover that the peer has been blocked by someone the participant trusts? This is similar to the administrator from the centralized context, but more flexible. In the centralized context, if the administrator is misbehaving and a participant loses trust in them, their only options are to live with it, or to leave the group. In the system where you effectively choose who can moderate for you, you can also choose to revert that decision if your trust later proves to have been misplaced.

How this can be achieved will be explored in the rest of this post in the form of a new system for managing and interacting with trust, **TrustNet**.

## Trust Network

The proposed system, **TrustNet**, is a system for interacting with and managing trust. Underlying the system is a transitive trust algorithm. The system as a whole was originally intended for use in combination with peer-to-peer distributed chat systems, where peers assign trust (a value between 0.0 and 1.0; 1.0 being complete trust, 0.0 the complete absence of trust) to other peers.

#### Trust Spectrum

```shell
    Trust weight (t)  Semantic         Human-meaningful label
    ----------------------------------------------------------
    t == 0            no trust         New person
    0 < t < 0.25      low trust        Acquaintance
    0.25 >= t < 0.75  medium trust     Friend
    0.75 >= t < 1.0   high trust       Peer
    t == 1            complete trust   Partner
```

In the above plot, we can see how different trust weights have different meanings. A trust weight between 0 and 0.25 is equivalent to _low trust_. But using direct trust weights, or the semantic labels, does not meaningfully translate to any real life situation. Thus, any system that allows people to issue trust to each other should have labels which are natural for people to use, enabling them to interface with the system in a way that causes for less individual distortions.

For Alice’s semantics regarding a trust assignment of 0.65 might differ from Bob’s view of what a trust level of 0.65 means, but they both have notions of **friend** which is more likely to converge than pure floating point representations of trust.

_Note_: the labels that are to be used should ideally be specific to the trust area. It might make less sense to use the label _friend_ in a trust area of _music recommendations_ as compared to using the label _good taste_, for instance.

#### Trust Areas

Trust is assigned in a **trust area**. The trust area acts as a grouping for related trust assignments; trust is only transitive within a given trust area. Example of trust areas could be _moderation_ or _music recommendation_, for example.

Thus, the trust area captures the **context** that the trust is extended within; for outside of the realm of computers, we trust each other varying amounts depending on a given domain. Friends I trust for financial advice may have _horrible_ taste in music, for example.

In the remaining text, we will be talking about how TrustNet works within a given trust area.

#### Transitive Trust

With the issued trust assignments as a basis, a trust network is then discovered for a particular peer by following their outgoing trust assignments transitively. So, for a particular peer, we find the set of peers that they trust. For each peer in the set of trusted peers, get the set of peers that _they_ trust. And so on, for a maximum path-length between 3-6 nodes away from the root.

We now have a basic trust graph, with a starting node and the peers they (transitively) trust. The varying levels of trust between nodes are represented by weighted and directed edges. Using this weighted trust graph we can now calculate a list of the most trusted peers for the starting node.

## Trust Computation

To find the most trusted peers, we use [Appleseed](http://www2.informatik.uni-freiburg.de/~cziegler/papers/ISF-05-CR.pdf), a peer-reviewed algorithm and trust metric which was proposed in the mid 2000s by Cai-Nicolas Ziegler and Georg Lausen from the University of Freiburg. Appleseed operates on precisely the kind of weighted graph structure we have described, and it is also guaranteed to converge after a variable (but finite) number of iterations. Appleseed, once converged, produces a ranking of the most trusted peers.

Appleseed produces its rankings by effectively releasing a predefined amount of energy at the trust source, or starting node, and letting that energy flow through the trust relations. The energy pools up in the nodes of the graph, with more energy remaining with the more trusted peers. After the computation has converged, each peer has captured a portion of the initial energy, where the peer with the most energy is regarded as the most trusted. The initial amount of energy is also completely distributed across the nodes in the trust graph, meaning there are no losses after convergence.

The computed trust ranking contains the same nodes as the initial weighted trust graph. Thus, it is a list where the most trusted peers are at the top of the list, and the peers with the least amount of trust (but still trusted) are at the bottom. In order to make this list of peers actionable and more useful, TrustNet uses something called _ranking strategies_. A ranking strategy basically operates on the ranked list and produces a subet of it, according to the chosen strategy.

## Ranking strategies

One ranking strategy: from the list of peers, pick the most trusted peer. Or, more generally, pick the `N` most trusted peers. Another form of strategy would be to use a clustering algorithm to identify clusters of nodes with similar ranks, creating groupings of high trusted peers, as well as medium trusted and low trusted peers. This latter strategy is what TrustNet uses to find its most trusted peers.

#### Clustering Strategy

The clustering strategy relies on the [k-means clustering algorithm](https://en.wikipedia.org/wiki/K-means_clustering), breaking Appleseed’s produced ranking into three groups. The group containing the lowest trust ranks is discarded, and the remaining two are merged, representing TrustNet’s trusted peers.

#### Low Trust Graphs

One caveat with all strategies is that, while the rankings produce the most trusted peers for the given trust graph, we might be operating on a graph based in low amounts of trust. That is, a graph where the trust source has only issued trust assignments with low trust weights: 0.1 instead of 0.75, for example. In this case, we would still get a ranking where one peer has the highest amount of trust, but it would be a relatively low trusted peer in terms of the trust source. The reason we end up with a low trust graph has to do with how Appleseed’s rankings are computed. The rankings returned from Appleseed are effectively calculated in proportion to the initial set of trust assignments issued by the trust source. Thus, Appleseed’s trust ranking will contain more reliable results if the trust source has issued at least one higher trust assignment, as the rest of the trust graph’s edges will be placed in relation to the trust source’s higher trust assignment.

A meta strategy could take this into consideration by also looking at the initial trust weights. The strategy could return an empty list if it discovers that the initial peer does not have any direct outgoing trust assignments of medium or high weights. TrustNet implements this type of **threshold strategy**.

#### A Note on Trusted Peers

TrustNet uses a clustering technique to get the most trusted peers, in combination with the described trust threshold. Importantly, however, **any peer which has been directly trusted is added to the pool of most trusted peers**. Because, if you know somebody and you trust them, then of course they are to be regarded as trusted.

The trust weight, then, acts more as classifying a **similarity in judgement**, where a higher trust weight entails a greater similarity. Thus, higher trust weights can be seen as being granted a greater recommendation influence. You might trust your best friend, but you also know that they aren’t that reliable for recommendations. Thus you assign a lower trust weight to them, lessening their impact with regard to recommendations for others to trust, while still being able to rely on them for moderation.

## Usecases of a Trust Network

You have a network of peers (with varying levels of trust) and running Appleseed gets you a ranking of those peers. We run the computed rankings through a ranking strategy and end up with an actionable list of trusted peers.

We can use this curated list of trusted peers in multiple ways:

-   Implement moderation, essentially granting the trusted peers moderator powers to act on your behalf. This effectively creates a dynamic blocklist, with many sources. The peer moderators and their changes can also be revoked if abused: you stop trusting them, unassigning any previously assigned trust.
-   Regard the list as a set of trusted sources from which to request secondary indexes in a distributed network.
-   Use the rankings as a suggestion on whom to follow / become friends with / talk more with.
-   Filter the greater network to only show messages from the trusted network. Make it is togglable.
-   Allow pooling of trust behind actions: if many medium to low trusted (they are still trusted, just low amounts) peers commit to an action, then that collectively adds up to an amount of weight/trust behind that action.
-   Use the rankings to inform recommendations within a particular kind of domain (e.g. having a domain of ‘music taste’, and using the trust weight for each peer as an indication of music taste, higher being more aligned music taste/subjectively better music taste) .
-   Use rankings in combination with news articles/sources to vet ‘fake news’ from reputable sources.
-   Automatically fetch resources / images / binary blobs from those within your trust network, while resources from those outside of the trusted peers might be fetched manually.
-   Perform computations issued from within the trust network.

## The Replication Layer

The proposed system does not live in isolation, there are so called peers who have to perform trust assignments. These peers come from other systems, distributed protocols whose primary use at the moment are for chat. Systems like [Secure Scuttlebutt](https://ssb.nz/), a gossip based network, or [Cabal](https://cabal.chat/), a peer-to-peer chat platform.

**The TrustNet structure**

A peer in these systems typically consists of an ed25519 keypair, which writes messages to a log and shares the log with other peers. Each message is signed by the peer’s private key, establishing authenticity of the message. Any peer can store and transmit the log of any other peer, which is how information propagates through the system.

This replication layer differs between protocols, but what remains the same in these systems is that every peer locally stores all of the messages for any other peer it can see in the network. In Cabal, all peers connect to the same identifier and will eventually see each other. In Secure Scuttlebutt the replication layer consists of follow relations, where a peer will replicate the peers they follow, and often one or two hops further. If a client is configured for two hop replication that means that a peer will replicate every peer they follow and also every followed peer of that first-order peer.

The replication layer is fundamentally the database of the local peer, and thus the substrate from which the trust graph, with its trust assignments, can be constructed. So, to be explicit, the trust network (or trust layer) lives ontop of the replication layer; it is fed the replicated data.

A trust assignment would be another message type in these systems, one with a type that indentifies it as a trust assignment and which contains:

-   a **trust weight** (between 0.0 and 1.0),
-   a **trust target** (the public key which is being assigned trust),
-   the **trust area** (e.g. moderation, or music recommendations),
-   as well as the **trust source** (the id of the peer issuing the trust assignment).

To construct the weighted trust graph of a particular peer we would do the following:

-   In the peer’s log, find all messages which are trust assignments. By definition, these are the trust assignments issued by this peer.
-   For each trust assignment: get the target and find all of their trust assignments.
-   Repeat this until we run out of trust assignments, or we have a maximum path length from the root to a leaf node in the graph of no more than 3-6 nodes.

To update the trust graph, we keep track of the latest message of each peer’s log. When new messages arrive, scan them to see if there are any new trust assignments. If there are, signal that we need to reconstruct the trust graph.

Implementation-wise, one would reconstruct the graph after a timeout. The timeout is refreshed each time a new message comes in from a log. This ensures that the trust graph is not constantly being reconstructed, which could be the case if we recently reconnected to the Internet for the first time in a few weeks and are in the middle of syncing up with peers.

Once we have all of the trust assignments for the trust graph, we normalise them to the form that TrustNet consumes. After trust computation has been performed with Appleseed, and the resulting ranked list has been produced, and a ranking strategy has been used, we now have a list of trusted peers.

Using the trusted peers we can now look at what moderation actions they have issued, and automatically issue our own as a result of theirs.

## Subjective Moderation & Disambiguating Intentions

A moderation action has a set of semantics that need to be regarded in order for a subjective moderation system to function according to the expectations of its participants. We will illustrate the semantics by looking closely at the **hide** action, which hides all content originating from the hidden peer.

We propose that there are three modes of the hide action for a subjective moderation system: the **personal**, **network**, and **propagated** modes. The three modes can be implemented in various ways, but they need to exist in some form in order to disambiguate the **intention** of the hide. We will now detail the three modes and the intentions they are intended to represent.

#### Personal

A personal hide is issued for the local participant and does not necessarily signal anything other than the participant not wanting to see the hidden participant anymore. A personal hide may be **private**, i.e. not transmitted in a format readable by any other participant (that is, it is encrypted to the local participant only).

The personal hide exists to reflect that, while the hide action is issued for a valid reason, it is purely personal and does not by necessity mean that other peers should issue hides of their own.

#### Network

A network hide is issued by a participant to signal to others who trust them that they, too, should hide the hidden participant. Thus, the network hide is the mechanism by which one participant can signal and warn others of a malicious participant.

#### Propagated

A propagated hide is issued as a **result** of a network hide by another, trusted participant; i.e. it has been **propagated** from a network hide. Propagated actions should not result in further actions—it is propagat**ed**, not propagat**ing**.

The propagated hide needs to exist so that hides that were issued as a result of trust in another participant can be tracked to their origin and also revoked, if the trust has been lost.

## On Distrust

The proposed system is transitive, meaning that there will be indirect trust relations by way of those whom you trust directly. This works out in a lot of cases but, occasionally, someone you trust will trust someone else whom you have had bad experiences with, and now distrust. Distrust can be be implemented in multiple ways, the simplest (and most robust) is to remove the distrusted nodes from the trust graph before computing.

For social reasons, these distrust statements are advised to be stored in a way which is unreadable to other peers. This could be in an encrypted format, or purely locally, unpublished to the peer’s log. Thus, after having constructed the transitive trust graph, we perform a final search through it, comparing each node to the list of distrusted public keys we maintain locally. If we find someone we distrust, we remove their node and the subgraph that extends out from them.

The reason we remove people which the distrusted peer trusts is that it is difficult to make any statements, or draw any conclusions, concerning a distrusted node. Maybe you hate them as a person, which is why you distrust them. Or they are someone you like a lot, but they just happen to differ too much in judgement as compared to yourself, so you distrust them to exclude their judgement from the final trust network.

## Conclusion

TrustNet is a system for representing, and interacting with, computational trust. The system is comprised of a trust metric, a system for trust propagation, and it takes distrust into account. Most importantly, it provides a simple interface for using and interacting with the results of the trust propagation.

#### Report

If you wish to know more, or you have any questions, [**read the report**](https://cblgh.org/dl/trustnet-cblgh.pdf).

It may be long, but it is easy to read. Areas such as _public-key cryptography_, _peer-to-peer_ and _distributed systems_, [Cabal](https://cabal.chat/) and [Secure Scuttlebutt](https://ssb.nz/), _trust_ and _computational trust_, are all introduced, **without expectations of prior knowledge**. Appleseed and TrustNet, the core of this work, are of course also thoroughly introduced and described.

Particularly recommended are **Chapter 5 (Trust)**, **Chapter 6 (Appleseed)**, and **Chapter 7 (TrustNet)**, if you are already familiar with peer-to-peer systems and moderation.

#### Code

-   [Visit the TrustNet repository](https://github.com/cblgh/trustnet)
-   [Visit the Appleseed repository](https://github.com/cblgh/appleseed-metric)

Both repositories contain full documentation, and an assortment of tests.

#### Presentation

If you want to watch my thesis presentation, which describes the above with other words and with examples, [**see the presentation**](https://www.youtube.com/watch?v=bVkesgWZlTU).

In the presentation, I show an interactive demo I built as part of the thesis. If there is an interest, I can look at finishing the demo such that it can be put online for others to explore.

#### Fin.

Finally, for more independent research like this, and you are able: [**support me**](https://cblgh.org/support.html).

Thank you for reading,  
2020-06-29  
~cblgh