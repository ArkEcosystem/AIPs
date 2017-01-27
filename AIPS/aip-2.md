<pre>
  AIP: 2
  Title: Number of Votes per Account
  Author: Guillaume Verbal <doweig@ark.io>
  Status: Draft
  Type: Standards Track
  Created: 2017-01-27
</pre>

Abstract
========

This AIP describes the details of a proposed change to the Ark Number of Votes per Account. In the codebase we inherited the number of votes per account equals the number of forging delegates. However we thinks that those two numbers should be different as they have different purpose.

The number of forging delegate is related to network speed and security and as such is driven by purely technical specifications.
The number of votes per account is used by and for the Ark token holders and therefore is people driven.

Motivation
==========

In this AIP we are trying to be as objective and neutral as possible. The project doesn't exist in a vaccuum so we will make references to observed behaviors, communities and other projects. Because the votes are made by people, we cannot only talk about technicalities in this paper. This paper is by no means a way to criticise or promote projects and/or behaviors but a necessary analysis of observed behaviors.

By learning from existing DPoS project, we have identified multiple recurrent players:
- `Pool` - Redistributes newly forged token back to voters depending of their vote weight (how many tokens they are holding). Keeps a percentage for server costs and profit.
- `Vote Alliance` - Group of tokoen holders composed of Whales and/or Regular holders seeking to maximize their ability to vote swap in order to become forging delegates.
- `Whale` - Holders of large amount of token. Likely to vote for pool and/or seek vote swap.
- `Regular Holders` - Most of the members of the community, having a sum of token that is not big enough to be considered a whale. Likely to vote for pool and/or seek vote swap.
- `Supported by the Core Team` - Those are community members very involved with the project that get voted in by the Core Team as most project have big bags of their own tokens, either as personal holders, funds, left over from crowfunding...

The goal of this AIP is to find a Vote Weight that provides the right balance between these entities as we believe they all provide value to the project.

Rationale
=========

Overview
--------

In the following section we will discuss the observed behaviors on a project where there are a lot of votes per account. The expected behavior when there are few. And finally our proposal.

Many votes
----------

We will use LISK blockchain as a study case. There's 101 available votes per account as well as 101 forging delegates. On this project, the community forging has been active for about two month at time of writing.

Let's see the different players behavior on this platform:
- `Pool` - Are a growing trend, there was no active pool when community forging begun, close to twenty now. Some pools are running multiple delegates
- `Whale` - Seeking vote swap to become forging delegate and uses remaining votes to vote for Pools to maximize profit. Most are not very involved in the community
- `Regular Holders` - Either seeking to join Vote Alliances, voting for Pools, trying to get votes from the Core Team
- `Supported by the Core Team` - Voted individuals contributing or planning to contribute to the project. The Core Team did not casting all their votes, about under fifty at time of writing

From analysing LISK project we can see that a high number of votes per account has contributed to a vibrant community that is seeking partnerships to exchange votes or want to contribute to the project. Furthermore, contributors are beeing rewarded by the Core Team as being able to forge, which is decupling the funds the Core Team has to improve the project. There are lots are benevolant and active members to organize meetups, improve the codebase, run testnet nodes, make tutorials. However 101 votes makes it difficult for voters to do proper due dilligence before voting, the high risk of one pool controlling multiple forging delegates is going against decentralization. Very few accounts are casting all their 101 votes, including the Core Team.

Finally, theoritically a Vote Alliance controlling 51 of more delegates could do at 51% attack (double spending).

Few votes
---------

There is no live project using this model at the moment. The idea is that votes are being "split" in a way that if you only vote for one account, that account will have 100% of your account weight. If you cast a second vote, the vote weight will be split 50-50 between the two accounts.

We think that this will have the following effects:
- `Pool` - Where most vote will go
- `Vote Alliance` - Won't exists. There is a counter incentive to vote swap with someone with less token than yourself as it will reduce your account's approval compared to if you directly and only vote for yourself
- `Whale` - Will vote for themself if they hold enough token to become a forging delegate. Will vote for a Pool otherwise
- `Regular Holders` - Will vote for a Pool
- `Supported by the Core Team` - Will be a lot less as votes are dilluted very quickly

The end result will be a group of forging delegates composed mostly of Pools and a few Whales. From other projects we can observe that Pools and Whales are not very involved in the project (similar to miners in BTC). The ability of the Core Team to influence who is forging is significantly impaired. Those 2 factors contributes to more decentralization, at the detriment of the community involvement.


Finding the right amount
------------------------

Too many votes promotes votes swap and alliances as keeping track of too many account requires more mental effort that anyone can afford. Too little promotes token holders to act in a selfish manner, only voting themselves or Pools.

Therefore the right amount should maximize both community involvement and decentalization.

Specifications
==============

We will note:
- `VA` the size of a `V`ote `A`lliance
- `MF` the `M`ajority of the `F`orging delegates
- `TF` the `T`otal of `F`orging delegate
- `NV` the `N`umber of `V`otes per account

Let's narrow down our choices from now, from everything written above, we believe that this number should be above 1 and less than the number of forging delegate. There 51 forging delegates in Ark so:

 `1 < NV < 51`

First of all, we do not want one Vote Alliance alone to be the mojority of forging delegates as they will be able to double spend. Double spending is possible when an entity is controlling the majority or more of forging delegates:

`1 < NV < MF`

`MF = 51 / 2 + 1`

`MF = 26`

`1 < NV < 26`

Double Spending here would still be possible if we only get a majority of a given Vote Alliance plus a few forging delegates outside of the group to agree on running an altered version of the client. So for extra safety we want 51% of the forging delegate to be more than the votes of two Vote Alliances (to handle worse case scenario).

So we want:

`MF = 2 * VA + 1`

`26 = 2 * VA + 1`

`25 = 2 * VA`

`VA = 12.5`

We end up with:

`1 < NV < VA`

`1 < NV <= 12`

So from a technical perpective, any number of votes from 2 to 12 makes sense.

From a human perpective, we want to be able to remember mentally for who we voted for.
We are very used to the number 10, it's easy to remember up to 10 things because we have 10 fingers to count on.

It also makes easy to make informed choices:
- The state of the project is calling for development, I want to fund `40%` toward development and `60%` toward my own profit. `4` votes to developers and `6` to pools
- New product is out, we need to promote it. `3` votes to meetup organizers, `2` to events funding and ads, `5` to pools
- Price is pumping, all `10` votes to Pool

With 10 votes it is easy to change priorities depending on what to project needs. It promotes monthy or even weekly votes update.

Therefore we propose the Number of Votes per Account to cast to be `10`

We hope that this will give the project the following results:
- `Pool` - Majority of the forging delegates for decentralization
- `Vote Alliance` - Will exists for middle size holders. Pushes the community members to exchange
- `Whale` - Will mostly vote for Pool. Promoting decentalization and increase in token price.
- `Regular Holders` - Vote for Pools and community contributors, will exchange about the project direction and future
- `Supported by the Core Team` - Only the top community contributors, will help very engaged community members to contribute full time on the project
