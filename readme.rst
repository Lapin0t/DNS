A Better Name Service
=====================


This is a draft describing a system for a name service with distributed trust.
It is inspired by the pgp web-of-trust, keybase_, the petname_ system and
possibly some other projects.


Concepts
--------

Each entity participating in the system will be identified by his master-key
(we will also call the entity an *account*). Each entity has an associated
linked list of blocks, each one containing an action on the account, the hash
of the preceding block and is signed by the account's master key. Following the 

Separation of claims and records
--------------------------------

A *claim* is a statement of identity, it must be inherently verifiable (or more
precisely have an absolute truth value). It is composed of 2 parts: a provider
and an identity, and also optionaly of a proof which is a helper to
automatically verify the claim in case it can be. The *provider* is an entity,
designated either by its key if it is part of the name system, or by a
consensual identifier. The *identity* is the name with which the provider
(uniquely?) identifies us.

Some examples:

- the provider ``github`` identifies me as ``lapin0t``, i can prove that with
  the URL of a gist containing this very statement signed by my private key
- the french state identifies me as ``Peio Borthelle (ID: 0123456789)``, as a
  proof i have my ID i can show IRL while demonstrating the knowledge of my
  private key.

I am not sure how to interpret a claim when the provider is the public key of a
physical individual (what is the associated identity? how to describe it?).

A *record* is a arbitrary blob of data we are associating with the account. It
can be interesting to make a record containing a pair of a string and a public
key (identifying an account), allowing to recreate the hierarchical dot-naming
of classical DNS: if account ``A`` has a record ``foo => keyB`` then account
``B`` can be designated by ``foo.A``.


Problem of the master keys
--------------------------

A *master-key* is a key which is allowed to sign a block. Multiple master-keys
seems to be a good idea (one per device, private keys don't move), but what if
one master key gets compromised?  Alas all my ideas boil down to complicated
ACLs (i would like to leave the system as simple as possible):

- some revocation needs approval by multiple master keys (say half of them)
- no master key but a delimited set of operations each key can do (basic being
  encrypt/decrypt, add identity claims, modify record)



.. _keybase: https://keybase.io
.. _petname: http://www.skyhunter.com/marcs/petnames/IntroPetNames.html
