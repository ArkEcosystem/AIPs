<pre>
  AIP: 10
  Title: Automatic Profit Sharing
  Authors: ryano
  Status: Draft
  Type: 
  Created: 2017-06-24
  Last Update: 2017-06-24
</pre>

Abstract
========

Delegates would be able to explicitly state their profit sharing in a public way such that the Ark they forge is 
automatically shared with their voters. 


Motivation
==========

Delegates should not have to use third party systems for profit sharing and given the wide spread use of profit sharing 
as a motivating for attracting voters, this should be a parameter built in and taken care of automatically. Voters are forced
to trust a delegate to pay. Small voters have their rewards consumed by unnecessary transaction fees. Existing delegates are
also forced to implement "fidelity" features to avoid pool hoppers. This AIP solves all of those issues. 


Rationale
=========
This has an additional advantage of allowing delegate profit sharing to be displayed with confidence in the voting portion of a wallet. 
Doing this also eliminates the need to do unnecessary transactions and incur unnecessary transaction fees for sharing forged
Ark with voters. Voters can accrue Ark immediately as their delegates do. Delegates will not need any form of voter fidelity because only
voters at the time a block is forged will receive Ark. 


Specifications
==============

A delegate, or node, would specify their promised profit sharing, from 0 to 1. When that delegate forges a block Ark is automatically added to their
voters accounts in the correct proportion. 
