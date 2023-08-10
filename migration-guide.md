# obs-websocket v4 to v5 migration reference

**This document is a migration reference for developers interested in updating their code from obs-websocket v4 to v5.**

While this is not a comprehensive reference, it aims to get you most of the way there. By far most upgrade work is in the request/event changes—almost every single call to the server has changed name and interface. The main purpose of this document is to allow you to search for your old v4 calls to easily find the v5 equivalent. That way you can update your code one call at a time.

Additionally, this document contains a brief overview of the changes to how you connect to the server and make calls, although this is mostly client-dependent. We use [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js) for the sake of example—if you're using a different client, check their documentation to see how these aspects changed.

See [this document's Github page](readme.md) for information on how to contribute.

## New features

* You can now store and request **persistent JSON data** in OBS using the [GetPersistentData](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md#getpersistentdata) call.
* A new **"vendor" API** has been added, which is similar to [CustomEvent](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#customevent) messages but designed specifically for plugin developers.
* The API is now **versioned**, meaning calls are guaranteed to be stable if the RPC version is specified on connect.

A number of [other new features](https://github.com/obsproject/obs-websocket/wiki/Notable-changes-between-4.x-and-5.x) have been added as well.

## Basics

**The protocol has completely changed since v4,** so old applications require extensive changes to bring them up to date. A v4 client can *not* connect to a v5 server—there is no backwards compatibility.

In the OBS settings panel, the following changes were made:

* The **default port** was changed from **4444** (v4) to **4455** (v5).
* Authentication is now enabled by default.

All basic aspects of connecting to obs-websocket, making calls and listening for events, is client-specific. For illustrative purposes, we're demonstrating the changes using the [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js) client. All code in this section is specific to Javascript for the sake of example.

If you're using a different client, check its documentation to see its equivalent.

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

It's **highly recommended** to add the RPC version to your `connect()` call to ensure stability in the API.

If you try to connect to a **v5** server with a **v4** client, the call will throw a `CONNECTION_ERROR`.

Disconnecting has remained the same.

### Sending requests

The method for sending requests has changed from `send()` to `call()`.

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

An important change is that **v5** has a new batch request API. In **v4**, batch requests could be sent using the **ExecuteBatch** call, whereas they're now a completely different type of request.

Since some calls in **v5** now carry less information than before, batch requests can be used to supplement additional data. For example, the **v4** **GetSceneList** call includes a list of scene sources for every scene, whereas the **v5** **GetSceneList** call does not and requires a **GetSceneItemList** call for every scene that you want sources for. In these cases the easiest way to recreate the legacy behavior is to just do a batch call and then combine the data.

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

Almost all events have changed name and return data in a different format.

It's worth mentioning that all event properties are now in camelCase, whereas in **v4** properties were a mix of camelCase and snake-case.

The process of adding and removing listeners **has remained the same**, so this overview here is purely for reference.

<table>
<tr>
<th colspan="2">Protocol v4 and v5</th>
</tr>
<tr>
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

## Request/event reference

This section contains a list of all **requests** and **events** that changed between obs-websocket v4 and v5.

Almost everything has changed between the two versions, and almost nothing is backwards compatible. The best way to upgrade your code is to go through each event individually and change it to the v5 equivalent.

For more information, see the full official documentation on requests and events:

* [4.9.1 protocol request list](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#requests) and [event list](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#events)
* [5.1.0 protocol request list](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#requests) and [event list](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#events)

All calls in this document have links to the relevant heading in the official documentation.

### Requests

**General:**

* <a href="#getversion">GetVersion</a>
* <a href="#getauthrequired">GetAuthRequired</a>
* <a href="#authenticate">Authenticate</a>
* <a href="#setheartbeat">SetHeartbeat</a>
* <a href="#setfilenameformatting">SetFilenameFormatting</a>
* <a href="#getfilenameformatting">GetFilenameFormatting</a>
* <a href="#getstats">GetStats</a>
* <a href="#broadcastcustommessage">BroadcastCustomMessage</a>
* <a href="#getvideoinfo">GetVideoInfo</a>
* <a href="#openprojector">OpenProjector</a>
* <a href="#triggerhotkeybyname">TriggerHotkeyByName</a>
* <a href="#triggerhotkeybysequence">TriggerHotkeyBySequence</a>
* <a href="#executebatch">ExecuteBatch</a>
* <a href="#sleep">Sleep</a>

**Media Control:**

* <a href="#playpausemedia">PlayPauseMedia</a>, <a href="#restartmedia">RestartMedia</a>, <a href="#stopmedia">StopMedia</a>, <a href="#nextmedia">NextMedia</a>, <a href="#previousmedia">PreviousMedia</a>
* <a href="#setmediatime">SetMediaTime</a>
* <a href="#getmediaduration">GetMediaDuration</a>, <a href="#getmediatime">GetMediaTime</a>, <a href="#getmediastate">GetMediaState</a>
* <a href="#scrubmedia">ScrubMedia</a>

**Sources:**

* <a href="#getmediasourceslist">GetMediaSourcesList</a>
* <a href="#createsource">CreateSource</a>
* <a href="#getsourceslist">GetSourcesList</a>
* <a href="#getsourcetypeslist">GetSourceTypesList</a>
* <a href="#getvolume">GetVolume</a>
* <a href="#setvolume">SetVolume</a>
* <a href="#getaudiotracks">GetAudioTracks</a>
* <a href="#setaudiotracks">SetAudioTracks</a>
* <a href="#getmute">GetMute</a>
* <a href="#setmute">SetMute</a>
* <a href="#togglemute">ToggleMute</a>
* <a href="#getsourceactive">GetSourceActive</a>
* <a href="#getaudioactive">GetAudioActive</a>
* <a href="#setsourcename">SetSourceName</a>
* <a href="#setsyncoffset">SetSyncOffset</a>
* <a href="#getsyncoffset">GetSyncOffset</a>
* <a href="#getsourcesettings">GetSourceSettings</a>, <a href="#settextgdiplusproperties">SetTextGDIPlusProperties</a>, <a href="#settextfreetype2properties">SetTextFreetype2Properties</a>, <a href="#getbrowsersourceproperties">GetBrowserSourceProperties</a>
* <a href="#setsourcesettings">SetSourceSettings</a>, <a href="#gettextgdiplusproperties">GetTextGDIPlusProperties</a>, <a href="#gettextfreetype2properties">GetTextFreetype2Properties</a>, <a href="#setbrowsersourceproperties">SetBrowserSourceProperties</a>
* <a href="#getspecialsources">GetSpecialSources</a>
* <a href="#getsourcefilters">GetSourceFilters</a>
* <a href="#getsourcefilterinfo">GetSourceFilterInfo</a>
* <a href="#addfiltertosource">AddFilterToSource</a>
* <a href="#removefilterfromsource">RemoveFilterFromSource</a>
* <a href="#reordersourcefilter">ReorderSourceFilter</a>
* <a href="#movesourcefilter">MoveSourceFilter</a>
* <a href="#setsourcefiltersettings">SetSourceFilterSettings</a>
* <a href="#setsourcefiltervisibility">SetSourceFilterVisibility</a>
* <a href="#getaudiomonitortype">GetAudioMonitorType</a>
* <a href="#setaudiomonitortype">SetAudioMonitorType</a>
* <a href="#getsourcedefaultsettings">GetSourceDefaultSettings</a>
* <a href="#takesourcescreenshot">TakeSourceScreenshot</a>
* <a href="#refreshbrowsersource">RefreshBrowserSource</a>

**Outputs:**

* <a href="#listoutputs">ListOutputs</a>
* <a href="#getoutputinfo">GetOutputInfo</a>
* <a href="#startoutput">StartOutput</a>
* <a href="#stopoutput">StopOutput</a>

**Profiles:**

* <a href="#setcurrentprofile">SetCurrentProfile</a>
* <a href="#getcurrentprofile">GetCurrentProfile</a>
* <a href="#listprofiles">ListProfiles</a>

**Recording:**

* <a href="#getrecordingstatus">GetRecordingStatus</a>
* <a href="#startstoprecording">StartStopRecording</a>
* <a href="#startrecording">StartRecording</a>
* <a href="#stoprecording">StopRecording</a>
* <a href="#pauserecording">PauseRecording</a>
* <a href="#resumerecording">ResumeRecording</a>
* <a href="#setrecordingfolder">SetRecordingFolder</a>
* <a href="#getrecordingfolder">GetRecordingFolder</a>

**Replay Buffer:**

* <a href="#getreplaybufferstatus">GetReplayBufferStatus</a>
* <a href="#startstopreplaybuffer">StartStopReplayBuffer</a>
* <a href="#startreplaybuffer">StartReplayBuffer</a>
* <a href="#stopreplaybuffer">StopReplayBuffer</a>
* <a href="#savereplaybuffer">SaveReplayBuffer</a>

**Scene Collections:**

* <a href="#setcurrentscenecollection">SetCurrentSceneCollection</a>
* <a href="#getcurrentscenecollection">GetCurrentSceneCollection</a>
* <a href="#listscenecollections">ListSceneCollections</a>

**Scene Items:**

* <a href="#getsceneitemlist">GetSceneItemList</a>
* <a href="#getsceneitemproperties">GetSceneItemProperties</a>
* <a href="#setsceneitemproperties">SetSceneItemProperties</a>
* <a href="#resetsceneitem">ResetSceneItem</a>
* <a href="#setsceneitemrender">SetSceneItemRender</a>
* <a href="#setsceneitemposition">SetSceneItemPosition</a>
* <a href="#setsceneitemtransform">SetSceneItemTransform</a>
* <a href="#setsceneitemcrop">SetSceneItemCrop</a>
* <a href="#deletesceneitem">DeleteSceneItem</a>
* <a href="#addsceneitem">AddSceneItem</a>
* <a href="#duplicatesceneitem">DuplicateSceneItem</a>

**Scenes:**

* <a href="#setcurrentscene">SetCurrentScene</a>
* <a href="#getcurrentscene">GetCurrentScene</a>
* <a href="#getscenelist">GetSceneList</a>
* <a href="#createscene">CreateScene</a>
* <a href="#reordersceneitems">ReorderSceneItems</a>
* <a href="#setscenetransitionoverride">SetSceneTransitionOverride</a>
* <a href="#removescenetransitionoverride">RemoveSceneTransitionOverride</a>
* <a href="#getscenetransitionoverride">GetSceneTransitionOverride</a>
* <a href="#getstreamingstatus">GetStreamingStatus</a>
* <a href="#startstopstreaming">StartStopStreaming</a>
* <a href="#startstreaming">StartStreaming</a>
* <a href="#stopstreaming">StopStreaming</a>
* <a href="#setstreamsettings">SetStreamSettings</a>
* <a href="#getstreamsettings">GetStreamSettings</a>
* <a href="#savestreamsettings">SaveStreamSettings</a>
* <a href="#sendcaptions">SendCaptions</a>

**Studio Mode:**

* <a href="#getstudiomodestatus">GetStudioModeStatus</a>
* <a href="#getpreviewscene">GetPreviewScene</a>
* <a href="#setpreviewscene">SetPreviewScene</a>
* <a href="#transitiontoprogram">TransitionToProgram</a>
* <a href="#enablestudiomode">EnableStudioMode</a>
* <a href="#disablestudiomode">DisableStudioMode</a>
* <a href="#togglestudiomode">ToggleStudioMode</a>

**Transitions:**

* <a href="#gettransitionlist">GetTransitionList</a>
* <a href="#getcurrenttransition">GetCurrentTransition</a>
* <a href="#setcurrenttransition">SetCurrentTransition</a>
* <a href="#settransitionduration">SetTransitionDuration</a>
* <a href="#gettransitionduration">GetTransitionDuration</a>
* <a href="#gettransitionposition">GetTransitionPosition</a>
* <a href="#gettransitionsettings">GetTransitionSettings</a>
* <a href="#settransitionsettings">SetTransitionSettings</a>
* <a href="#releasetbar">ReleaseTBar</a>
* <a href="#settbarposition">SetTBarPosition</a>

**Virtual Cam:**

* <a href="#getvirtualcamstatus">GetVirtualCamStatus</a>
* <a href="#startstopvirtualcam">StartStopVirtualCam</a>
* <a href="#startvirtualcam">StartVirtualCam</a>
* <a href="#stopvirtualcam">StopVirtualCam</a>

### Events

**Scenes:**

* <a href="#switchscenes">SwitchScenes</a>
* <a href="#sceneschanged">ScenesChanged</a>
* <a href="#scenecollectionchanged">SceneCollectionChanged</a>
* <a href="#scenecollectionlistchanged">SceneCollectionListChanged</a>

**Transitions:**

* <a href="#switchtransition">SwitchTransition</a>
* <a href="#transitionlistchanged">TransitionListChanged</a>
* <a href="#transitiondurationchanged">TransitionDurationChanged</a>
* <a href="#transitionbegin">TransitionBegin</a>
* <a href="#transitionend">TransitionEnd</a>
* <a href="#transitionvideoend">TransitionVideoEnd</a>

**Profiles:**

* <a href="#profilechanged">ProfileChanged</a>
* <a href="#profilelistchanged">ProfileListChanged</a>

**Streaming:**

* <a href="#streamstarting">StreamStarting</a>, <a href="#streamstarted">StreamStarted</a>, <a href="#streamstopping">StreamStopping</a>, <a href="#streamstopped">StreamStopped</a>
* <a href="#streamstatus">StreamStatus</a>

**Recording:**

* <a href="#recordingstarting">RecordingStarting</a>, <a href="#recordingstarted">RecordingStarted</a>, <a href="#recordingstopping">RecordingStopping</a>, <a href="#recordingstopped">RecordingStopped</a>, <a href="#recordingpaused">RecordingPaused</a>, <a href="#recordingresumed">RecordingResumed</a>

**Virtual Cam:**

* <a href="#virtualcamstarted">VirtualCamStarted</a>, <a href="#virtualcamstopped">VirtualCamStopped</a>

**Replay Buffer:**

* <a href="#replaystarting">ReplayStarting</a>, <a href="#replaystarted">ReplayStarted</a>, <a href="#replaystopping">ReplayStopping</a>, <a href="#replaystopped">ReplayStopped</a>

**Other:**

* <a href="#exiting">Exiting</a>

**General:**

* <a href="#heartbeat">Heartbeat</a>
* <a href="#broadcastcustommessage">BroadcastCustomMessage</a>

**Sources:**

* <a href="#sourcecreated">SourceCreated</a>
* <a href="#sourcedestroyed">SourceDestroyed</a>
* <a href="#sourcevolumechanged">SourceVolumeChanged</a>
* <a href="#sourcemutestatechanged">SourceMuteStateChanged</a>
* <a href="#sourceaudiodeactivated">SourceAudioDeactivated</a>, <a href="#sourceaudioactivated">SourceAudioActivated</a>
* <a href="#sourceaudiosyncoffsetchanged">SourceAudioSyncOffsetChanged</a>
* <a href="#sourceaudiomixerschanged">SourceAudioMixersChanged</a>
* <a href="#sourcerenamed">SourceRenamed</a>
* <a href="#sourcefilteradded">SourceFilterAdded</a>
* <a href="#sourcefilterremoved">SourceFilterRemoved</a>
* <a href="#sourcefiltervisibilitychanged">SourceFilterVisibilityChanged</a>
* <a href="#sourcefiltersreordered">SourceFiltersReordered</a>

**Media:**

* <a href="#mediastarted">MediaStarted</a>
* <a href="#mediaended">MediaEnded</a>
* <a href="#mediaplaying">MediaPlaying</a>, <a href="#mediapaused">MediaPaused</a>, <a href="#mediarestarted">MediaRestarted</a>, <a href="#mediastopped">MediaStopped</a>, <a href="#medianext">MediaNext</a>, <a href="#mediaprevious">MediaPrevious</a>

**Scene Items:**

* <a href="#sourceorderchanged">SourceOrderChanged</a>
* <a href="#sceneitemadded">SceneItemAdded</a>
* <a href="#sceneitemremoved">SceneItemRemoved</a>
* <a href="#sceneitemvisibilitychanged">SceneItemVisibilityChanged</a>
* <a href="#sceneitemlockchanged">SceneItemLockChanged</a>
* <a href="#sceneitemtransformchanged">SceneItemTransformChanged</a>
* <a href="#sceneitemselected">SceneItemSelected</a>
* <a href="#sceneitemdeselected">SceneItemDeselected</a>

**Studio Mode:**

* <a href="#previewscenechanged">PreviewSceneChanged</a>
* <a href="#studiomodeswitched">StudioModeSwitched</a>

## Requests

### GetVersion

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getversion">GetVersion</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getversion">GetVersion</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>version</td>
<td>—</td>
</tr>
<tr>
<td>obs-websocket-version</td>
<td>obsWebSocketVersion</td>
</tr>
<tr>
<td>obs-studio-version</td>
<td>obsVersion</td>
</tr>
<tr>
<td>—</td>
<td>rpcVersion</td>
</tr>
<tr>
<td>available-requests</td>
<td>availableRequests</td>
</tr>
<tr>
<td>supported-image-export-formats</td>
<td>supportedImageFormats</td>
</tr>
<tr>
<td>—</td>
<td>platform</td>
</tr>
<tr>
<td>—</td>
<td>platformDescription</td>
</tr>
</table>

**Notes:**

* **available-requests** and **supported-image-export-formats** changed from a comma-separated list string into an array of strings.

----

### GetAuthRequired

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getauthrequired">GetAuthRequired</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Authentication is no longer done through regular calls, but through the [`Identify` OpCode](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#identify-opcode-1) sent while initiating the connection. See your client's documentation for more details.

----

### Authenticate

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#authenticate">Authenticate</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Authentication is no longer done through regular calls, but through the [`Identify` OpCode](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#identify-opcode-1) sent while initiating the connection. See your client's documentation for more details.

----

### SetHeartbeat

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setheartbeat">SetHeartbeat</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Was deprecated since 4.9.0.

----

### SetFilenameFormatting

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setfilenameformatting">SetFilenameFormatting</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### GetFilenameFormatting

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getfilenameformatting">GetFilenameFormatting</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### GetStats

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getstats">GetStats</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getstats">GetStats</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>fps</td>
<td>activeFps</td>
</tr>
<tr>
<td>render-total-frames</td>
<td>renderTotalFrames</td>
</tr>
<tr>
<td>render-missed-frames</td>
<td>renderSkippedFrames</td>
</tr>
<tr>
<td>output-total-frames</td>
<td>outputTotalFrames</td>
</tr>
<tr>
<td>output-skipped-frames</td>
<td>outputSkippedFrames</td>
</tr>
<tr>
<td>average-frame-time</td>
<td>averageFrameRenderTime</td>
</tr>
<tr>
<td>cpu-usage</td>
<td>cpuUsage</td>
</tr>
<tr>
<td>memory-usage</td>
<td>memoryUsage</td>
</tr>
<tr>
<td>free-disk-space</td>
<td>availableDiskSpace</td>
</tr>
<tr>
<td>—</td>
<td>webSocketSessionIncomingMessages</td>
</tr>
<tr>
<td>—</td>
<td>webSocketSessionOutgoingMessages</td>
</tr>
</table>

----

### BroadcastCustomMessage

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#broadcastcustommessage">BroadcastCustomMessage</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#broadcastcustomevent">BroadcastCustomEvent</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>realm</td>
<td>—</td>
</tr>
<tr>
<td>data</td>
<td>eventData</td>
</tr>
</table>

**Response fields:**

*None.*

**Notes:**

* **realm** is gone. Pass on the realm value in your eventData object instead.
* Note that the related v5 **CallVendorRequest** call does have a **requestType** value that mirrors **realm**.

----

### GetVideoInfo

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getvideoinfo">GetVideoInfo</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getvideosettings">GetVideoSettings</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>baseWidth</td>
<td>baseWidth</td>
</tr>
<tr>
<td>baseHeight</td>
<td>baseHeight</td>
</tr>
<tr>
<td>outputWidth</td>
<td>outputWidth</td>
</tr>
<tr>
<td>outputHeight</td>
<td>outputHeight</td>
</tr>
<tr>
<td>scaleType</td>
<td>—</td>
</tr>
<tr>
<td>fps</td>
<td>fpsNumerator</td>
</tr>
<tr>
<td>—</td>
<td>fpsDenominator</td>
</tr>
<tr>
<td>videoFormat</td>
<td>—</td>
</tr>
<tr>
<td>colorSpace</td>
<td>—</td>
</tr>
<tr>
<td>colorRange</td>
<td>—</td>
</tr>
</table>

**Notes:**

* To get the true FPS value, divide the FPS numerator by the FPS denominator.+sa: SetVideoSettings

----

### OpenProjector

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#openprojector">OpenProjector</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#opensourceprojector">OpenSourceProjector</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>type</td>
<td>—</td>
</tr>
<tr>
<td>monitor</td>
<td>monitorIndex</td>
</tr>
<tr>
<td>geometry</td>
<td>projectorGeometry</td>
</tr>
<tr>
<td>name</td>
<td>sourceName</td>
</tr>
</table>

**Response fields:**

*None.*

**Notes:**

* Docs state this is likely to be changed or deprecated in the future.

----

### TriggerHotkeyByName

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#triggerhotkeybyname">TriggerHotkeyByName</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#triggerhotkeybyname">TriggerHotkeyByName</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>hotkeyName</td>
<td>hotkeyName</td>
</tr>
</table>

**Response fields:**

*None.*

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#gethotkeylist">GetHotkeyList</a>

----

### TriggerHotkeyBySequence

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#triggerhotkeybysequence">TriggerHotkeyBySequence</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#triggerhotkeybykeysequence">TriggerHotkeyByKeySequence</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>keyId</td>
<td>keyId</td>
</tr>
<tr>
<td>keyModifiers</td>
<td>keyModifiers</td>
</tr>
</table>

**Response fields:**

*None.*

----

### ExecuteBatch

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#executebatch">ExecuteBatch</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* This is no longer a single request, but a separate API entirely. See [readme.md](readme.md#sending-requests) for more information.

----

### Sleep

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sleep">Sleep</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sleep">Sleep</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sleepMillis</td>
<td>sleepMillis</td>
</tr>
<tr>
<td>—</td>
<td>sleepFrames</td>
</tr>
</table>

**Response fields:**

*None.*

----

### PlayPauseMedia, RestartMedia, StopMedia, NextMedia, PreviousMedia

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#playpausemedia">PlayPauseMedia</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#restartmedia">RestartMedia</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#stopmedia">StopMedia</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#nextmedia">NextMedia</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#previousmedia">PreviousMedia</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#triggermediainputaction">TriggerMediaInputAction</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>—</td>
<td>mediaAction</td>
</tr>
</table>

**Response fields:**

*None.*

**Notes:**

* **mediaAction** must be an item from [the ObsMediaInputAction enum](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#obsmediainputaction).

----

### SetMediaTime

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setmediatime">SetMediaTime</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setmediainputcursor">SetMediaInputCursor</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>timestamp</td>
<td>mediaCursor</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetMediaDuration, GetMediaTime, GetMediaState

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getmediaduration">GetMediaDuration</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getmediatime">GetMediaTime</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getmediastate">GetMediaState</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getmediainputstatus">GetMediaInputStatus</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>mediaState</td>
</tr>
<tr>
<td>mediaDuration</td>
<td>mediaDuration</td>
</tr>
<tr>
<td>timestamp</td>
<td>mediaCursor</td>
</tr>
</table>

**Notes:**

* **mediaState** will be an item from [the ObsMediaState enum](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#media-inputs-requests).

----

### ScrubMedia

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#scrubmedia">ScrubMedia</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#offsetmediainputcursor">OffsetMediaInputCursor</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>timeOffset</td>
<td>mediaCursorOffset</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetMediaSourcesList

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getmediasourceslist">GetMediaSourcesList</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### CreateSource

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#createsource">CreateSource</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#createinput">CreateInput</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>sourceKind</td>
<td>inputKind</td>
</tr>
<tr>
<td>sceneName</td>
<td>sceneName</td>
</tr>
<tr>
<td>sourceSettings</td>
<td>inputSettings</td>
</tr>
<tr>
<td>setVisible</td>
<td>sceneItemEnabled</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>itemId</td>
<td>sceneItemId</td>
</tr>
</table>

----

### GetSourcesList

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsourceslist">GetSourcesList</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputlist">GetInputList</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>inputKind</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sources[].name</td>
<td>inputs[].inputName</td>
</tr>
<tr>
<td>sources[].typeId</td>
<td>—</td>
</tr>
<tr>
<td>sources[].type</td>
<td>inputs[].inputKind</td>
</tr>
<tr>
<td>—</td>
<td>inputs[].unversionedInputKind</td>
</tr>
</table>

----

### GetSourceTypesList

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsourcetypeslist">GetSourceTypesList</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputkindlist">GetInputKindList</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>unversioned</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>types</td>
<td>inputKinds</td>
</tr>
</table>

**Notes:**

* In v4 an object with lots of metadata was returned. In v5 only an array of strings representing the internal input kind names is returned.

----

### GetVolume

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getvolume">GetVolume</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputvolume">GetInputVolume</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>source</td>
<td>inputName</td>
</tr>
<tr>
<td>useDecibel</td>
<td>—</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>—</td>
</tr>
<tr>
<td>volume</td>
<td>—</td>
</tr>
<tr>
<td>muted</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>inputVolumeMul</td>
</tr>
<tr>
<td>—</td>
<td>inputVolumeDb</td>
</tr>
</table>

----

### SetVolume

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setvolume">SetVolume</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setinputvolume">SetInputVolume</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>source</td>
<td>inputName</td>
</tr>
<tr>
<td>—</td>
<td>inputVolumeMul</td>
</tr>
<tr>
<td>—</td>
<td>inputVolumeDb</td>
</tr>
<tr>
<td>volume</td>
<td>—</td>
</tr>
<tr>
<td>useDecibel</td>
<td>—</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetAudioTracks

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getaudiotracks">GetAudioTracks</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputaudiotracks">GetInputAudioTracks</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>track1</td>
<td>—</td>
</tr>
<tr>
<td>track2</td>
<td>—</td>
</tr>
<tr>
<td>track3</td>
<td>—</td>
</tr>
<tr>
<td>track4</td>
<td>—</td>
</tr>
<tr>
<td>track5</td>
<td>—</td>
</tr>
<tr>
<td>track6</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>inputAudioTracks</td>
</tr>
</table>

----

### SetAudioTracks

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setaudiotracks">SetAudioTracks</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setinputaudiotracks">SetInputAudioTracks</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>track</td>
<td>—</td>
</tr>
<tr>
<td>active</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>inputAudioTracks</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetMute

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getmute">GetMute</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputmute">GetInputMute</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>source</td>
<td>inputName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>—</td>
</tr>
<tr>
<td>muted</td>
<td>inputMuted</td>
</tr>
</table>

----

### SetMute

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setmute">SetMute</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setinputmute">SetInputMute</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>source</td>
<td>inputName</td>
</tr>
<tr>
<td>mute</td>
<td>inputMuted</td>
</tr>
</table>

**Response fields:**

*None.*

----

### ToggleMute

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#togglemute">ToggleMute</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#toggleinputmute">ToggleInputMute</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>source</td>
<td>inputName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>inputMuted</td>
</tr>
</table>

----

### GetSourceActive

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsourceactive">GetSourceActive</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsourceactive">GetSourceActive</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceActive</td>
<td>videoActive</td>
</tr>
<tr>
<td>—</td>
<td>videoShowing</td>
</tr>
</table>

----

### GetAudioActive

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getaudioactive">GetAudioActive</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SetSourceName

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsourcename">SetSourceName</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setinputname">SetInputName</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>newName</td>
<td>newInputName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### SetSyncOffset

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsyncoffset">SetSyncOffset</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setinputaudiosyncoffset">SetInputAudioSyncOffset</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>source</td>
<td>inputName</td>
</tr>
<tr>
<td>offset</td>
<td>inputAudioSyncOffset</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetSyncOffset

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsyncoffset">GetSyncOffset</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputaudiosyncoffset">GetInputAudioSyncOffset</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>source</td>
<td>inputName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>—</td>
</tr>
<tr>
<td>offset</td>
<td>inputAudioSyncOffset</td>
</tr>
</table>

----

### GetSourceSettings, SetTextGDIPlusProperties, SetTextFreetype2Properties, GetBrowserSourceProperties

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsourcesettings">GetSourceSettings</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#settextgdiplusproperties">SetTextGDIPlusProperties</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#settextfreetype2properties">SetTextFreetype2Properties</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getbrowsersourceproperties">GetBrowserSourceProperties</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputsettings">GetInputSettings</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>sourceType</td>
<td>—</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>—</td>
</tr>
<tr>
<td>sourceType</td>
<td>inputKind</td>
</tr>
<tr>
<td>sourceSettings</td>
<td>inputSettings</td>
</tr>
</table>

----

### SetSourceSettings, GetTextGDIPlusProperties, GetTextFreetype2Properties, SetBrowserSourceProperties

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsourcesettings">SetSourceSettings</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#gettextgdiplusproperties">GetTextGDIPlusProperties</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#gettextfreetype2properties">GetTextFreetype2Properties</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setbrowsersourceproperties">SetBrowserSourceProperties</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setinputsettings">SetInputSettings</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>sourceType</td>
<td>—</td>
</tr>
<tr>
<td>sourceSettings</td>
<td>inputSettings</td>
</tr>
<tr>
<td>—</td>
<td>overlay</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>—</td>
</tr>
<tr>
<td>sourceType</td>
<td>—</td>
</tr>
<tr>
<td>sourceSettings</td>
<td>—</td>
</tr>
</table>

**Notes:**

* No longer reflects the new updated settings back.

----

### GetSpecialSources

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getspecialsources">GetSpecialSources</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getspecialinputs">GetSpecialInputs</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>desktop-1</td>
<td>desktop1</td>
</tr>
<tr>
<td>desktop-2</td>
<td>desktop2</td>
</tr>
<tr>
<td>mic-1</td>
<td>mic1</td>
</tr>
<tr>
<td>mic-2</td>
<td>mic2</td>
</tr>
<tr>
<td>mic-3</td>
<td>mic3</td>
</tr>
<tr>
<td>mic-4</td>
<td>mic4</td>
</tr>
</table>

----

### GetSourceFilters

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsourcefilters">GetSourceFilters</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsourcefilterlist">GetSourceFilterList</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>filters[].enabled</td>
<td>filters[].filterEnabled</td>
</tr>
<tr>
<td>filters[].type</td>
<td>filters[].filterKind</td>
</tr>
<tr>
<td>filters[].name</td>
<td>filters[].filterName</td>
</tr>
<tr>
<td>filters[].settings</td>
<td>filters[].filterSettings</td>
</tr>
<tr>
<td>—</td>
<td>filters[].filterIndex</td>
</tr>
</table>

----

### GetSourceFilterInfo

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsourcefilterinfo">GetSourceFilterInfo</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsourcefilter">GetSourceFilter</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>enabled</td>
<td>filterEnabled</td>
</tr>
<tr>
<td>type</td>
<td>filterKind</td>
</tr>
<tr>
<td>name</td>
<td>—</td>
</tr>
<tr>
<td>settings</td>
<td>filterSettings</td>
</tr>
<tr>
<td>—</td>
<td>filterIndex</td>
</tr>
</table>

----

### AddFilterToSource

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#addfiltertosource">AddFilterToSource</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#createsourcefilter">CreateSourceFilter</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
<tr>
<td>filterType</td>
<td>filterKind</td>
</tr>
<tr>
<td>filterSettings</td>
<td>filterSettings</td>
</tr>
</table>

**Response fields:**

*None.*

----

### RemoveFilterFromSource

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#removefilterfromsource">RemoveFilterFromSource</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#removesourcefilter">RemoveSourceFilter</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### ReorderSourceFilter

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#reordersourcefilter">ReorderSourceFilter</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsourcefilterindex">SetSourceFilterIndex</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
<tr>
<td>newIndex</td>
<td>filterIndex</td>
</tr>
</table>

**Response fields:**

*None.*

----

### MoveSourceFilter

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#movesourcefilter">MoveSourceFilter</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Gone as of v5. To achieve the same thing, retrieve the current index using **GetSourceFilter** and then use **SetSourceFilterIndex**.

----

### SetSourceFilterSettings

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsourcefiltersettings">SetSourceFilterSettings</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsourcefiltersettings">SetSourceFilterSettings</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
<tr>
<td>filterSettings</td>
<td>filterSettings</td>
</tr>
<tr>
<td>—</td>
<td>overlay</td>
</tr>
</table>

**Response fields:**

*None.*

----

### SetSourceFilterVisibility

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsourcefiltervisibility">SetSourceFilterVisibility</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsourcefilterenabled">SetSourceFilterEnabled</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
<tr>
<td>filterEnabled</td>
<td>filterEnabled</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetAudioMonitorType

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getaudiomonitortype">GetAudioMonitorType</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputaudiomonitortype">GetInputAudioMonitorType</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>monitorType</td>
<td>monitorType</td>
</tr>
</table>

----

### SetAudioMonitorType

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setaudiomonitortype">SetAudioMonitorType</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setinputaudiomonitortype">SetInputAudioMonitorType</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetSourceDefaultSettings

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsourcedefaultsettings">GetSourceDefaultSettings</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getinputdefaultsettings">GetInputDefaultSettings</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceKind</td>
<td>inputKind</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceKind</td>
<td>—</td>
</tr>
<tr>
<td>defaultSettings</td>
<td>defaultInputSettings</td>
</tr>
</table>

----

### TakeSourceScreenshot

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#takesourcescreenshot">TakeSourceScreenshot</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsourcescreenshot">GetSourceScreenshot</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#savesourcescreenshot">SaveSourceScreenshot</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Split into two different requests, depending on whether you want to save the image or get it data URI encoded.

----

### RefreshBrowserSource

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#refreshbrowsersource">RefreshBrowserSource</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#pressinputpropertiesbutton">PressInputPropertiesButton</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>—</td>
<td>propertyName</td>
</tr>
</table>

**Response fields:**

*None.*

**Notes:**

* Gone as of v5. To achieve the same thing, use **PressInputPropertiesButton** and set **propertyName** to **"refreshnocache"**.

----

### ListOutputs

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#listoutputs">ListOutputs</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getoutputlist">GetOutputList</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>outputs</td>
<td>outputs</td>
</tr>
</table>

----

### GetOutputInfo

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getoutputinfo">GetOutputInfo</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getoutputstatus">GetOutputStatus</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>outputName</td>
<td>outputName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>active</td>
<td>outputActive</td>
</tr>
<tr>
<td>congestion</td>
<td>outputCongestion</td>
</tr>
<tr>
<td>droppedFrames</td>
<td>outputSkippedFrames</td>
</tr>
<tr>
<td>reconnecting</td>
<td>outputReconnecting</td>
</tr>
<tr>
<td>settings</td>
<td>—</td>
</tr>
<tr>
<td>totalBytes</td>
<td>outputBytes</td>
</tr>
<tr>
<td>totalFrames</td>
<td>outputTotalFrames</td>
</tr>
<tr>
<td>—</td>
<td>outputDuration</td>
</tr>
<tr>
<td>—</td>
<td>outputTimecode</td>
</tr>
<tr>
<td>type</td>
<td>—</td>
</tr>
<tr>
<td>flags</td>
<td>—</td>
</tr>
<tr>
<td>width</td>
<td>—</td>
</tr>
<tr>
<td>height</td>
<td>—</td>
</tr>
<tr>
<td>name</td>
<td>—</td>
</tr>
</table>

----

### StartOutput

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startoutput">StartOutput</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#startoutput">StartOutput</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>outputName</td>
<td>outputName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### StopOutput

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#stopoutput">StopOutput</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#stopoutput">StopOutput</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>outputName</td>
<td>outputName</td>
</tr>
<tr>
<td>force</td>
<td>—</td>
</tr>
</table>

**Response fields:**

*None.*

----

### SetCurrentProfile

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setcurrentprofile">SetCurrentProfile</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setcurrentprofile">SetCurrentProfile</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>profile-name</td>
<td>profileName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetCurrentProfile

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getcurrentprofile">GetCurrentProfile</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getprofilelist">GetProfileList</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>profile-name</td>
<td>currentProfileName</td>
</tr>
<tr>
<td>—</td>
<td>profiles</td>
</tr>
</table>

----

### ListProfiles

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#listprofiles">ListProfiles</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getprofilelist">GetProfileList</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>currentProfileName</td>
</tr>
<tr>
<td>profiles</td>
<td>profiles</td>
</tr>
</table>

----

### GetRecordingStatus

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getrecordingstatus">GetRecordingStatus</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getrecordstatus">GetRecordStatus</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>isRecording</td>
<td>outputActive</td>
</tr>
<tr>
<td>isRecordingPaused</td>
<td>outputPaused</td>
</tr>
<tr>
<td>recordTimecode</td>
<td>outputTimecode</td>
</tr>
<tr>
<td>recordingFilename</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>outputDuration</td>
</tr>
<tr>
<td>—</td>
<td>outputBytes</td>
</tr>
</table>

----

### StartStopRecording

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startstoprecording">StartStopRecording</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#togglerecord">ToggleRecord</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### StartRecording

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startrecording">StartRecording</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#startrecord">StartRecord</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### StopRecording

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#stoprecording">StopRecording</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#stoprecord">StopRecord</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>outputPath</td>
</tr>
</table>

----

### PauseRecording

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#pauserecording">PauseRecording</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#pauserecord">PauseRecord</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#togglerecordpause">ToggleRecordPause</a>

----

### ResumeRecording

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#resumerecording">ResumeRecording</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#resumerecord">ResumeRecord</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#togglerecordpause">ToggleRecordPause</a>

----

### SetRecordingFolder

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setrecordingfolder">SetRecordingFolder</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setrecorddirectory">SetRecordDirectory</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>rec-folder</td>
<td>recordDirectory</td>
</tr>
</table>

----

### GetRecordingFolder

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getrecordingfolder">GetRecordingFolder</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getrecorddirectory">GetRecordDirectory</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>rec-folder</td>
<td>recordDirectory</td>
</tr>
</table>

----

### GetReplayBufferStatus

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getreplaybufferstatus">GetReplayBufferStatus</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getreplaybufferstatus">GetReplayBufferStatus</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>isReplayBufferActive</td>
<td>outputActive</td>
</tr>
</table>

**Response fields:**

*None.*

----

### StartStopReplayBuffer

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startstopreplaybuffer">StartStopReplayBuffer</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#togglereplaybuffer">ToggleReplayBuffer</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>outputActive</td>
</tr>
</table>

**Response fields:**

*None.*

----

### StartReplayBuffer

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startreplaybuffer">StartReplayBuffer</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#startreplaybuffer">StartReplayBuffer</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### StopReplayBuffer

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#stopreplaybuffer">StopReplayBuffer</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#stopreplaybuffer">StopReplayBuffer</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SaveReplayBuffer

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#savereplaybuffer">SaveReplayBuffer</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#savereplaybuffer">SaveReplayBuffer</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SetCurrentSceneCollection

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setcurrentscenecollection">SetCurrentSceneCollection</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setcurrentscenecollection">SetCurrentSceneCollection</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sc-name</td>
<td>sceneCollectionName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetCurrentSceneCollection

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getcurrentscenecollection">GetCurrentSceneCollection</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getscenecollectionlist">GetSceneCollectionList</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sc-name</td>
<td>currentSceneCollectionName</td>
</tr>
<tr>
<td>—</td>
<td>sceneCollections</td>
</tr>
</table>

----

### ListSceneCollections

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#listscenecollections">ListSceneCollections</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getscenecollectionlist">GetSceneCollectionList</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>currentSceneCollectionName</td>
</tr>
<tr>
<td>scene-collections</td>
<td>sceneCollections</td>
</tr>
</table>

----

### GetSceneItemList

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsceneitemlist">GetSceneItemList</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsceneitemlist">GetSceneItemList</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneName</td>
<td>sceneName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneName</td>
<td>—</td>
</tr>
<tr>
<td>sceneItems[].itemId</td>
<td>sceneItems[].sceneItemId</td>
</tr>
<tr>
<td>—</td>
<td>sceneItems[].sceneItemIndex</td>
</tr>
<tr>
<td>sceneItems[].sourceKind</td>
<td>sceneItems[].inputKind</td>
</tr>
<tr>
<td>sceneItems[].sourceName</td>
<td>sceneItems[].sourceName</td>
</tr>
<tr>
<td>sceneItems[].sourceType</td>
<td>sceneItems[].sourceType</td>
</tr>
<tr>
<td>—</td>
<td>sceneItems[].sceneItemBlendMode</td>
</tr>
<tr>
<td>—</td>
<td>sceneItems[].sceneItemEnabled</td>
</tr>
<tr>
<td>—</td>
<td>sceneItems[].sceneItemLocked</td>
</tr>
<tr>
<td>—</td>
<td>sceneItems[].sceneItemTransform</td>
</tr>
<tr>
<td>—</td>
<td>sceneItems[].isGroup</td>
</tr>
</table>

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getgroupsceneitemlist">GetGroupSceneItemList</a>

----

### GetSceneItemProperties

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getsceneitemproperties">GetSceneItemProperties</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsceneitemtransform">GetSceneItemTransform</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsceneitemenabled">GetSceneItemEnabled</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsceneitemlocked">GetSceneItemLocked</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsceneitemindex">GetSceneItemIndex</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getsceneitemblendmode">GetSceneItemBlendMode</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SetSceneItemProperties

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsceneitemproperties">SetSceneItemProperties</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemtransform">SetSceneItemTransform</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemenabled">SetSceneItemEnabled</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemlocked">SetSceneItemLocked</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemindex">SetSceneItemIndex</a><br><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemblendmode">SetSceneItemBlendMode</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### ResetSceneItem

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#resetsceneitem">ResetSceneItem</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SetSceneItemRender

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsceneitemrender">SetSceneItemRender</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemenabled">SetSceneItemEnabled</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>source</td>
<td>—</td>
</tr>
<tr>
<td>item</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>render</td>
<td>sceneItemEnabled</td>
</tr>
</table>

**Response fields:**

*None.*

----

### SetSceneItemPosition

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsceneitemposition">SetSceneItemPosition</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SetSceneItemTransform

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsceneitemtransform">SetSceneItemTransform</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemtransform">SetSceneItemTransform</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item</td>
<td>—</td>
</tr>
<tr>
<td>x-scale</td>
<td>—</td>
</tr>
<tr>
<td>y-scale</td>
<td>—</td>
</tr>
<tr>
<td>rotation</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>—</td>
<td>sceneItemTransform</td>
</tr>
</table>

**Response fields:**

*None.*

----

### SetSceneItemCrop

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setsceneitemcrop">SetSceneItemCrop</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setsceneitemtransform">SetSceneItemTransform</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item</td>
<td>—</td>
</tr>
<tr>
<td>top</td>
<td>—</td>
</tr>
<tr>
<td>bottom</td>
<td>—</td>
</tr>
<tr>
<td>left</td>
<td>—</td>
</tr>
<tr>
<td>right</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>—</td>
<td>sceneItemTransform</td>
</tr>
<tr>
<td>—</td>
<td>-</td>
</tr>
</table>

**Response fields:**

*None.*

----

### DeleteSceneItem

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#deletesceneitem">DeleteSceneItem</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#removesceneitem">RemoveSceneItem</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene</td>
<td>sceneName</td>
</tr>
<tr>
<td>item.name</td>
<td>—</td>
</tr>
<tr>
<td>item.id</td>
<td>sceneItemId</td>
</tr>
</table>

**Response fields:**

*None.*

----

### AddSceneItem

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#addsceneitem">AddSceneItem</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#createsceneitem">CreateSceneItem</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneName</td>
<td>sceneName</td>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>setVisible</td>
<td>sceneItemEnabled</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>itemId</td>
<td>sceneItemId</td>
</tr>
</table>

----

### DuplicateSceneItem

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#duplicatesceneitem">DuplicateSceneItem</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#duplicatesceneitem">DuplicateSceneItem</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>fromScene</td>
<td>sceneName</td>
</tr>
<tr>
<td>toScene</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>item.name</td>
<td>destinationSceneName</td>
</tr>
<tr>
<td>item.id</td>
<td>-</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene</td>
<td>—</td>
</tr>
<tr>
<td>item.id</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>item.name</td>
<td>—</td>
</tr>
</table>

----

### SetCurrentScene

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setcurrentscene">SetCurrentScene</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setcurrentprogramscene">SetCurrentProgramScene</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
</table>

**Response fields:**

*None.*

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setcurrentpreviewscene">SetCurrentPreviewScene</a>

----

### GetCurrentScene

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getcurrentscene">GetCurrentScene</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getcurrentprogramscene">GetCurrentProgramScene</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>currentProgramSceneName</td>
</tr>
<tr>
<td>sources</td>
<td>—</td>
</tr>
</table>

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getcurrentpreviewscene">GetCurrentPreviewScene</a>

----

### GetSceneList

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getscenelist">GetSceneList</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getscenelist">GetSceneList</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>current-scene</td>
<td>currentProgramSceneName</td>
</tr>
<tr>
<td>—</td>
<td>currentPreviewSceneName</td>
</tr>
<tr>
<td>scenes</td>
<td>scenes</td>
</tr>
</table>

----

### CreateScene

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#createscene">CreateScene</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#createscene">CreateScene</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneName</td>
<td>sceneName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### ReorderSceneItems

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#reordersceneitems">ReorderSceneItems</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SetSceneTransitionOverride

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setscenetransitionoverride">SetSceneTransitionOverride</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setscenescenetransitionoverride">SetSceneSceneTransitionOverride</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneName</td>
<td>sceneName</td>
</tr>
<tr>
<td>transitionName</td>
<td>transitionName</td>
</tr>
<tr>
<td>transitionDuration</td>
<td>transitionDuration</td>
</tr>
</table>

**Response fields:**

*None.*

----

### RemoveSceneTransitionOverride

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#removescenetransitionoverride">RemoveSceneTransitionOverride</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Gone as of v5. To achieve the same thing, use **SetSceneSceneTransitionOverride** and set **transitionName** and **transitionDuration** to **null**.

----

### GetSceneTransitionOverride

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getscenetransitionoverride">GetSceneTransitionOverride</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getscenescenetransitionoverride">GetSceneSceneTransitionOverride</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneName</td>
<td>sceneName</td>
</tr>
</table>

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>transitionName</td>
<td>transitionName</td>
</tr>
<tr>
<td>transitionDuration</td>
<td>transitionDuration</td>
</tr>
</table>

----

### GetStreamingStatus

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getstreamingstatus">GetStreamingStatus</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getstreamstatus">GetStreamStatus</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>streaming</td>
<td>outputActive</td>
</tr>
<tr>
<td>recording</td>
<td>—</td>
</tr>
<tr>
<td>recording-paused</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>outputReconnecting</td>
</tr>
<tr>
<td>virtualcam</td>
<td>—</td>
</tr>
<tr>
<td>preview-only</td>
<td>—</td>
</tr>
<tr>
<td>stream-timecode</td>
<td>outputTimecode</td>
</tr>
<tr>
<td>rec-timecode</td>
<td>—</td>
</tr>
<tr>
<td>virtualcam-timecode</td>
<td>—</td>
</tr>
</table>

**Notes:**

* In v4, this one call also returned information about the recording and virtual cam status. As of v5, this has been moved to the separate **GetRecordStatus** and **GetVirtualCamStatus** calls.

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getrecordstatus">GetRecordStatus</a>
* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getvirtualcamstatus">GetVirtualCamStatus</a>

----

### StartStopStreaming

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startstopstreaming">StartStopStreaming</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#togglestream">ToggleStream</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>outputActive</td>
</tr>
</table>

----

### StartStreaming

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startstreaming">StartStreaming</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#startstream">StartStream</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### StopStreaming

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#stopstreaming">StopStreaming</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#stopstream">StopStream</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SetStreamSettings

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setstreamsettings">SetStreamSettings</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setstreamservicesettings">SetStreamServiceSettings</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>type</td>
<td>streamServiceType</td>
</tr>
<tr>
<td>settings</td>
<td>streamServiceSettings</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetStreamSettings

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getstreamsettings">GetStreamSettings</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getstreamservicesettings">GetStreamServiceSettings</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>type</td>
<td>streamServiceType</td>
</tr>
<tr>
<td>settings</td>
<td>streamServiceSettings</td>
</tr>
</table>

----

### SaveStreamSettings

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#savestreamsettings">SaveStreamSettings</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SendCaptions

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sendcaptions">SendCaptions</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sendstreamcaption">SendStreamCaption</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>text</td>
<td>captionText</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetStudioModeStatus

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getstudiomodestatus">GetStudioModeStatus</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getstudiomodeenabled">GetStudioModeEnabled</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>studio-mode</td>
<td>studioModeEnabled</td>
</tr>
</table>

----

### GetPreviewScene

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getpreviewscene">GetPreviewScene</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getcurrentpreviewscene">GetCurrentPreviewScene</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>currentPreviewSceneName</td>
</tr>
<tr>
<td>sources</td>
<td>—</td>
</tr>
</table>

----

### SetPreviewScene

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setpreviewscene">SetPreviewScene</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setcurrentpreviewscene">SetCurrentPreviewScene</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### TransitionToProgram

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#transitiontoprogram">TransitionToProgram</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#triggerstudiomodetransition">TriggerStudioModeTransition</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>with-transition.name</td>
<td>—</td>
</tr>
<tr>
<td>with-transition.duration</td>
<td>—</td>
</tr>
</table>

**Response fields:**

*None.*

**Notes:**

* TODO. Unknown if it's still possible to override the transition like in v4.

----

### EnableStudioMode

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#enablestudiomode">EnableStudioMode</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setstudiomodeenabled">SetStudioModeEnabled</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>studioModeEnabled</td>
</tr>
</table>

**Response fields:**

*None.*

----

### DisableStudioMode

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#disablestudiomode">DisableStudioMode</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getstudiomodeenabled">GetStudioModeEnabled</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>studioModeEnabled</td>
</tr>
</table>

**Response fields:**

*None.*

----

### ToggleStudioMode

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#togglestudiomode">ToggleStudioMode</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Gone as of v5. Instead, keep track of whether studio mode is enabled (use **[GetStudioModeStatus](migration-guide.md#getstudiomodestatus)**) and then use either **EnableStudioMode** or **DisableStudioMode**.

----

### GetTransitionList

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#gettransitionlist">GetTransitionList</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getscenetransitionlist">GetSceneTransitionList</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>current-transition</td>
<td>currentSceneTransitionName</td>
</tr>
<tr>
<td>—</td>
<td>currentSceneTransitionKind</td>
</tr>
<tr>
<td>transitions[].name</td>
<td>transitions[].transitionName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetCurrentTransition

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getcurrenttransition">GetCurrentTransition</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getcurrentscenetransition">GetCurrentSceneTransition</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>transitionName</td>
</tr>
<tr>
<td>duration</td>
<td>transitionDuration</td>
</tr>
<tr>
<td>—</td>
<td>transitionKind</td>
</tr>
<tr>
<td>—</td>
<td>transitionFixed</td>
</tr>
<tr>
<td>—</td>
<td>transitionConfigurable</td>
</tr>
<tr>
<td>—</td>
<td>transitionSettings</td>
</tr>
</table>

**Response fields:**

*None.*

----

### SetCurrentTransition

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#setcurrenttransition">SetCurrentTransition</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setcurrentscenetransition">SetCurrentSceneTransition</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>transition-name</td>
<td>transitionName</td>
</tr>
</table>

**Response fields:**

*None.*

----

### SetTransitionDuration

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#settransitionduration">SetTransitionDuration</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#setcurrentscenetransitionduration">SetCurrentSceneTransitionDuration</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>duration</td>
<td>transitionDuration</td>
</tr>
</table>

**Response fields:**

*None.*

----

### GetTransitionDuration

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#gettransitionduration">GetTransitionDuration</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* TODO. Use **GetCurrentSceneTransition**.

----

### GetTransitionPosition

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#gettransitionposition">GetTransitionPosition</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getcurrentscenetransitioncursor">GetCurrentSceneTransitionCursor</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>position</td>
<td>transitionCursor</td>
</tr>
</table>

----

### GetTransitionSettings

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#gettransitionsettings">GetTransitionSettings</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* TODO. Use **GetCurrentSceneTransition**.

----

### SetTransitionSettings

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#settransitionsettings">SetTransitionSettings</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Use **SetCurrentSceneTransitionSettings**. Unknown if it's currently possible to set the transition settings of a different one than the current scene transition.

----

### ReleaseTBar

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#releasetbar">ReleaseTBar</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Gone as of v5. To achieve the same thing, use **SetTBarPosition** and set **"release"** to **true**.

----

### SetTBarPosition

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#settbarposition">SetTBarPosition</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#settbarposition">SetTBarPosition</a></td>
</tr>
</table>

**Request fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>position</td>
<td>position</td>
</tr>
<tr>
<td>release</td>
<td>release</td>
</tr>
</table>

**Response fields:**

*None.*

**Notes:**

* Deprecated and very likely to be removed in a future update.

----

### GetVirtualCamStatus

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#getvirtualcamstatus">GetVirtualCamStatus</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#getvirtualcamstatus">GetVirtualCamStatus</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>isVirtualCam</td>
<td>outputActive</td>
</tr>
<tr>
<td>virtualCamTimecode</td>
<td>—</td>
</tr>
</table>

**Notes:**

* TODO. Unknown if it's possible to get **virtualCamTimecode** in any way in v5.

----

### StartStopVirtualCam

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startstopvirtualcam">StartStopVirtualCam</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#togglevirtualcam">ToggleVirtualCam</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>outputActive</td>
</tr>
</table>

----

### StartVirtualCam

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#startvirtualcam">StartVirtualCam</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#startvirtualcam">StartVirtualCam</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### StopVirtualCam

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#stopvirtualcam">StopVirtualCam</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#stopvirtualcam">StopVirtualCam</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

## Events

### SwitchScenes

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#switchscenes">SwitchScenes</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentprogramscenechanged">CurrentProgramSceneChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>sources</td>
<td>—</td>
</tr>
</table>

----

### ScenesChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneschanged">ScenesChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#scenelistchanged">SceneListChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scenes</td>
<td>scenes</td>
</tr>
</table>

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#scenecreated">SceneCreated</a>
* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneremoved">SceneRemoved</a>
* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#scenenamechanged">SceneNameChanged</a>

----

### SceneCollectionChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#scenecollectionchanged">SceneCollectionChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentscenecollectionchanged">CurrentSceneCollectionChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneCollection</td>
<td>sceneCollectionName</td>
</tr>
</table>

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentscenecollectionchanging">CurrentSceneCollectionChanging</a>

----

### SceneCollectionListChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#scenecollectionlistchanged">SceneCollectionListChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#scenecollectionlistchanged">SceneCollectionListChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sceneCollections</td>
<td>sceneCollections</td>
</tr>
</table>

**Notes:**

* **sceneCollections** changed from **Array\<{name: string}>** to **Array\<string>**.

----

### SwitchTransition

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#switchtransition">SwitchTransition</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentscenetransitionchanged">CurrentSceneTransitionChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>transition-name</td>
<td>transitionName</td>
</tr>
</table>

----

### TransitionListChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#transitionlistchanged">TransitionListChanged</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### TransitionDurationChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#transitiondurationchanged">TransitionDurationChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentscenetransitiondurationchanged">CurrentSceneTransitionDurationChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>new-duration</td>
<td>transitionDuration</td>
</tr>
</table>

----

### TransitionBegin

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#transitionbegin">TransitionBegin</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#scenetransitionstarted">SceneTransitionStarted</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>transitionName</td>
</tr>
<tr>
<td>type</td>
<td>—</td>
</tr>
<tr>
<td>duration</td>
<td>—</td>
</tr>
<tr>
<td>from-scene</td>
<td>—</td>
</tr>
<tr>
<td>to-scene</td>
<td>—</td>
</tr>
</table>

----

### TransitionEnd

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#transitionend">TransitionEnd</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#scenetransitionended">SceneTransitionEnded</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>transitionName</td>
</tr>
<tr>
<td>type</td>
<td>—</td>
</tr>
<tr>
<td>duration</td>
<td>—</td>
</tr>
<tr>
<td>to-scene</td>
<td>—</td>
</tr>
</table>

----

### TransitionVideoEnd

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#transitionvideoend">TransitionVideoEnd</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#scenetransitionvideoended">SceneTransitionVideoEnded</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>name</td>
<td>transitionName</td>
</tr>
<tr>
<td>type</td>
<td>—</td>
</tr>
<tr>
<td>duration</td>
<td>—</td>
</tr>
<tr>
<td>from-scene</td>
<td>—</td>
</tr>
<tr>
<td>to-scene</td>
<td>—</td>
</tr>
</table>

----

### ProfileChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#profilechanged">ProfileChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentprofilechanged">CurrentProfileChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>profile</td>
<td>profileName</td>
</tr>
</table>

**See also:**

* <a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentprofilechanging">CurrentProfileChanging</a>

----

### ProfileListChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#profilelistchanged">ProfileListChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#profilelistchanged">ProfileListChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>profiles</td>
<td>profiles</td>
</tr>
</table>

**Notes:**

* **profiles** changed from **Array\<{name: string}>** to **Array\<string>**.

----

### StreamStarting, StreamStarted, StreamStopping, StreamStopped

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#streamstarting">StreamStarting</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#streamstarted">StreamStarted</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#streamstopping">StreamStopping</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#streamstopped">StreamStopped</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#streamstatechanged">StreamStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>preview-only</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>outputActive</td>
</tr>
<tr>
<td>—</td>
<td>outputState</td>
</tr>
</table>

----

### StreamStatus

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#streamstatus">StreamStatus</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Gone as of v5. To achieve the same thing, send a **GetStats** request.

----

### RecordingStarting, RecordingStarted, RecordingStopping, RecordingStopped, RecordingPaused, RecordingResumed

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#recordingstarting">RecordingStarting</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#recordingstarted">RecordingStarted</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#recordingstopping">RecordingStopping</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#recordingstopped">RecordingStopped</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#recordingpaused">RecordingPaused</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#recordingresumed">RecordingResumed</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#recordstatechanged">RecordStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>outputActive</td>
</tr>
<tr>
<td>—</td>
<td>outputState</td>
</tr>
<tr>
<td>recordingFilename</td>
<td>outputPath</td>
</tr>
</table>

----

### VirtualCamStarted, VirtualCamStopped

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#virtualcamstarted">VirtualCamStarted</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#virtualcamstopped">VirtualCamStopped</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#virtualcamstatechanged">VirtualcamStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>outputActive</td>
</tr>
<tr>
<td>—</td>
<td>outputState</td>
</tr>
</table>

----

### ReplayStarting, ReplayStarted, ReplayStopping, ReplayStopped

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#replaystarting">ReplayStarting</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#replaystarted">ReplayStarted</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#replaystopping">ReplayStopping</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#replaystopped">ReplayStopped</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#replaybufferstatechanged">ReplayBufferStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>outputActive</td>
</tr>
<tr>
<td>—</td>
<td>outputState</td>
</tr>
</table>

----

### Exiting

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#exiting">Exiting</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#exitstarted">ExitStarted</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### Heartbeat

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#heartbeat">Heartbeat</a></td>
<td valign="top">–</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

**Notes:**

* Gone as of v5. To achieve the same thing, send a **GetStats** request on a timer.

----

### BroadcastCustomMessage

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#broadcastcustommessage">BroadcastCustomMessage</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#customevent">CustomEvent</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>realm</td>
<td>—</td>
</tr>
<tr>
<td>data</td>
<td>eventData</td>
</tr>
</table>

----

### SourceCreated

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcecreated">SourceCreated</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemcreated">SceneItemCreated</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>sceneName</td>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>sourceType</td>
<td>—</td>
</tr>
<tr>
<td>sourceKind</td>
<td>—</td>
</tr>
<tr>
<td>sourceSettings</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>—</td>
<td>sceneItemIndex</td>
</tr>
</table>

----

### SourceDestroyed

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcedestroyed">SourceDestroyed</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemremoved">SceneItemRemoved</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>—</td>
<td>sceneName</td>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>sourceType</td>
<td>—</td>
</tr>
<tr>
<td>sourceKind</td>
<td>—</td>
</tr>
</table>

----

### SourceVolumeChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcevolumechanged">SourceVolumeChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#inputvolumechanged">InputVolumeChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>volume</td>
<td>inputVolumeMul</td>
</tr>
<tr>
<td>volumeDb</td>
<td>inputVolumeDb</td>
</tr>
</table>

----

### SourceMuteStateChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcemutestatechanged">SourceMuteStateChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#inputmutestatechanged">InputMuteStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>muted</td>
<td>inputMuted</td>
</tr>
</table>

----

### SourceAudioDeactivated, SourceAudioActivated

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourceaudiodeactivated">SourceAudioDeactivated</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourceaudioactivated">SourceAudioActivated</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SourceAudioSyncOffsetChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourceaudiosyncoffsetchanged">SourceAudioSyncOffsetChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#inputaudiosyncoffsetchanged">InputAudioSyncOffsetChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>syncOffset</td>
<td>inputAudioSyncOffset</td>
</tr>
</table>

**Notes:**

* The sync offset has been changed from nanoseconds to milliseconds (milliseconds = nanoseconds × 1000000).

----

### SourceAudioMixersChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourceaudiomixerschanged">SourceAudioMixersChanged</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### SourceRenamed

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcerenamed">SourceRenamed</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#inputnamechanged">InputNameChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>previousName</td>
<td>oldInputName</td>
</tr>
<tr>
<td>newName</td>
<td>inputName</td>
</tr>
<tr>
<td>sourceType</td>
<td>—</td>
</tr>
</table>

----

### SourceFilterAdded

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcefilteradded">SourceFilterAdded</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sourcefiltercreated">SourceFilterCreated</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
<tr>
<td>filterType</td>
<td>filterKind</td>
</tr>
<tr>
<td>—</td>
<td>filterIndex</td>
</tr>
<tr>
<td>filterSettings</td>
<td>filterSettings</td>
</tr>
<tr>
<td>—</td>
<td>defaultFilterSettings</td>
</tr>
</table>

----

### SourceFilterRemoved

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcefilterremoved">SourceFilterRemoved</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sourcefilterremoved">SourceFilterRemoved</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
<tr>
<td>filterType</td>
<td>—</td>
</tr>
</table>

----

### SourceFilterVisibilityChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcefiltervisibilitychanged">SourceFilterVisibilityChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sourcefilterenablestatechanged">SourceFilterEnableStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filterName</td>
<td>filterName</td>
</tr>
<tr>
<td>filterEnabled</td>
<td>filterEnabled</td>
</tr>
</table>

----

### SourceFiltersReordered

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourcefiltersreordered">SourceFiltersReordered</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sourcefilterlistreindexed">SourceFilterListReindexed</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>sourceName</td>
</tr>
<tr>
<td>filters</td>
<td>filters</td>
</tr>
</table>

----

### MediaStarted

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#mediastarted">MediaStarted</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#mediainputplaybackstarted">MediaInputPlaybackStarted</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>sourceKind</td>
<td>—</td>
</tr>
</table>

----

### MediaEnded

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#mediaended">MediaEnded</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#mediainputplaybackended">MediaInputPlaybackEnded</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>sourceKind</td>
<td>-</td>
</tr>
</table>

----

### MediaPlaying, MediaPaused, MediaRestarted, MediaStopped, MediaNext, MediaPrevious

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#mediaplaying">MediaPlaying</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#mediapaused">MediaPaused</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#mediarestarted">MediaRestarted</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#mediastopped">MediaStopped</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#medianext">MediaNext</a><br><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#mediaprevious">MediaPrevious</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#mediainputactiontriggered">MediaInputActionTriggered</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>sourceName</td>
<td>inputName</td>
</tr>
<tr>
<td>sourceKind</td>
<td>—</td>
</tr>
<tr>
<td>—</td>
<td>mediaAction</td>
</tr>
</table>

----

### SourceOrderChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sourceorderchanged">SourceOrderChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemlistreindexed">SceneItemListReindexed</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>scene-items</td>
<td>sceneItems</td>
</tr>
</table>

----

### SceneItemAdded

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneitemadded">SceneItemAdded</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemcreated">SceneItemCreated</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item-name</td>
<td>sourceName</td>
</tr>
<tr>
<td>item-id</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>—</td>
<td>sceneItemIndex</td>
</tr>
</table>

----

### SceneItemRemoved

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneitemremoved">SceneItemRemoved</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemremoved">SceneItemRemoved</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item-name</td>
<td>sourceName</td>
</tr>
<tr>
<td>item-id</td>
<td>sceneItemId</td>
</tr>
</table>

----

### SceneItemVisibilityChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneitemvisibilitychanged">SceneItemVisibilityChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemenablestatechanged">SceneItemEnableStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item-name</td>
<td>—</td>
</tr>
<tr>
<td>item-id</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>item-visible</td>
<td>sceneItemEnabled</td>
</tr>
</table>

----

### SceneItemLockChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneitemlockchanged">SceneItemLockChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemlockstatechanged">SceneItemLockStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item-name</td>
<td>—</td>
</tr>
<tr>
<td>item-id</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>item-locked</td>
<td>sceneItemLocked</td>
</tr>
</table>

----

### SceneItemTransformChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneitemtransformchanged">SceneItemTransformChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemtransformchanged">SceneItemTransformChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item-name</td>
<td>—</td>
</tr>
<tr>
<td>item-id</td>
<td>sceneItemId</td>
</tr>
<tr>
<td>transform</td>
<td>sceneItemTransform</td>
</tr>
</table>

----

### SceneItemSelected

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneitemselected">SceneItemSelected</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#sceneitemselected">SceneItemSelected</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>item-name</td>
<td>—</td>
</tr>
<tr>
<td>item-id</td>
<td>sceneItemId</td>
</tr>
</table>

----

### SceneItemDeselected

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#sceneitemdeselected">SceneItemDeselected</a></td>
<td valign="top">?</td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

*None.*

----

### PreviewSceneChanged

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#previewscenechanged">PreviewSceneChanged</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#currentpreviewscenechanged">CurrentPreviewSceneChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>scene-name</td>
<td>sceneName</td>
</tr>
<tr>
<td>sources</td>
<td>—</td>
</tr>
</table>

----

### StudioModeSwitched

**Name:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#studiomodeswitched">StudioModeSwitched</a></td>
<td valign="top"><a href="https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#studiomodestatechanged">StudioModeStateChanged</a></td>
</tr>
</table>

**Request fields:**

*None.*

**Response fields:**

<table>
<tr>
<th>Protocol v4.9.1</th>
<th>Protocol v5 (RPC 1)</th>
</tr>
<tr>
<td>new-state</td>
<td>studioModeEnabled</td>
</tr>
</table>

## External links

* [obs-websocket 4.9.1 Protocol](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md) ([JSON](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/comments.json), [requests](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#requests), [events](https://github.com/obsproject/obs-websocket/blob/310c297a3655f8c3132c1f936e7cb1674e6a724c/docs/generated/protocol.md#events))
* [obs-websocket 5.1.0 Protocol](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md) ([JSON](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.json), [requests](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#requests), [events](https://github.com/obsproject/obs-websocket/blob/6db08f960e8cdf93cf6afc7059d61dc3c811b465/docs/generated/protocol.md#events), [design goals](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md#design-goals))
* [obs-websocket Wiki: Notable Changes between 4.x and 5.x](https://github.com/obsproject/obs-websocket/wiki/Notable-changes-between-4.x-and-5.x)
* [obs-websocket-js 5.0.0 Release: Breaking Changes](https://github.com/obs-websocket-community-projects/obs-websocket-js/releases/tag/v5.0.0)
* [obs-websocket](https://github.com/obsproject/obs-websocket) ([v4](https://github.com/obsproject/obs-websocket/tree/8823ecd2094ffa17dac6ceaa2987981b12f0820a), [v5](https://github.com/obsproject/obs-websocket/tree/6fd18a7ef1ecb149e8444154af1daab61d4241a9))
* [obs-websocket-js](https://github.com/obs-websocket-community-projects/obs-websocket-js) ([v4](https://github.com/obs-websocket-community-projects/obs-websocket-js/tree/v4), [v5](https://github.com/obs-websocket-community-projects/obs-websocket-js/tree/d6244bb26c473ced366d710d091f4a51c5d76fc2))
