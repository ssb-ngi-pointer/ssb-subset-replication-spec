# Secure partial replication for SSB

Status: Design phase

This module comes from the desire to synchronize a subset of a SSB
log. The main motivation is to enable faster initial sync by only
replicating a certain subset of messages that can then be supplimented
with a frontier sync such as EBT.

It is recommended to use this module in conjunction with
[ssb-secure-partial-feed].

This module consists of two methods:

## getSubset(query): source

The query parameter is interface similar to [JITDB] except the data
array should be unbounded. To support pagination, `offset`, `limit`
and `reverse` can be supplied. To get the latest 10 post messages of a
particular user the following query can be used:

```
options: { limit 10, reverse: true},
query: {
  type: 'AND',
  data: [
    { type: 'EQUAL', data: { type: 'type', value: 'post' } },
    { type: 'EQUAL', data: { type: 'author', value: '@6CAxOI3f+LUOVrbAl0IemqiS7ATpQvr9Mdw9LC4+Uv0=.ed25519' } }
  ]
}
```

Which will result in the following:

```
{
   proofs: [proof1, ...],
   data: [msg1, ...]
}
```

Where a proof is defined as the latest version of a [ssb-observable]
that describes the thing you are querying. This can be useful for
contact messages for an author where messages needs to be in order and
nothing left out by a third party. Note you can have multiple
proofs. This allows other feeds than the author to assign proofs to a
result set.

This api replaces `getFeed`, `getFeedReverse`, `getMessagesOfType` of
[partial replication v1]. This leaves `getTangle`.

## getTangle(rootId): source

Source is a stream of messages that refers to the rootId. `getTangle`
was introduced as a way to get older messages in an easy way. Say
someone mentions an old thread that is not part of your frontier of
messages. `getTangle` is a special case of [set-reconciliation] where
you don't have any messages in the set. Because of this the protocol
can be a lot simplier by just specifying the root you are interested
in. New messages in the tangle will be get replicated using the
frontier synchonization the same way as say new contact messages.

[JITDB]: https://github.com/arj03/jitdb
[ssb-observable]: https://github.com/arj03/ssb-observables
[partial replication v1]: https://github.com/arj03/ssb-partial-replication
[set-reconciliation]: https://github.com/AljoschaMeyer/set-reconciliation
[ssb-secure-partial-feed]: https://github.com/arj03/ssb-secure-partial-feed
[EBT]: https://github.com/ssbc/epidemic-broadcast-trees/
