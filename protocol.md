The functions in the [protocol](protocol.md) namespace provide a low-level interface for
managing one's Google Play Music library. All of the functions are
asynchronous, returning their "return values" via their last parameter, a
`callback` function. Following Node.js conventions, the callback takes as its
first parameter an error argument, which will be `null` if the function
finished successfully. When the error argument is not `null`, it is one of:

**[window#XMLHttpRequest](window#XMLHttpRequest.md):**
Indicates that a remote procedure call in the function failed. See the
`status` and `statusText` properties of the object to diagnose what went
wrong. `statusText === 'Unauthorized'` and `statusText === 'Forbidden'` are
particularly common errors, indicating an invalid OAuth 2.0 token and
unregistered client, respectively.

**[window#Error](window#Error.md):**
Indicates that there was an error in parsing the response to a remote
procedure call.

**some function-specific value:**
Indicates that the remote procedure call returned an error value.

In the event of an error, no other arguments are passed to the callback. With
the exception of [#readMetadata](#readMetadata.md), all of the following functions take as
their first two parameters an OAuth 2.0 `token` and an `id` string,
respectively.

# getDownloadLink [(token, id, serverId, callback)](method.md) #

Returns a download URL for the song with the given `serverId`. This doesn't
count towards the download limit given in the web interface.

Actually using this function in client-side code is bothersome, as sending the
`sjsaid` cookie with the underlying request apparently causes it to be
rejected, and [window#XMLHttpRequest](window#XMLHttpRequest.md) defaults to sending all cookies for a
domain. Clients with an ID string containing the character `@` have also been
known to cause this request to fail.

# getTracksToExport [(token, id, type, (contToken,) callback)](method.md) #

Lists all songs in a user's Google Play Music library if `type === 1`, and
only the purchased and promotional ones if `type === 2`. The return value is
an object of the following format:

```
{
	"continuation_token": string,
	"download_track_info": [{
		"id": string,
		"title": string,
		"album": string,
		"album_artist": string,
		"artist": string,
		"track_number": number,
		"track_size": number (in bytes)
	}, ...]
}
```

Notably, `id` is the `serverId` argument that [#getDownloadLink](#getDownloadLink.md) requires.

The songs are listed in chunks: the next chunk can be retrieved by passing the
`continuation_token` received in the last response (if any) as the optional
`contToken` parameter of the next call.

The error value is an object with a property `status` that contains an error
code between 2 and 5, signifying:
```
2	TRANSIENT_ERROR
3	MAX_NUM_CLIENTS_REACHED
4	UNABLE_TO_AUTHENTICATE_CLIENT
5	UNABLE_TO_REGISTER_CLIENT
```
The error definitions would seem to suggest that this request also tries to
register the `id` if it is not already, but this has not been confirmed.

# getUploadSession [(token, id, clientId, serverId, bitrate, callback)](method.md) #

Requests an upload session for the given `clientId`-`serverId`-pair. The
return value is an object of the following format:

```
{
	"clientId": string,
	"sessionUrl": string
}
```

`clientId` is just the `clientId` parameter round-tripped, and `sessionUrl` is
a URL to which a file may be uploaded with a HTTP PUT.

`bitrate` is the bitrate of the file in kbps; it may be 0 if unknown or
irrelevant.

The error value is an object with a property `errorCode`. Known values for it
are:
```
503	upload servers still syncing (transient error, fixable with a retry.)
200	this song is already uploaded
404	the request was rejected
```

# readMetadata [(file, callback)](method.md) #

Reads the metadata of the given `file`.

The `callback` function should take four parameters: `error`, `metadata`,
`image` and `file`. `file` is simply the `file` argument round-tripped.
`metadata` is an object of the following format:

```
{
	"client_id": string,
	"title": string,
	"album": string,
	"album_artist": string,
	"composer": string,
	"genre": string,
	"play_count": number,
	"comment": string,
	"compilation": boolean,
	"beats_per_minute": number,
	"year": number,
	"track_number": number,
	"total_track_count": number,
	"disc_number": number,
	"total_disc_count": number,
	"rating": number (1-6)
}
```

`image` is an array containing the album art data, or `undefined` if the file
contains no album art.

The `client_id` property of the `metadata` argument is the `clientId`
parameter that needs to be passed to [#getUploadSession](#getUploadSession.md).

Due to the limitations of [mp3.js](https://github.com/audiocogs/mp3.js),
`error` is always `null`. [#readMetadata](#readMetadata.md) can still fail, but unfortunately
only by throwing an asynchronous (and thus uncatchable) exception.

This function makes no remote procedure calls and can thus be invoked without
an Internet connection.

In the official music manager, `client_id` is calculated as the MD5 hash of
the input file sans its metadata. `google-musicmanager.js` uses instead the
FNV-1a hash of the whole file.

# upAuth [(token, id, name, callback)](method.md) #

Registers the client with the given `id` as a
[Google Play Music device](https://play.google.com/music/listen#/settings)
under the given `name`. The `id` must be registered before it can be used in
any of the other functions.

The error value is an object with a property `auth_status`, which contains one
of the following error codes:
```
9	MAX_LIMIT_REACHED
10	CLIENT_BOUND_TO_OTHER_ACCOUNT
11	CLIENT_NOT_AUTHORIZED
12	MAX_PER_MACHINE_USERS_EXCEEDED
13	CLIENT_PLEASE_RETRY
14	NOT_SUBSCRIBED
15	INVALID_REQUEST
16	UPGRADE_MUSIC_MANAGER
```
`UPGRADE_MUSIC_MANAGER` is returned when [#upAuth](#upAuth.md) is called with an invalid
`token`.

# uploadMetadatas [(token, id, metadatas, callback)](method.md) #

Uploads an array `metadatas` of song metadata objects (as returned by
[#readMetadata](#readMetadata.md)) to Google servers. Returns an object of the following format:

```
{
	"signed_challenge_info": [{
		"challenge_info": {
			"client_track_id": string,
			"start_millis": number,
			"duration_millis": number
		}
	}, ...],
	"track_sample_response": [{
		"client_track_id": string,
		"server_track_id": string,
		"response_code": number
	}, ...]
}
```

The members of the `signed_challenge_info` array represent song sample requests;
see [#uploadSamples](#uploadSamples.md) for how to handle them.

In the members of the `track_sample_response` array, `client_track_id` is
simply the `client_id` property of a metadata object round-tripped, and
`server_track_id` is the `serverId` parameter that [#getUploadSession](#getUploadSession.md)
expects. `response_code` is one of the following numbers:
```
1	MATCHED
2	UPLOAD_REQUESTED
3	INVALID_SIGNATURE
4	ALREADY_EXISTS
5	TRANSIENT_ERROR
6	PERMANENT_ERROR
7	TRACK_COUNT_LIMIT_REACHED
8	REJECT_STORE_TRACK
9	REJECT_STORE_TRACK_BY_LABEL
10	REJECT_DRM_TRACK
```
A valid `server_track_id` is returned only if `response_code === 2`. The most
common error condition is `ALREADY_EXISTS`, which is returned if a file with
the same `client_track_id` has been uploaded already.

# uploadSamples [(token, id, samples, callback)](method.md) #

Uploads an array of song samples to the Google servers. `samples` is an array
of objects having the following properties:

`track`: The metadata (as returned by [#readMetadata](#readMetadata.md)) of the song of which
this object contains a sample.

`sample`: The actual song sample, a string or array of bytes in 128kbps MP3
format. This may be an empty string if you do not wish to provide a sample.

`signed_challenge_info`: The sample upload request this object is a response
to, as returned by [#uploadMetadatas](#uploadMetadatas.md).

Returns an object similar to [#uploadMetadatas](#uploadMetadatas.md), with the exception that
it has no `signed_challenge_info` property.