---
title: "Transferring funds between different ILP implementations  \n  \nDraft proposal"
---

Document
========

Revision history
----------------

| Version | Date        | Reason                         | Author           |
|---------|-------------|--------------------------------|------------------|
| 0.1     | 23 Apr 2018 | Initial draft                  | Michael Richards |
| 0.2     | 16 Aug 2018 | Following Convening discussion | Michael Richards |

References
----------

The following references are used in this document

| Reference | Document                                        | Version | Date          |
|-----------|-------------------------------------------------|---------|---------------|
|           | Open API for FSP Interoperability Specification | 1.0     | 13 March 2018 |

Glossary 
---------

The following abbreviations are used in this document.

| Abbreviation   | Text                                                                                               | Reference                                                 |
|----------------|----------------------------------------------------------------------------------------------------|-----------------------------------------------------------|
| ALS            | Account Lookup System                                                                              |                                                           |
| CIP            | Cross-implementation Provider                                                                      |                                                           |
| DFSP           | Digital Financial Service Provider                                                                 |                                                           |
| DMZ            | DeMilitarised Zone: the area of a service which supports public access via internet connections.   |                                                           |
| ILP            | Interledger Protocol                                                                               | https://interledger.org/rfcs/0027-interledger-protocol-4/ |
| Implementation | A group of ledgers implementing the ILP protocol which are visible to each other over a single VPN |                                                           |
| MSISDN         | Mobile Station International Subscriber Directory Number: a subscription to a mobile network       |                                                           |
| oracle         | A specialist service providing directory services for identifiers of a particular type.            |                                                           |
| VPN            | Virtual Private Network                                                                            |                                                           |

Table of contents
-----------------

Introduction 
=============

This document describes an outline design to support transfers of funds between
parties who are customers of DFSPs which belong to different Mojaloop
implementations. It represents the content of a discussion which took place as
part of the Mojaloop PI-2 planning sessions held in Johannesburg, South Africa
on April 5th, 2018.

The discussion originally included multi-currency issues as well as cross-border
issues. However, the group decided that it was prudent to treat multi-currency
questions separately, and this document therefore discusses multi-currency
issues only where they impact directly on cross-border issues. In addition, the
group noted that the same issues would arise where more than one Mojaloop
solution was implemented in a single country; and the solution is therefore
characterised in the rest of this document as a cross-implementation solution.

The document takes the form of a journey through the stages of a transaction,
identifying the additional requirements for a cross-implementation solution at
each stage of the process. At each point, reference is made to the corresponding
sections of the Open API specification where this is feasible (see **1 above**.)

Schematic of a cross-implementation solution
============================================

The following schematic shows the proposed structure of a cross-implementation
payments system. In this system, Alice subscribes to a DFSP which is on the same
VPN as Mojaloop implementation 1. She wants to make a payment to Bob, who
subscribes to a DFSP on a different implementation (implementation 2.) The
Mojaloop protocol means that, in the normal run of things, it would not be
possible to transfer money between these systems. This proposal discusses how
this could be done.

Figure 1: schematic of a cross-implementation solution

Cross-implementation providers
==============================

The proposal makes use of a new type of entity: the cross-implementation
provider (CIP). The CIP implements a bridge between two implementations of
Mojaloop, as can be seen in the schematic in Figure 1. A cross-implementation
provider is a member of more than one Mojaloop implementation. In the schematic
above, the CIPs are both members of two Mojaloop implementations (Implementation
1 and Implementation 2,) but there is no restriction on the number of Mojaloop
implementations of which a CIP can be a member. A CIP will have a settlement
account in each of the Mojaloop implementations it supports, and will therefore
be able to transfer funds between the two systems.

The sequence of steps for transferring funds between Alice (who belongs to DFSP
1A in implementation 1) and Bob (who belongs to DFSP 2A in implementation 2) via
CIP 1 is as follows. Note that this sequence disregards charges for the sake of
simplicity.

1.  Alice’s account is decremented in DFSP 1A

2.  DFSP 1A’s settlement account is decremented

3.  CIP 1’s settlement account in implementation 1 is incremented.

4.  CIP 1’s settlement account in implementation 2 is decremented.

5.  DFSP 2A’s settlement account is incremented.

6.  Bob’s account is incremented in DFSP 2A.

It can therefore be seen that this arrangement leaves the settlement accounts in
both implementations consistent, allowing the CIP to make its own arrangements
in relation to its settlement accounts in each implementation.

The following sections describe the specific responsibilities of CIPs.

Routing lists
-------------

CIPs will be responsible for maintaining routing lists to enable participants in
a given implementation to obtain information about how to route requests to
DFSPs in other implementations. This responsibility has two parts: broadcasting
information about the DFSPs that the CIP has direct access to and maintaining a
routing table based on the information broadcast to it by other CIPs. The
following sections describe the proposed operation of these two functions.

