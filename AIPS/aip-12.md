<pre>
  AIP: 11
  Title: Delegate Withdrawal
  Authors: dafty
  Status: Draft
  Type: Standard Track
  Created: 2017-09-07
  Last Update: 2017-09-07
</pre>

Abstract
========

This AIP describes the process involving delegate withdrawal ("resignation") from all delegate responsibilities in a non-destructive way, in terms of network stability and relations in the Ark community.

The proposed AIP is a novel concept, and would be the first implementation of it's kind in the DPoS community.


Motivation
==========

Network participants currently undertake an "opt-in" process when registering as delegate for the network. It is expected that participants opting in to the delegate role are committed to the responsibilities that the role requires. This assumption is supported by the delegate registration fee of 10 ARK, a significant investment.

A large proportion of delegates on the network offer profit sharing in order to secure enough votes to achieve a forging position. This is an important dynamic in this AIP, since these delegates usually have little to no control on whether they can achieve, maintain, and ultimately withdraw from an active forging position. This AIP specifically targets these profit sharing delegates. Delegates that have the capacity to support themselves (with a low or no requirement for profit share) are less likely to be affected, since they can withdraw themselves from a forging position by unvoting their own delegate.

Most delegates will eventually choose, or be forced to remove their delegate from a forging position. For these profit sharing delegates, there are several methods to try and remove a delegate out from a forging position:

* The delegate stops the forging node so that no more blocks are processed. Voters will eventually respond by unvoting that particular delegate.

This approach is clearly damaging to the stability of the network, causing slower block rounds and lowering the overall consensus needed to maintain a secure network.

* The delegate stops paying voters. Voters will eventually respond by unvoting that particular delegate.

This approach can be viewed as morally incorrect and can lead to bad reputation, a divide in the community and generally harm the image of the network as a whole.

This AIP attempts to outline a solution where delegates can remove themselves from a forging position with as minimal impact as possible.

Rationale
=========

A non-reversable transaction can be sent to the network to indicate the delegate should no longer be included in any future forging rounds. As a result, the rank of all delegates with an approval ranking lower than the disabled delegate will be incremented by 1, causing a delegate in the 52nd position to be promoted to 51st position and assume forging status.

Since the voting order is only counted at the start of each round, this process becomes seamless where, in theory, no blocks would be missed in the process of a delegate resigning, and a delegate being promoted to forging position. Once marked as resigned, it is impossible for the delegate to forge future blocks, leaving little room for dispute between delegate owners and profit share voters, since no future ARK has, or will, be generated.

Specifications
==============

1. Transaction type `RESIGNATION`

* Standard transaction fee of 0.01 ARK. This transaction type is only valid if the delegate is active (not resigned). There is little to no risk of it being used to spam the network. A high fee is not required.

* Only valid if the delegate is active (not resigned).

* First and second passphrases (if set) required to broadcast. A second passphrase protects against a malicious user resigning a delegate they do not control, if for example, a perpetrator was to gain remote access to a forging node (since only the first passphrase should be present on the server in order to forge).

**Rounds**

* Delegate are only marked as resigned and the query used to calculate the forging order of the next round can be adjusted  to exclude resigned delegates (`AND d.resigned = false`). This results in minimal impact and requires no modification of existing block history.

**Votes**

* The network should reject any transactions that involves placing a new vote for a resigned delegate. Transactions to remove votes from a resigned delegate are still permitted.

**Clients**
* If it is decided that the resignation transaction can be sent from a client such as ark-desktop, it must be clearly explained that it is a non-reversable transaction and will immediately remove the delegate from a forging position.
