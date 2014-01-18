node-tomahawkjs
==========================

Implementation of the JS plugins API from Tomahawk for NodeJS

(This implementation is not fully feature complete compared to the desktop client but [hubot-tomahk](https://github.com/xhochy/hubot-tomahk) is an exmple usage of parts of it)

Usage
-----

### Create a resolver instance

To load a single resolver, we simply use the following code if we know the path to the .axe file:

```javascript
var TomahawkJS = require('tomahawkjs');

TomahawkJS.loadAxe(pathtoaxe, function(err, axe) {
  // TODO: Check for error in err
  // After load the axe, we most likely want to have an instance of the resolver
  axe.getInstace(function(err, instance_context) {
    // TODO: Check for error in err
    var instance = instance_context.instance;
    // Each Resolver instance runs in its own global JavaScript context
    var context = instance_context.context;
    // Start the instance
    // Due to the architecture of the resolveres, this may do some background work and will not block.
    instance.init();
    // We can now use the instance
  });
});
```

### Checking for a capability

As the time of writing, Tomahawk resolvers were all implicitly able to resolve from a combination of Artist and Title (and Albumname) to a stream URL if their feeding service provided this tune. Furthermore there are some other capabilities that can be implemented:

* `browsable`: Resolver exposes a music collection that can be browsed.
* `playlistsync`: Can sync playlists
* `urllookup`: Can resolve from an URL to a combination of Artist+Title/Album/Playlist

Checking for one of these can be done via:

```javascript
context.hasCapability('urllookup')
```

### URL Lookup

As an inverse for the lookup of (Artist, Title) -> stream-URL, many resolvers have the `urllookup` capability to resolve from a URL pointing to a music service to artist/album/playlist/.. that is encoded in the URL.

```javascript
// Define a handler for found URL mappings
context.on('url-result', function (url, result) {
  if (result.type == 'track') {
    console.log('Found URL for track ' + result.artist + ' - ' + result.title);
  } else if (result.type == 'album') {
    console.log('Found URL for album ' + result.artist + ' - ' + result.name);
  } else if (result.type == 'playlist') {
    console.log('Found URL for playlist ' + result.title);
  } else if (result.type == 'artist') {
    console.log('Found URL for artist ' + result.name);
  }
});

// Check if resolver has URLLookup capability
if (context.hasCapability('urllookup')) {
  // Can we handle this URL?
  if (instance.canParseUrl(url)) {
    instance.lookupUrl(url);
  }
}
```