### Broadcasting information

A CIP will be responsible for broadcasting information about the DFSPs which it
has access to. The CIP will send a broadcast message to each of the switches to
which they have access. The switch concerned will broadcast the message to each
of the CIPs which are attached to it. Given the complex example described in
*Figure 2*, this would imply the following broadcasts:

-   CIP1a2 will broadcast information about DFSP1a and DFSP1b (which it has
    access to in implementation 1,) and about DFSP2a and DFSP2b (which it has
    access to in Implementation 2). It will broadcast this information to the
    switches in implementation 1 and implementation 2.

-   CIP2a3 will broadcast information about DFSP2a and DFSP2b (which it has
    access to in Implementation 2,) and DFSP3a and DFSP3b (which it has access
    to in Implementation 3.) It will broadcast this information to the switches
    in implementation 2 and implementation 3.

-   CIP3a4 will broadcast information about DFSP3a and DFSP3b (which it has
    access to in Implementation 3,) and DFSP4a, DFSP4b and DFSP4c (which it has
    access to in Implementation 4.) It will broadcast this information to the
    switches in implementation 3 and implementation 4.

-   The switch for Implementation 1 will not need to forward any broadcasts,
    since there is only one CIP attached to implementation 1.

-   The switch for implementation 2 will forward the broadcast from CIP1a2 to
    CIP2a3, and will forward the broadcast from CIP2a3 to CIP1a2.

-   The switch for implementation 3 will forward the broadcast from CIP2a3 to
    CIP3b4, and will forward the broadcast from CIP3b4 to CIP2a3.

At the end of this process:

-   CIP1a2 will know how to reach all DFSPs in implementation 1, implementation
    2 and implementation 3 (both from CIP2a3.)

-   CIP2a3 will know how to reach all DFSPs in implementation 1 (from CIP1a2,)
    implementation 2, implementation 3, and implementation 4 (from CIP3b4.)

-   CIP3b4 will know how to reach all DFSPs in implementation 2 (from CIP2a3,)
    implementation 3 and implementation 4.

In order for each CIP to have knowledge of all the others, each CIP will forward
the routing information it receives from other CIPs, substituting its own
identifier for the identification it receives from the other CIPs. This is
equivalent to saying “you can get to this DFSP through me,” without needing to
expose the whole routing chain to other DFSPs.

### Receiving broadcast information

When a CIP receives a broadcast from a switch, it will store the results in a
routing table. The structure of this routing table will be defined more fully in
later versions of this document. For each entry in the broadcast that the CIP
receives, it will check the entry to see if it is itself part of the forwarding
chain. If it is, it will ignore the entry, since it will already know about the
DFSP identified in it. Otherwise, it will store the entry in the routing table.

In the example described in *Figure 2*, the CIP closest to Alice (CIP1a2) will
obtain information about how to reach Bob in the following way:

1.  CIP3b4 will broadcast references to DFSP4a, DFSP4b (Bob’s DFSP) and DFSP4c
    to CIP2a3. These references will be in an example form CIP3b4/DFSP4a,
    CIP3b4/DFSP4b and CIP3b4/DFSP4c

2.  CIP2a3 will re-broadcast references to these DFSPs to CIP1a2.These in the
    form CIP2a3/DFSP4a, CIP2a3/DFSP4b and CIP2a3/DFSP4c

3.  CP1a2 will now know that, in order to request a transfer to Bob’s DFSP, it
    will need to contact CP2a3.

4.  CP2a3 will know that, in order to request a transfer to Bob’s DFSP, it will
    need to contact CP3b4.

Identifying the destination party
=================================

The first stage in the transfer of funds is the identification of the
counterparty in the transaction: that is, the party who is to be credited. This
stage is described in Section 5.2 of the Open API specification (see **1
above**.)

In order for a cross-implementation solution to work, the directories used by
the ALS will need to be modified to include information about counterparties who
are represented by DFSPs in implementations other than the one who represents
the originating party. These modifications divide into two different categories,
as described below

Globally unique identifiers
---------------------------

There are some ways of identifying a counterparty which are reliably globally
unique: for instance, an MSISDN. In these cases, it is sufficient for the
originating party’s DFSP to request information from the ALS in the form used at
present. The expectation is that the oracle for MSISDNs and, by extension,
oracles for other globally unique identifiers, will contain information for all
subscribers who are associated with a DFSP which identifies the DFSP to which
they belong. If a subscriber does not belong to a DFSP, then the search will
return nothing. If, on the other hand, a subscriber does belong to a DFSP, then
the ALS will return the URL of the DFSP to which the subscriber belongs, as at
present. This form of search does not require any modifications to the existing
interface, although the consequences for the requesting DFSP of the information
returned will be different. These differences are described in Section 6 below.

Non-unique identifiers
----------------------

