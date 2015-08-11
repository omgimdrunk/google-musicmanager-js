The [Client](Client.md) class provides a high-level interface for uploading songs to
Google Play. Its methods return instances of the [Operation](Operation.md) class; see its
documentation for how the return value is to be used. The operations use
certain functions of the lower-level [protocol](protocol.md) namespace internally; see
their documentation for possible `error` event values.

# constructor [(id, token)](method.md) #

Returns a new [Client](Client.md) instance.

`id` is an arbitrary string that serves as the client's identifier. It must be
(near-)unique on Google's side, so pick it reasonably randomly.

`token` is a valid OAuth 2.0 token that is associated with the Google Account
whose Google Play Music library you intend to manage. One of its scopes must
be `https://www.googleapis.com/auth/musicmanager`.

The arguments can be later accessed and set through the `id` and `token`
properties of a [Client](Client.md) instance. Though changing the `id` of a client is
largely pointless, OAuth tokens have a lifetime of roughly an hour, so
updating the `token` becomes necessary for long-lived instances.

# register [(name)](method.md) #

Uses [protocol#upAuth](protocol#upAuth.md) to register the client under the given `name`. This
must be done before the [Client](Client.md) can be used for anything else.

# upload [(files)](method.md) #

Uploads the given `files` to Google Play Music. `files` is a [window#Array](window#Array.md) or
array-like collection of [window#File](window#File.md) objects. Uses [protocol#readMetadata](protocol#readMetadata.md),
[protocol#uploadMetadatas](protocol#uploadMetadatas.md), [protocol#uploadSamples](protocol#uploadSamples.md) and
[protocol#getUploadSession](protocol#getUploadSession.md) internally.

[#upload](#upload.md) emits the following events in addition to the [Operation](Operation.md) defaults:

**metadata-start(file):**
Emitted when the operation starts parsing the metadata of a `file`.

**metadata-end(file, metadata, image):**
Emitted when the operation is done parsing the metadata of a `file`. See
[protocol#readMetadata](protocol#readMetadata.md) for the format of the `metadata` and `image`
parameters.

**metadata-upload():**
Emitted when the operation starts uploading the metadata to Google servers.

**status(file, status):**
Emitted when a response for the metadata of a `file` is received. `status` is
the name of one of the return codes returned by [protocol#uploadMetadatas](protocol#uploadMetadatas.md).

**upload-start(file):**
Emitted when the operation requests an upload session for a `file`.

**progress(file, event):**
Emitted whenever there is progress in the upload of a `file`. `event` is a
[window#ProgressEvent](window#ProgressEvent.md) instance.

**upload-end(file):**
Emitted when a `file` is done uploading.