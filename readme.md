# obs-websocket v4 to v5 migration reference

This is a quick reference I put together to help people migrate their code from [obs-websocket](https://github.com/obsproject/obs-websocket) protocol v4 to v5.

At this time of writing, there unfortunately is no official migration guide, although [one is apparently in the works](https://github.com/obsproject/obs-websocket/wiki/Notable-changes-between-4.x-and-5.x). This means the only way to update your code right now is to go through both protocol documents and guess which old calls correspond to which new calls.

**I am not involved in the development of obs-websocket or OBS itself,** so I'm not the best person for writing a migration guide, but I thought it would be useful to at least share my work with others. This document primarily aims to connect all v4 requests and events to their v5 equivalents, so that if you're migrating you can Ctrl+F your old v4 calls and instantly find out what the new call should be.

While most information here is client-agnostic, some examples are written with [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js) in mind, just for the same of having practical examples. If you use a different client, you can skip over to the list of calls. If you'd like to help improve this guide by adding information specific to other clients, or anything else that might help, feel free to open an issue or send in a PR.

## Migration document

For ease of reading, the actual reference guide is its own separate document.

Read the [obs-websocket v4 to v5 migration reference](migration-guide.md) on Github.

## External links

* [obs-websocket 4.9.1 Protocol](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md) ([JSON](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/comments.json), [requests](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#requests), [events](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#events))
* [obs-websocket 5.1.0 Protocol](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md) ([JSON](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.json), [requests](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#requests), [events](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#events), [design goals](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md#design-goals))
* [obs-websocket Wiki: Notable Changes between 4.x and 5.x](https://github.com/obsproject/obs-websocket/wiki/Notable-changes-between-4.x-and-5.x)
* [obs-websocket-js 5.0.0 Release: Breaking Changes](https://github.com/obs-websocket-community-projects/obs-websocket-js/releases/tag/v5.0.0)
* [obs-websocket](https://github.com/obsproject/obs-websocket) ([v4](https://github.com/obsproject/obs-websocket/tree/8823ecd2094ffa17dac6ceaa2987981b12f0820a), [v5](https://github.com/obsproject/obs-websocket/tree/6fd18a7ef1ecb149e8444154af1daab61d4241a9))
* [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js) ([v4](https://github.com/obs-websocket-community-projects/obs-websocket-js/tree/v4), [v5](https://github.com/obs-websocket-community-projects/obs-websocket-js/tree/d6244bb26c473ced366d710d091f4a51c5d76fc2))