Other ways of identifying a counterparty are not reliably unique across all
possible implementations. An example is a national ID number, which might be
duplicated in another country’s ID system. In this case, additional information
must be supplied to disambiguate the ID number: to say, for instance, that this
is a national ID belonging to the Kenyan or the Peruvian national ID scheme.

Disambiguation will require the inclusion of an additional piece of information
to the submission to the ALS. One such additional piece of information which
already exists is the Party Sub ID or Type (defined in Section 6.3.27 of **1
above**.) However, the addressing examples given in Section 4.2 of **1 above**
suggest that this will not be sufficient. In one of the examples given there, an
employee of a business is identified by giving the employee’s name as a Sub ID,
while the business name is given as the main identifier. Since a business name
will not be globally unique, the Sub ID will not be available to define the
context in which the non-unique identifier is to be evaluated. It will therefore
be necessary to use an addition identifier (for instance, “IdentifierContext”)
to specify the context in which a non-unique identifier is to be evaluated.

In the case of globally unique identifiers, a single instance of an oracle is
assumed to be capable of including information about all members of that
information type who are represented by a DFSP. Maintenance of the list
therefore devolves on a central authority, and no individual implementation need
assume responsibility for it. With non-unique identifiers, however this is not
the case. The model for the maintenance of directories described in **1 above**
suggests that each DFSP will be responsible for updating its local ALS (if there
is one) using the /participants URI (see Section 5.2.) this is the only model
that really makes sense, as it is hard to see how a properly segmented global
directory of (for instance) business names could be created and maintained.

Two possibilities suggest themselves. These are outlined in more detail below as
a stimulus to further discussion.

### Registration 

In order to implement a cross-border model, each Mojaloop system will be
registered with all other Mojaloop systems with which it will support
interactions. This is required to support the provision of retail quotations for
transfers (see Section 6 below.) As part of the registration process, a Mojaloop
instance could register a callback to enable it to receive directory updates for
non-unique identifiers from the Mojaloop instance with which it was registering,
and could register a callback from the Mojaloop service with which it was
registering to enable it to update that service with new directory input as it
became available. This would enable it to receive ALS updates from all the
participating Mojaloop instances with which it was connected as they were made,
and to broadcast changes to its own directory to all interested parties. This
would require changes to be made to the operation of the ALS, but would not
require further API changes.

### Consolidation 

As described above, each globally unique identifier will have an oracle service
which will provide a global view of identifiers and the DFSP to which each
belongs. An alternative to the distributed system of directories for non-unique
identifiers described above would be to maintain a consolidated service to be
accessed via an oracle, which would be updated by each subscribing ALS as it
received updates from the DFSPs which used it. This would mean that a global
oracle would gradually be built up, with the advantages over the registration
system described in Section 5.2.1 above that an update to the oracle would only
need to be made once; they would then be available to any ALS whatsoever. This
mechanism could also be used, of course, to keep oracles of unique identifiers
up to date as well.

The detailed design for implementing the selected maintenance method(s) is left
to a later stage of this document.

Obtaining a retail quote
========================

Once a subscriber has been identified, the next stage in the process of
transferring funds is the generation of a retail quote. This is a statement from
the target DFSP of the charges it proposes to levy to execute the transaction
requested by the originator. It is described in Section 5.5 of **1 above**. This
section discusses the process of obtaining a retail quote where the destination
DFSP is on a different Implementation from the originating DFSP.

When an oracle returns information to the ALS that it has identified a
counterparty, it does so in the form of the URL of the DFSP to which the
counterparty subscribes. Since all Mojaloop implementations use VPNs to secure
communications between their participants, they will only be able directly to
see DFSPs which belong to the same VPN. The ability to reach the URL returned by
an oracle is therefore a reliable indicator that the DFSP belongs to the same
Mojaloop implementation as the originator.

If the URL is not visible on the VPN, then it is a reliable sign that the
counterparty belongs to a different Mojaloop implementation from the originator.
In this case, the originating Mojaloop implementation needs to find a route to
the destination Mojaloop implementation. This will be done in the following way.

Each Implementation will register a URL which it can use to communicate with the
DMZ of each of the other Implementations with which it supports interoperation.
When a DFSP requests a quote from a payee’s DFSP, the ILP switch will check
whether the payee’s DFSP is visible to it or not. If it is not visible (i.e. if
the payee DFSP belongs to another ILP,) then the payer DFSP’s ILP will ask each
of the other ILPs with which it supports interoperation whether the payee DFSP
is known to them or not. If any of the queried ILPs recognises that the payee
DFSP, then it will respond positively.

The payer ILP will then request a retail quote from those ILP switches which
responded positively. The payee DFSP will create the quote and seal it in the
normal way, and will return it to its own ILP switch. The payee ILP switch will
return the sealed quote to the payer ILP switch, which will then return it to
the payer DFSP in the normal way. No changes will be required to the existing
interface.

The description above rests on the following assumptions:

