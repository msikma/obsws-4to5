# obs-websocket v4 to v5 migration reference

> Note: I only just started writing this guide, so it's far from ready. Feel free to leave a comment/issue/PR to help out.

This is a quick reference I put together to help people migrate their code from [obs-websocket](https://github.com/obsproject/obs-websocket) protocol v4 to v5.

At this time of writing, there unfortunately is no official migration guide, although [one is apparently in the works](https://github.com/obsproject/obs-websocket/wiki/Notable-changes-between-4.x-and-5.x). This means the only way to update your code right now is to go through both protocol documents and guess which old calls correspond to which new calls.

I am not involved in the development of obs-websocket or OBS itself, so I'm not the best person for writing a migration guide, but I thought it would be useful to at least share my work with others. This document aims to at least link together all v4 calls and v5 calls, so that if you're migrating you can Ctrl+F your old v4 calls and instantly find out what the new call should be.

**This guide is written with [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js) in mind,** so some of the information here is JS-specific—if you use a different client, you can skip over to the list of calls. If you'd like to help improve this guide by adding information specific to other clients, or anything else that might help, feel free to open an issue or send in a PR.

## New features

* You can now store and request **persistent JSON data** in OBS using the [GetPersistentData](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md#getpersistentdata) call.
* A new **"vendor" API** has been added, which is similar to `CustomEvent` messages but designed specifically for plugin developers.

A number of [other new features](https://github.com/obsproject/obs-websocket/wiki/Notable-changes-between-4.x-and-5.x) have been added as well.

## Basics

The protocol has completely changed since v4, so old applications will need to be totally changed. A v4 client can not connect to a v5 server—there is no backwards compatibility.

* The **default port** was changed from **4444** (v4) to **4455** (v5).
* Authentication is now always enabled by default.

*The following examples are specific to [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js)—if you're using a different client, check their documentation to see its equivalent.*

### Connecting

<table>
<tr>
<th>Protocol v4</th><th>Protocol v5</th>
</tr>
<tr>
<td valign="top">

```js
await client.connect({
  address: 'localhost:4444',
  password: 'my-password'
})
```
</td>
<td valign="top">

```js
await client.connect(
  'ws://localhost:4455',
  'my-password',
  // identificationParams {}
)
```
</td>
</tr>
<tr>
<td valign="top" colspan="2">

• The `ws://` part is now mandatory.<br>
• A new [identificationParams](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md#identify-opcode-1) object can be passed ([example](https://github.com/obs-websocket-community-projects/obs-websocket-js#connecting)).

</td>
</tr>
</table>

If you try to connect to a **v5** server with a **v4** client, the call will throw a `CONNECTION_ERROR`.

Disconnecting has remained the same.

### Sending requests

The method for sending requests has changed from `.send()` to `.call()`.

<table>
<tr>
<th>Protocol v4</th><th>Protocol v5</th>
</tr>
<tr>
<td valign="top">

```js
await client.send('RequestName', {})
```
</td>
<td valign="top">

```js
await client.call('RequestName', {})
```
</td>
</tr>
</table>

An important change is that **v5** has a new batch request API. In **v4**, batch requests could be sent using the `ExecuteBatch` call, whereas they're now a completely different type of request.

Since some calls in **v5** now carry less information than before, batch requests can be used to supplement additional data. For example, the **v4** `GetSceneList` call includes a list of scene sources for every scene, whereas the **v5** `GetSceneList` does not and requires a `GetSceneItemList` call for every scene that you want sources for. In these cases the easiest way to rewrite legacy code is to just do a batch call and then restructure the data.

<table>
<tr>
<th>Protocol v4</th><th>Protocol v5</th>
</tr>
<tr>
<td valign="top">

```js
const results = await client.send(
  'ExecuteBatch',
  {
    requests: [
      {
        'request-type': 'GetVersion'
      },
      {
        'request-type': 'SetPreviewScene',
        'scene-name': 'Scene 5'
      }
    ]
  }
)
```
</td>
<td valign="top">

```js
const results = await client.callBatch([
  {
    requestType: 'GetVersion',
  },
  {
    requestType: 'SetCurrentPreviewScene',
    requestData: {sceneName: 'Scene 5'}
  }
])
```
</td>
</tr>
<tr>
<td valign="top" colspan="2">

• The request data is now split off into its own object.

</td>
</tr>
</table>

### Listening for events

Almost all events have changed name and return data in a different format. The process of adding listeners has remained the same.

It's worth mentioning that all event properties are now in camelCase, whereas in **v4** properties were a mix of camelCase and snake-case.

<table>
<tr>
<th>Protocol v4</th><th>Protocol v5</th>
</tr>
<tr>
<td valign="top">

```js
// Adds a listener for EventName.
client.on('EventName', data => {
  // Do something with 'data'.
})
// TODO
```
</td>
<td valign="top">

```js
// Adds a listener for EventName.
client.on('EventName', data => {
  // Do something with 'data'.
})
client.addListener('EventName', fn)

// Removes a specific EventName listener.
client.off('EventName', fn)
client.removeListener('EventName', fn)

// Runs a callback only once.
client.once('EventName', fn)
```
</td>
</tr>
<tr>
<td valign="top" colspan="2">

• Data attributes are now always in camelCase.

</td>
</tr>
</table>

## Requests and events

Almost all requests and events have changed between v4 and v5—the names are different and the data formats have changed. The full list of changes is so long that it's split off into its own document.

For a full list of the changes, see [calls.md](calls.md).



## External links

* [obs-websocket 4.9.1 Protocol](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md) ([JSON](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/comments.json), [requests](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#requests), [events](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#events))
* [obs-websocket 5.1.0 Protocol](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md) ([JSON](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.json), [requests](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#requests), [events](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#events), [design goals](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md#design-goals))
* [obs-websocket Wiki: Notable Changes between 4.x and 5.x](https://github.com/obsproject/obs-websocket/wiki/Notable-changes-between-4.x-and-5.x)
* [obs-websocket-js 5.0.0 Release: Breaking Changes](https://github.com/obs-websocket-community-projects/obs-websocket-js/releases/tag/v5.0.0)
* [obs-websocket](https://github.com/obsproject/obs-websocket) ([v4](https://github.com/obsproject/obs-websocket/tree/8823ecd2094ffa17dac6ceaa2987981b12f0820a), [v5](https://github.com/obsproject/obs-websocket/tree/6fd18a7ef1ecb149e8444154af1daab61d4241a9))
* [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js) ([v4](https://github.com/obs-websocket-community-projects/obs-websocket-js/tree/v4), [v5](https://github.com/obs-websocket-community-projects/obs-websocket-js/tree/d6244bb26c473ced366d710d091f4a51c5d76fc2))
