<pre>
  AIP: *11*
  Title: *'Seed' instead of 'passphrase'*
  Authors: *Simon Downey @sleepdefic1t*
  Status: *Draft*
  Type: *Standards Track*
  Created: *2017-07-15*
  Last Update: *2017-07-15*
</pre>

# 

Abstract
========

This AIP proposes changing the API language of "passphrase" to "Seed."



Motivation
==========

Making the ARK UX more intuitive, effortless, and enjoyable.

 

Rationale
=========  

Using the word "passphrase" is confusing.

Most users are accustomed to a "passphrase" or password being something they set up to login to an account somewhere.
And while this word does describe just that; it's just too broad.

Using "Seed" would be a much more concice solution.

While "Seed" is absolutely a newer concept to most end-users;
it has the capacity to be very intuitive and informative.



Specifications  
==============

```diff
- Create second passphrase
+ Create 2nd Seed
```
#
```diff
- Enter the passphrase of your account
+ Enter your accounts' Seed
```
#
```diff
- Save your passphrase in a safe place!
+ Write your seed down, and keep it somewhere safe!
```
#
```diff
- Passphrase is not saved on this computer
+ Your seed is never saved on this device
```
#
```diff
- Passphrase does not match your address
+ That Seed does not match your address
```
#
```diff
- Passphrase is not corresponding to account
+ That Seed does not match an account
```
#
```diff
- Passphrases are not secured with high standard...
- Use this feature only on safe computers and for accounts you can afford to lose balance.
+ Seeds--by nature--are not very secure.
+ It's best to rely on them for smaller balances.
+ For larger balances that require heightened security,
+ choose a "cold-storage" solution like the 'Ark Ledger.'
```