1.  According to this description, a DFSP can belong to more than one ILP. This
    would be the case if a DFSP could also act as a CIP.

2.  We assume that a DFSP can only belong to more than one ILP if it also acts
    as a CIP.

3.  The existing interface assumes that only one quote will be returned for each
    request. This assumption is valid as long as all DFSPs in fact belong to one
    and only one ILP.

Authorizations 
---------------

The considerations detailed in this section will also need to apply to the
*/authorizations* resource (see Section 5.6 of **1 above**.) If an authorisation
is required from a payee, the payee DFSP will need to be contacted in the same
way as for the provision of a quote.

Obtaining a wholesale quote
===========================

When a retail quote has been obtained from the payee DFSP, it will be augmented
with a wholesales quote. A wholesale quote adds charges which are paid by the
payer DFSP to cover the transport of the transaction to the payee DFSP. In order
to calculate these charges, it is necessary to work out available routes between
the payer and payee DFSPs, rather than going directly to the payee DFSP, as was
the case with the retail quote. This section covers the way in which a payer
DFSP will obtain wholesale quotes.

The process of obtaining a wholesale quote also raises the question of
regulatory compliance. For the sake of simplicity, this is dealt with separately
in Section 7.2 below.

Once a record has been returned to the originating DFSP confirming that the
requested identifier has been matched to a DFSP somewhere in the world, a
further question remains to be answered: is it possible to map a route by which
the requested transfer can reach the destination? This section addresses the
means by which this question can be answered.

To recap: when an oracle returns information to the ALS that it has identified a
counterparty, it does so in the form of the URL of the DFSP to which the
counterparty subscribes. Since all Mojaloop implementations use VPNs to secure
communications between their participants, they will only be able directly to
see DFSPs which belong to the same VPN. The ability to reach the URL returned by
an oracle is therefore a reliable indicator that the DFSP belongs to the same
Mojaloop implementation as the originator.

If the URL is not visible on the VPN, then it is a reliable sign that the
counterparty belongs to a different Mojaloop implementation from the originator.
In this case, the originating Mojaloop implementation needs to find a route to
the destination Mojaloop implementation.

The following diagram shows an example which is used to illustrate this
discussion:

Figure 2: Multi-implementation example

In this illustration, Alice is still intending to send money to Bob, but her
route is of necessity slightly more complicated. Alice’s DFSP is attached to
Implementation 1, while Bob’s DFSP is attached to Implementation 4. The
following explains how Alice’s DFSP can find a route to Bob’s DFSP.

The payer DFSP will identify a CIP in the implementation that it belongs to. It
then queries that CIP’s routing table to find out whether the CIP supports
access to the payee DFSP. In the example, DFSP1a will ask CIP1a2 if it knows
about DFSP4b. It can be seen from the procedures described in Section 4.1 above
that CIP1a2 will have this information, and it will respond positively. DFSP1b
will therefore generate a request for a retail quote and send it to CIP1a2.

CIP1a2 will then check for DFSP4b in its local environment; it will not be
found. So CIP1a2 will send a request to CIP2a3 to ask if it can route the
request to DFSP4b. CIP2a3 will respond positively, so the request will be
forwarded to CIP2a3.

CIP 2a3 will go through the same process, and will route the request to CIP3b4.
CIP3b4 will recognise that it has direct access to DFSPb4, and will complete the
routing for the retail request. It will return its results to CIP2a3, which will
return its results to CIP1a2.

Routing
-------

The process described above creates assumptions about the route that a transfer
will follow to move from the payer DFSP to the payee DFSP. In the situation
described in Figure 2, only one CIP manages the transitions between each
implementation and therefore there is only one possible route from Alice’s DFSP
to Bob’s DFSP. If it were possible for more than one CIP to manage transfers
from one implementation to another, then alternative routes might be returned.
It is therefore important for a transfer to carry as part of its payload
information about what route it is intended to follow.

### Selecting a quote from alternatives

If alternate routes are possible and hence alternate quotes are produced, some
means of selecting a quote will be required. The following strategies might be
used.

#### DFSP decides

A CIP might PUT more than one response to a request for quotation (if it is the
sole CIP on the payer DFSP’s implementation but there are multiple CIPs
elsewhere in the chain;) or multiple CIPs might PUT one or more responses to a
request for quotation (if there is more than one CIP on the payer DFSP’s
implementation.) It would be the responsibility of the payer DFSP to decide
which of the responses to use as the basis for its transfer.

#### Switch decides

In a given implementation, responsibility for this decision might be delegated
to the switch to which the payer DFSP is connected. The switch could follow
scheme rules or rules explicitly delegated to it by an individual DFSP (in which
case a given implementation could support a mix of this case and 7.1.1 above.)
Example rules might be: lowest overall cost or fewest hops.

#### CIP decides

