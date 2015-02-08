# Ember Pouch

Ember Pouch is a PouchDB/CouchDB adapter for Ember Data.

With Ember Pouch, all of your app's data is automatically saved on the client-side using IndexedDB or WebSQL, and you just keep using the regular [Ember Data `store` API](http://emberjs.com/api/data/classes/DS.Store.html#method_all). This data may be automatically synced to a remote CouchDB (or compatible servers) using PouchDB replication.

What's the point?

1. You don't need to write any server-side logic. Just use CouchDB.

2. Data syncs automatically.

3. Your app works offline, and requests are super fast, because they don't need the network.

For more on PouchDB, check out [pouchdb.com](https://pouchdb.com).

## Install and setup

    bower install ember-pouch --save

In `Brocfile.js`:

```js
app.import('bower_components/pouchdb/dist/pouchdb.js');
app.import('bower_components/relational-pouch/dist/pouchdb.relational-pouch.js');
app.import('bower_components/ember-pouch/dist/globals/main.js');
```

This defines `window.PouchDB` and `window.EmberPouch` globally.

`Ember-Pouch` requires you to add a `rev: DS.attr('string')` field to all your models. This is for PouchDB/CouchDB to handle revisions:

```js
var Todo = DS.Model.extend({
  title       : DS.attr('string'),
  isCompleted : DS.attr('boolean'),
  rev         : DS.attr('string')    // <-- Add this to all your models
});
```

## Configuring /app/adapters/application.js

A local PouchDB that syncs with a remote CouchDB looks like this:

```js
var remote = new PouchDB('http://localhost:5984/my_couch');
var db  = new PouchDB('local_couch');
db.sync(remote);

export default EmberPouch.Adapter.extend({
  db: db
});
```

You can also turn on debugging:

```js
PouchDB.debug.enable('*');
```

And you will probably want to turn on retries in case the user goes offline:

```js
var remote = new PouchDB('http://localhost:5984/my_couch');
var db  = new PouchDB('local_couch');

function doSync() {
  db.sync(remote, {live: true})
    .on('error', function(err) {
      // Retry connection every 5 seconds
      setTimeout(doSync, 5000);
    });
}

doSync();

export default EmberPouch.Adapter.extend({
  db: db
});
```

See the [PouchDB sync API](http://pouchdb.com/api.html#sync) for full usage instructions.

## Sample app

Tom Dale's blog example using Ember CLI and EmberPouch: [broerse/ember-cli-blog](https://github.com/broerse/ember-cli-blog)


## Notes

### LocalStorage

Currently PouchDB doesn't use LocalStorage unless you include an experimental plugin. Amazingly, this is only necessary to support IE ≤ 9.0 and Opera Mini. It's recommended you read more about this, what storage mechanisms modern browsers now support, and using SQLite in Cordova on [the PouchDB adapters page](http://pouchdb.com/adapters.html).

### CouchDB

From day one, CouchDB and its protocol have been designed to be always **A**vailable and handle **P**artitioning over the network well (AP in the CAP theorem). PouchDB/CouchDB gives you a solid way to manage conflicts. It is "eventually consistent," but CouchDB has an API for listening to changes to the database, which can be then pushed down to the client in real-time.

To learn more about how CouchDB sync works, check out [the PouchDB guide to replication](http://pouchdb.com/guides/replication.html).

### Plugins

With PouchDB, you also get access to a whole host of [PouchDB plugins](http://pouchdb.com/external.html).

### Relational Pouch

Ember Pouch is really just a thin layer of Ember-y goodness over [Relational Pouch](https://github.com/nolanlawson/relational-pouch). Before you file an issue, check to see if it's more appropriate to file over there.

### Offline first

If you want to go completely [offline-first](http://offlinefirst.org/), you'll also need an HTML5 appcache.manifest with [broccoli-manifest](https://github.com/racido/broccoli-manifest). This will allow your HTML/CSS/JS assets to load even if the user is offline. Plus your users can "add to homescreen" on a mobile device (iOS/Android).

### Security

An easy way to secure your Ember Pouch-using app is to ensure that data can only be fetched from CouchDB &ndash; not from some other sever (e.g. in an [XSS attack](https://en.wikipedia.org/wiki/Cross-site_scripting)).

To do so, add a Content Security Policy whitelist entry to `/config/environment.js`:

```js
ENV.contentSecurityPolicy = {
  "connect-src": "'self' http://your_couch_host.com:5984"
};
```

Ember CLI includes the [content-security-policy](https://github.com/rwjblue/ember-cli-content-security-policy) plugin by default to ensure that CSP is kept in the forefront of your thoughts. You still have actually to set the CSP HTTP header on your backend in production.

## Build

    $ npm run build

## Credits

This project was originally based on the [ember-data-hal-adapter](https://github.com/locks/ember-data-hal-adapter) by [@locks](https://github.com/locks), and I really benefited from his guidance during its creation.

And of course thanks to all our wonderful contributors, [here](https://github.com/nolanlawson/ember-pouch/graphs/contributors) and [in Relational Pouch](https://github.com/nolanlawson/relational-pouch/graphs/contributors)! 
