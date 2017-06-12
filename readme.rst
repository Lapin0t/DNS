=====================
A Better Name Service
=====================


This is a draft describing a system for a name service with distributed trust.
It is inspired by classical DNS, the pgp web-of-trust, keybase_, the petname_
system and possibly some other projects.

The goal here is to replace the current DNS system used to attribute global
names to computers on the internet, but doing it in a cryptographically
authentificated way it will also replace the current hierarchical TLS PKI and
will provide some basis to build an name system not only for computers but also
for humans. It does not seek to be compatible with anything, but care is taken
to allow it to exist alongside current systems (it can also obviously exist by
itself) and transition smoothly: external identity system (computer-based or
IRL) are a first-class concept.


Concepts
========

The main idea is to create a distributed PKI (like the PGP WoT or keybase) with
fine grained expression of trust links. This graph is is a generalization of
the DNS or TLS PKI trust hierarchy in the sense that no particular topology is
enforced. Moreover the root of trust is always self, ie one is free to trust
whoever he wants and noone is trust by default.

Each entity participating in the system will have two public names: one unique,
the public part of the master-keypair we'll call *master-key*, one non-unique,
a string we'll call *nick*. Concretely, an entity can be seen as a database and
is defined by a partially ordered set of transactions. Additionnaly, every
entity can maintain a private dictionnary, mapping a *petname* to a unique
entity (by it's master-key). These 3 namings together (master-keys, nicks and
petnames) enable to overcome the problem raised by zooko_ (see also `this
<stiegler_>`_ for more informations on petnames).


Transactions
------------

We define the ancestors of a transaction T as the set of transactions which are
smaller than T. A transaction contains:

- a class and some data depending on it
- a signature by the master-key

Besides that, each transaction except the initial one has a set of (hash of)
parent transactions which must be pairwise incomparable, it is defined to be
bigger than all of it's parents (and transitively of all their ancestors).

Transaction classes:

``initial``
   The initial transaction, associated data is the master-key and the nick.

``revoke``
   Undoes a past transaction, associated data is the hash of the transaction,
   which must be an ancestor.

``validate``
   This transaction is the equivalent of keybase's "tracking": the associated
   data is the hash of a ``claim`` transaction, having no common ancestor with
   the current one (ie from an other entity). It states that the current entity
   believes that the given claim is true after having seen a convincing proof.

``claim``
   See below.

``record``
   See below.


Separation of claims and records
--------------------------------

A *claim* is a statement of identity, it must be inherently verifiable (or more
precisely have an absolute truth value). It is composed of 2 parts: a provider
and an identity, and also optionaly a proof-hint which is a helper to
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
physical individual also part of this system (what is the associated identity?
how to describe it? isn't this redundant with the validation statements?).

A *record* is an arbitrary blob of data associated with the account. This can
be seen as the equivalent of DNS RRs. I am still thinking about the format,
CBOR_ might be a good candidate. 


Records, or how to express the current DNS and TLS-PKI features
---------------------------------------------------------------

Basically, everything happens as if every account is a DNS root. Their TLD is
either their public key, but that's cumbersome, or the petname that the user
attributed to them. This is enough to express simple (leaf) records: TXT-like
(containing text) or A/AAAA-like (containing an IP adress).

But just like a NS record, one can create a record with the master-key of
another entity as value: let's assume there are two accounts with master-keys
``A`` and ``B``, account ``A`` has a record ``"foo" => B``. Now account
``B`` can also be refered to as ``foo.A``.

So here we have the essence of DNS: domains are paths and we can point names to
text, an ip or an entity. But because each entity (the equivalent of name
servers) is first-of-all the bundling of identity claims and cryptographic
keys, we just have built a certificate infrastructure. Recall that a
certificate is just 3 things: an identity (eg public-key and some legal infos),
a claim (eg domain name) and a proof (eg signature of a trusted third-party).
Thus, each domain is self-certifying and an the communication with the target
of a domain should be negociated using the public key of the last entity in the
chain (no need for X.509 certificates to initiate the TLS handshake because the
relevant key is already known).


Questions
=========

Some things i still havent settled on a opinion:

Is it a good idea to specify the protocol down to the same level as keybase: an
account is just some structured data and enforcing the representation by a DAG
of transactions could be burdensome in some ways (does it scale well with old
accounts?..). Maybe a way out: add a new transaction that is compressing the
history into a flat state at a given point, allowing future transactions to be
checked for sanity either against the full (original) transactions or
equivalently against this one.

Shouldn't validation be not only against a particular transaction but against a
flat-state (ie a transaction and all of it's ancestors)? It could be burdensome
to generate a validation for every claim we checked, but validation of a state
is more coarse-grained (we should be able to exclude some claims we don't
validate)... One simple answer would be to allow validation transaction to
include as data an arbitrary set of (compatible, this is the complication:
where no claim in the set has as ancestor the revocation of a claim of this
set) claims by one entity.


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
.. _CBOR: http://cbor.io/
.. _zooko: https://web.archive.org/web/20120204172516/http://zooko.com/distnames.html
.. _stiegler: http://www.skyhunter.com/marcs/petnames/IntroPetNames.html