It would be possible to delegate responsibility for this task to the CIP or CIPs
serving the implementation of which the payer DFSP is a member. However, this
would still leave room for ambiguity in cases where the implementation of which
the payer DFSP is a member has more than one CIP. We therefore omit this
alternative in future discussions.

Neither of the first two options would require a change to the existing API
specification. In the first case, participant DFSPs would need to ensure that
their implementations dealt correctly with multiple responses to a request for
quotation; in the second case, scheme-specific implementations of the switch
would ensure that only one response would be returned to the payer DFSP who
requested the quote.

### Routing table

A routing table could be attached to a request for quotation and filled in as
part of the process by which the materials are gathered for a wholesale quote,
as described above.

A routing table would be most conveniently implemented as part of the ILP
packet. The ILP packet contains information for use at a lower level than the
information which is communicated between the participating DFSPs, and it
already contains a flexible extensions structure which can be used ot store this
information in the extensions element, which is more flexible than the
extensions element in the Open API definition. This would require a change to
the API definition as given at present, since the ILP packet is not currently
part of a request for quotation: it is only attached as part of the response.

Assuming that no change to the ILP packet specification were required, the
extensions element could include a routing object, to which CIPs could add their
addresses as the route is traced. The response can therefore be returned by each
participant CIP finding its own address in the routing table and returning the
response to the participant before it in the list.

An example JSON structure for the routing table constructed in the example might
be:

"routing"**: [**

"point"**: {**

"id"**:** "CIP1a2"

**},**

"point"**: {**

"id"**:** "CIP2a3"

**},**

"point"**: {**

"id"**:** "CIP3b4"

**}**

**]**

Two further questions need to be dealt with as part of this discovery. These are
dealt with below.

Charges
-------

A CIP may make a charge for processing a transaction. If it does, then it will
need to include that charge in its response; and implementation switches will
need to include the charges in the route lists that they eventually return to
the payer DFSP.

It may well be that in this world a CIP may make its charges in a currency which
is not used by either of the parties to a transaction. Imagine, for instance,
that a US-based organisation decides to make itself a world hub for
inter-implementation transactions, such that any payer DFSP can reliably find a
payee DFSP in one hop by routing the transaction through its hub. If the world
hub CIP levies its charge in USD, how will that charge in fact be levied? When
the transaction arrives at the hub, one would expect that the charge will need
to be levied in a known currency and at an exchange rate fixed at the time of
quotation. These questions are addressed in Section 9 below.

Charges should not be visible to other CIPs in the routing chain. Since this is
the case, charges will need to be encrypted when they are added to the routing
table by a CIP. The mechanism for managing encryption is described in Section 10
below.

For the purposes of illustration, an example of how the routing table might be
extended to include charges is given below. Values are not shown encrypted.

"routing"**: [**

"point"**: {**

"id"**:** "CIP1a2"**,**

"encryptedStuff"**: {**

"charge"**:** 0.25

**}**

**},**

"point"**: {**

"id"**:** "CIP2a3"**,**

"encryptedStuff"**: {**

"charge"**:** 0.3

**}**

**},**

"point"**: {**

"id"**:** "CIP3b4"**,**

"encryptedStuff"**: {**

"charge"**:** 0.2

**}**

**}**

**]**

It will be the responsibility of the payer DFSP to aggregate the charges levied
by the intermediate parties, and to add them to the amount being transferred.
This must be done at the level of the transfer amount on the transfer request
(see Table 22 of **Ref_1** above,) since this is the value used by switches in
ledger calculations. Each intermediate party is then licensed to deduct its fee
from the transfer as it processes it. When the transfer reaches the payee DFSP,
the fee amount specified in the transaction should be the same as that
originally specified by the payee DFSP when it responded to the request for
quotation.

Regulatory compliance
---------------------

When a CIP handles a transaction between two parties, it may make itself liable
to demonstrate that it has taken proper steps to assure itself that the
transaction complies with the regulations in force in the jurisdiction where the
CIP is implemented. These regulations may differ widely between jurisdictions,
and any solution to this problem should therefore be decentralised to the
providers themselves, rather than rely on a central authority. The problem may
not need to be solved solely for CIPs, since it will also apply to other
participants in the execution of a transaction, such as providers of FX
services; however, the term CIP is used for simplicity in the rest of this
discussion. The proposal for meeting this requirement is as follows.

As a wholesale quote is being prepared, a CIP should be able to add to its
response, as well as the charge described in Section 7.1 above, a statement of
the information required from the parties to the transaction to meet the
requirements of the provider’s regulatory regime. This statement of regulatory
requirements should contain a variable number of categories for which
information is requested.

**Assumption**: at some point we will need an overall definition of terms, such
that participants will be able to understand each other’s requests and map them
onto their information. This definition will be extensible, so that new
requirements from new or existing regulators can be included.

If the CIP is asking for information about the payee, then the payee DFSP will
parse the information requests and fill in those for which it has information.
For any items which are requested for which the payee DFSP does not have
information available, the payee DFSP will make a standard response stating
this.

