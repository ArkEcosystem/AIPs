<pre>
  AIP: 8
  Title: toonsbuf : an improvement of transaction protocol
  Authors: toons <moustikitos@gmail.com>
  Status: Draft
  Type: Standards Track
  Created: 2017-01-29
  Last Update: 2017-02-01
</pre>

Abstract
========

Developing a python API for ARK ecosystem, i read javascript source code from ark-js repository. Considering the way a transaction is granted by the ARK blockchain, it seems to me there are rooms for improvments within the core protocol.


Motivation
==========

Speed and simplicity are roots of efficiency. ARK stands to provide mass adoption of blockchain. Delegate nodes have to be very fast.

How speed can be improved ?

 - on server side : reducing javascript function calls
 - within ARK net : reducing transaciont data size


Rationale
=========

In fact, transactions are sent as [json data](http://www.w3schools.com/js/js_json.asp) via REST protocol, using POST method, on the ARK net.

ARK defines 6 types of transaction:

  1. token transaction
  2. delegate registering
  3. delegate up/down voting
  4. second signature registering
  5. multisignature registering
  6. interplanetary file system transaction 

Transaction is a set of information (acount, amount, name, fees and so on...) tied in a javascript object, signed using a private key and identified by an id. Note that singature and id are also stored in the same javascript object.


```
Example used for this AIP : AJWRd23HNEhPLkK1ymMnwnDBX2a7QBZqff send 1 ARK to AR1LhtKphHSAPdef8vksHWaXYFxLPjDQNU
This transaction is defined by a js object :
{
  senderPublicKey = '03a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a44c4a05e79f19de933';
  timestamp       = 21781590;
  asset           = {};
  type            = 0;
  amount          = 100000000;
  fee             = 10000000;
  recipientId     = 'AR1LhtKphHSAPdef8vksHWaXYFxLPjDQNU';
  signature       = '3045022100dc3ba689329c38aa5b76d186511d5123fbf3d4638d577cdf92c017a0db927ad502
201ba724bd978b347217c1418653604215f1ef343e809df1084360c13a75540352';
  id              = '3570708643510389756'
}
```

When doing a POST request via REST protocol, javascript object is serialized into json before going on the Internet. Basicly, it is sent as string describing the js object, gzipped or not according to what is asked in the request headder :

```
serialized transaction -> 318 chars = 1.272 ko with utf-8 charset defined in request header
'{"id":"3570708643510389756","senderPublicKey":"03a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a4
4c4a05e79f19de933","timestamp":21781590,"asset":{},"type":0,"amount":100000000,"fee":10000000,"re
cipientId":"AR1LhtKphHSAPdef8vksHWaXYFxLPjDQNU","signature":"3045022100dc3ba689329c38aa5b76d18651
1d5123fbf3d4638d577cdf92c017a0db927ad502201ba724bd978b347217c1418653604215f1ef343e809df1084360c13
a75540352"}'
```

When a delegate receive json data it has to:

  1. unserialize json into a js object
  2. transform js object into a serie of bytes (i call it ``getbyte``) using ``getBytes()`` function
  3. compute a sha256 hash of concatenation [``getbyte`` + ``signature``]
  4. check ``id`` with that hash [accept transaction if check succesfull]
  5. check ``signature`` with ``getbyte`` [apply the transaction if check succesfull]
  6. save transaction with the ``id``


In my opignon, a transaction can be reduced to its ``signature`` and its ``getbyte`` computed on client side. Knowing [structure](https://github.com/bitcoin/bips/blob/master/bip-0066.mediawiki#der-encoding-reference) of ``signature``, it is possible to quicly separate it from the ``getbytes``.  

```
signature + getbytes = 202 bytes = 0.202 ko (1.272 ko for json data)
30450221009a97b331d4f28a2000552005100b38ddbf556b350f615bb554e7195ab6ea84d502206108bed6d08d72247298
d7c0309ac30268029caf05750075234303e67bbf242c00e0624c0103a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb
3f3a44c4a05e79f19de933176545207451d8191c61047bef96d29581563b36910000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000e1
f50500000000
```

When a delegate receive json data it has to:

  1. create an ``id`` with data
  2. separate ``signature`` from ``getbyte`` and check signature [accept transaction into blockchain]
  3. retrieve transaction info and apply it
  4. save transaction with the ``id``

So what ?
=========

  - on server side : less operation to do
  - within ARK net : smaller data size per transaction
