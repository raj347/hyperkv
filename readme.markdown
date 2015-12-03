# hyperkv

p2p key/value store over a [hyperlog][1]
using a [multi-value register conflict strategy][2]

[1]: https://npmjs.com/package/hyperlog
[2]: https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type#Others

# example

## put

``` js
var hyperkv = require('hyperkv')
var hyperlog = require('hyperlog')
var sub = require('subleveldown')

var level = require('level')
var db = level('/tmp/kv.db')

var kv = hyperkv({
  log: hyperlog(sub(db, 'log'), { valueEncoding: 'json' }),
  db: sub(db, 'kv')
})

var key = process.argv[2]
var value = process.argv[3]

kv.put(key, value, function (err, node) {
  if (err) console.error(err)
	else console.log(node.key)
})
```

```
$ node example/put.js greeting hello
1f1b819cd3f7d379914037b473a855b7867f71c76126e379c91cbb31df2a859b
$ node example/put.js greeting 'beep boop'
eadb22a224313d5fb5b811e50915f16491e7714dd32b83503c1e1a1db2bd9e9b
```

## get

``` js
var hyperkv = require('hyperkv')
var hyperlog = require('hyperlog')
var sub = require('subleveldown')

var level = require('level')
var db = level('/tmp/kv.db')

var kv = hyperkv({
  log: hyperlog(sub(db, 'log'), { valueEncoding: 'json' }),
  db: sub(db, 'kv')
})

var key = process.argv[2]
kv.get(key, function (err, value) {
  if (err) console.error(err)
  else console.log(value)
})
```

```
$ node example/get.js greeting
{ eadb22a224313d5fb5b811e50915f16491e7714dd32b83503c1e1a1db2bd9e9b: 'beep boop' } 
```

# api

``` js
var hyperkv = require('hyperkv')
```

## var kv = hyperkv(opts)

* `opts.log` - [hyperlog](https://npmjs.org/package/hyperlog) instance created
with `valueEncoding: 'json'`
* `opts.db` - [level](https://npmjs.com/package/level) instance

## kv.put(key, value, opts={}, cb)

Set `key` to a json `value`.

If `opts.links` is set, refer to previously set keys. Otherwise, the key will
refer to the current "head" key hashes.

`cb(err, node)` fires from the underlying `log.add()` call.

## kv.get(key, cb)

Get the list of current values for `key` as `cb(err, values)` where `values`
maps hyperlog hashes to set values.

## kv.on('put', function (key, value, node) {})

Whenever a node is put, this event fires.

# install

```
npm install hyperkv
```

# license

BSD