If the CIP is asking for information about the payer, then the requests will be
returned along with the route and charge information. It will be the
responsibility of the payer DFSP to parse the information requests for the
selected route and to fill in those for which it has the requested information.
At this point, matching regulatory requests with the information available to a
DFSP may become an important part of selecting a route. In any case, the
information will be encrypted using a key provided by the CIP in question, and
will be included in the information accompanying the transaction when it is
executed. It will be the responsibility of each CIP through which the
transaction passes to decrypt the information using its private key, check it
for completeness and either forward the transaction or reject it depending on
the result of the check.

Executing a transaction
=======================

When the payer DFSP has selected a route, it will execute the transaction. This
process is described in Section 5.7 of **1** above.

The execution process will be the same as at present, with the following
exceptions:

1.  The initiator of the transaction will need to separate the definition of the
    ultimate recipient of a transaction (the payee DFSP) from the definition of
    the next recipient of the transfer request. This could be accomplished by
    putting the address of the CIP in the destination field of the ILP packet
    attached to the transfer request. The recipient CIP will look up the next
    destination using the method defined in Section

2.  The initiator will send an amount which is equal to the amount specified by
    the payee DFSP as part of the quotation process plus any charges levied by
    intermediates on the specified route.

3.  When an intermediate receives a transaction for forwarding, it will perform
    the following actions:

    1.  Decrypt the compliance information, if there is any, using the private
        key. Persist the compliance information.

    2.  Check the compliance information for correctness. If the information is
        incorrect or insufficient, cancel the transaction and return the
        cancellation to the forwarder.

    3.  Remove the amount of the charge it has agreed to levy from the
        transaction amount.

    4.  Forward the transaction to the next intermediate. Reservations in the
        relevant ledgers will be handled by the switches through which the
        transfers are routed.

4.  When the eventual destination DFSP has confirmed or declined the
    transaction, the confirm or decline message will be passed back down the
    chain of intermediaries.

5.  When an intermediate receives a confirmation message, it will forward the
    confirmation to the provider before it in the chain.

6.  When an intermediate receives a cancellation message, it will cancel the
    funds reservations in its ledgers for the charge and the residual amount,
    and will return to the provider before it in the chain.

This concludes the changes required to support intermediate processing and
charging for inter-implementation transfers.

Currency conversion support
===========================

This section describes how currency conversion might be supported in a Mojaloop
environment. It is a hypothesis which is subject to further discussion by the
wider Mojaloop community. The question of currency conversion is separate from,
though it is clearly related to, the question of cross-implementation support;
however, it is possible for currency conversion support to be required in a
single implementation where that implementation crosses currency boundaries.

This discussion is based around the following assumptions:

1.  Currency conversion is only required in switch-based environments

2.  Currency conversion will be provided by DFSPs and not, for instance, by
    services within the switch.

3.  DFSPs which provide currency conversion services will be able to advertise
    their provision of such services, and their ability to perform specific
    currency conversions (e.g. ZAR to USD, KES to GBP etc.)

4.  Currency conversion will be executed either by the payer DFSP or by the
    payee DFSP. Intermediaries will not perform currency conversions on a
    transfer. Questions relating to the levying of fees by an intermediary in
    currencies other than the payer or payee currency are dealt with in section
    TODO.

Nomination of receiving currency
--------------------------------

At present, the definition of the PartyIdInfo complex type (see Section 7.4.13
of **Ref_1** above) which is returned from a call to the **/parties** resource
does not include any information about the currency to be used to remit funds to
the party. This information can at present only be obtained by issuing a **GET**
call on the **/participants** resource for a specific party and currency and
seeing whether or not information is returned.

In order properly to support cross-currency conversion, the definition of the
PartyIdInfo complex type should be extended to support the definition of the
currency or currencies in which the party can receive or pay funds.

Determination of currency conversion requirements
-------------------------------------------------

The following process will be followed to determine whether currency conversion
is required for a transfer or not.

1.  When the payee party is identified using the **/parties** resource, the
    payer DFSP will be able to ascertain whether the payer party can support a
    transfer in (one of) the currencies in which the payee party can receive
    funds.

2.  If the payer party and the payee party can both support transfers in the
    same currency, then that currency should be used for the transfer and no
    conversion is required.

3.  If the payer party cannot support transfers in the payee party’s currency,
    then:

    1.  The payer party may still nominate that the payee is to receive funds in
        the payee party’s currency.

    2.  If the payer DFSP wants the payee DFSP to perform the currency
        conversion, then it should denominate the request for quotation in the
        payer party’s currency.

    3.  If the payer DFSP wants to perform currency conversion itself, then it
        should:

        1.  Select a currency conversion provider and request a currency
            conversion to return the amount of the destination currency to be
            transferred.

        2.  Denominate the request for quotation in the payee party’s currency,
            using the amount returned from the currency conversion provider.

        3.  Add an entry to the routing table (see Section 7.1.2 above) to pass
            the transfer to the selected currency conversion provider.

    4.  If, when the request for quotation is received by the payee DFSP, it is
        denominated in a currency which the payee party cannot support, the
        payee DFSP should assume that the payer DFSP wants the payee DFSP to
        perform the currency conversion. It should:

        1.  Select a currency conversion provider and request a currency
            conversion to return the amount of the destination currency to be
            credited.

        2.  Change the amount in the request for quotation to the converted
            amount and currency.

        3.  Add an entry to the routing table (see Section 7.1.2 above) to pass
            the transfer to the selected currency conversion provider.

Foreign exchange providers
--------------------------

The Mojaloop system will allow a DFSP to identify itself as a Foreign Exchange
Provider (FXP). An FXP will provide the following services:

### Specify currency support

When an FXP is registered with a scheme, it will specify the currency
conversions that it wants to support. These conversions will be stored by the
switch as part of the information relating to the FXP.

### Ledgers

An FXP will be required to maintain a ledger at the Mojaloop switch for each
currency for which it offers conversions. This will allow the movement of funds
associated with currency conversions to be tracked in the same way as other
transfers. In the example given in Figure 3, this will work in the following
way:

1.  DFSP 1A’s USD ledger at the switch will be debited by \$100.

2.  FXP1’s USD ledger at the switch will be credited by \$100.

3.  FXP1’s EUR ledger at the switch will be debited by €88.

4.  DFSP 1B’s ledger at the switch will be credited by €88.

The consequence of this architecture is that settlement with FXPs will proceed
in the same way as settlement between other DFSPs in the system: FXPs will be
paid, or required to pay, the net amounts in each currency of the transfers they
have processed during a given settlement period.

### Quote for currency conversion

If a DFSP wants to obtain a quotation for a currency conversion, it should be
able to send a request for a currency conversion. This should have a similar
form to an existing quotation request, and should contain at a minimum:

1.  The payer DFSP

2.  The payee DFSP

3.  The amount in the source currency, including a specification of the currency
    to be used

4.  The amount in the destination currency, including a specification of the
    currency to be used.

5.  A statement of the validity period required.

Either the source amount or the destination amount should be present, depending
on whether the requester wants to specify the amount that should be sent or the
amount that should be received. If both amounts are present, then the FXP should
assume that the source currency is the one specified, and that the destination
amount can be changed. The currency should be present for both source and
destination. If one or both currencies are absent, or if they are the same, then
the request for quotation should be rejected.

The FXP will assume that payee DFSP involved in the transaction is capable of
receiving funds in the currency denominated. It does not need to test or confirm
this assumption.

The recipient of a currency quotation will respond in a way analogous to the
current **/queries** resource: that is, it will return the quotation request
with both source and destination currencies filled in, and with an ILP packet
which can be used by the FXP to verify the *bona fides* of the transfer request.

This structure will support both currency conversions in which the source and
destination currencies are with different payees, and conversions where the
payer and payee are the same. This will allow DFSPs to fund bulk currency
conversions if they support both currencies, while also not requiring DFSPs to
support both currencies if the payee supports a different currency.

### Executing a currency conversion

When a DFSP has received a response to a request for a quotation for currency
conversion, it can include the FXP in its list of intermediaries when it issues
a request for transfer using the transfer resource. The information required by
the FXP to execute the transaction is limited to:

1.  A UID identifying the transaction

2.  The ILP data packet and condition which were returned from the request for
    quotation.

>   This information will be included in the extensions element of the ILP
>   packet. An example of how this might look is shown below:

"routing"**: [**

"point"**: {**

"id"**:** "FXP1"**,**

"unEncryptedStuff"**: {**

"transactionId"**:** "11436b17-c690-4a30-8505-42a2c4eafb9d",

" ilpPacket":
"AQAAAAAAACasIWcuc2UubW9iaWxlbW9uZXkubXNpc2RuLjEyMzQ1Njc4OYIEIXsNCiAgICAidHJhbnNhY3Rpb25JZCI6ICI4NWZlYWMyZi0zOWIyLTQ5MWItODE3ZS00YTAzMjAzZDRmMTQiLA0KICAgICJxdW90ZUlkIjogIjdjMjNlODBjLWQwNzgtNDA3Ny04MjYzLTJjMDQ3ODc2ZmNmNiIsDQogICAgInBheWVlIjogew0KICAgICAgICAicGFydHlJZEluZm8iOiB7DQogICAgICAgICAgICAicGFydHlJZFR5cGUiOiAiTVNJU0ROIiwNCiAgICAgICAgICAgICJwYXJ0eUlkZW50aWZpZXIiOiAiMTIzNDU2Nzg5IiwNCiAgICAgICAgICAgICJmc3BJZCI6ICJNb2JpbGVNb25leSINCiAgICAgICAgfSwNCiAgICAgICAgInBlcnNvbmFsSW5mbyI6IHsNCiAgICAgICAgICAgICJjb21wbGV4TmFtZSI6IHsNCiAgICAgICAgICAgICAgICAiZmlyc3ROYW1lIjogIkhlbnJpayIsDQogICAgICAgICAgICAgICAgImxhc3ROYW1lIjogIkthcmxzc29uIg0KICAgICAgICAgICAgfQ0KICAgICAgICB9DQogICAgfSwNCiAgICAicGF5ZXIiOiB7DQogICAgICAgICJwZXJzb25hbEluZm8iOiB7DQogICAgICAgICAgICAiY29tcGxleE5hbWUiOiB7DQogICAgICAgICAgICAgICAgImZpcnN0TmFtZSI6ICJNYXRzIiwNCiAgICAgICAgICAgICAgICAibGFzdE5hbWUiOiAiSGFnbWFuIg0KICAgICAgICAgICAgfQ0KICAgICAgICB9LA0KICAgICAgICAicGFydHlJZEluZm8iOiB7DQogICAgICAgICAgICAicGFydHlJZFR5cGUiOiAiSUJBTiIsDQogICAgICAgICAgICAicGFydHlJZGVudGlmaWVyIjogIlNFNDU1MDAwMDAwMDA1ODM5ODI1NzQ2NiIsDQogICAgICAgICAgICAiZnNwSWQiOiAiQmFua05yT25lIg0KICAgICAgICB9DQogICAgfSwNCiAgICAiYW1vdW50Ijogew0KICAgICAgICAiYW1vdW50IjogIjEwMCIsDQogICAgICAgICJjdXJyZW5jeSI6ICJVU0QiDQogICAgfSwNCiAgICAidHJhbnNhY3Rpb25UeXBlIjogew0KICAgICAgICAic2NlbmFyaW8iOiAiVFJBTlNGRVIiLA0KICAgICAgICAiaW5pdGlhdG9yIjogIlBBWUVSIiwNCiAgICAgICAgImluaXRpYXRvclR5cGUiOiAiQ09OU1VNRVIiDQogICAgfSwNCiAgICAibm90ZSI6ICJGcm9tIE1hdHMiDQp9DQo\\u003d\\u003d",

"condition": "fH9pAYDQbmoZLPbvv3CSW2RfjU4jvM4ApG_fqGnR7Xs"

**}**

**},**

"point"**: {**

"id"**:** "CIP2a3"**,**

"encryptedStuff"**: {**

"charge"**:** 0.3

**}**

**},**

"point"**: {**

"id"**:** "CIP3b4"**,**

"encryptedStuff"**: {**

"charge"**:** 0.2

**}**

**}**

**]**

>   The FXP can use this information to reconstruct the original request and
>   verify that it has not been tampered with. It can then execute the transfer
>   of funds and pass control to the next intermediary in the system, or to the
>   payee DFSP if there are no further intermediaries in the system.

Figure : Currency conversion

Security
========

This section proposes a way in which information could be made available to the
appropriate parties in a routing chain, while also ensuring that other parties
cannot read the information. It uses standard public/private key encryption
techniques.

These techniques need to meet the following requirements:

1.  When an intermediary attaches information to the request for quotation, that
    information should only be readable by the payer DFSP.

2.  When an intermediary attaches a request for compliance information from the
    payee DFSP to a request for quotation, that information should only be
    readable by the payee DFSP

3.  When a DFSP attaches information to the transfer request, that information
    should only be readable by the intermediary to which it relates.

These requirements will be met in the following way:

1.  When the payer DFSP issues a request to identify a party (i.e. a **GET**
    command on the **/parties** resource,) the information returned should be
    extended to allow the return of a public key to be used for the encryption
    of information to be read by the payee DFSP.

2.  When the payer DFSP issues a request for transfer, it will include the
    following information in the extensions element of the ILP packet:

    1.  A public key to be used by intermediaries to encrypt requests for
        information from the payer DFSP;

    2.  the public key that was returned from the payee DFSP in step 1 above and
        which is to be used to encrypt information requests for the payee DFSP.

3.  Intermediaries will use the appropriate public key to encrypt the
    information on charges and regulatory requirements which they are sending
    back to the payer DFSP or forward to the payee DFSP. This information will
    only be readable by the addressee DFSP using its private key, but the DFSP
    will be able to read all the intermediary responses.

4.  Each intermediary will include a public key for encrypting its own
    information in the (encrypted) information it sends back to either DFSP.

5.  When a DFSP constructs the regulatory information to be sent back as part of
    the transfer, it will use the public key transmitted by each intermediary to
    encrypt that intermediary’s information. This information will now be
    readable only by the intermediary for whom it is intended, using the
    intermediary’s private key.
