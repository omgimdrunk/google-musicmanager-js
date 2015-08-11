# Introduction #

`google-musicmanager.js` is a clientside JavaScript port of the [MusicManager interface](http://unofficial-google-music-api.readthedocs.org/en/latest/reference/musicmanager.html) of [Simon Weber's unofficial Google Music API](https://github.com/simon-weber/Unofficial-Google-Music-API). It allows uploading songs to Google Play Music libraries in browser JavaScript. Though only tested in Firefox to date, it should work in all standards-compliant browsers.

**Note that use of this library (and the unofficial Google Music API in general) likely constitutes a violation of Google's Terms of Service: ["For example, don’t interfere with our Services or try to access them using a method other than the interface and the instructions that we provide."](https://www.google.com/intl/en/policies/terms/) Use at your own risk.**

For an example project using this library, see the [Google Play Music Uploader](https://code.google.com/p/google-musicmanager-user-js) userscript.

# Requirements #

`google-musicmanager.js` depends on quite a few other JavaScript libraries:

  * [aurora.js ^0.4.2](https://github.com/audiocogs/aurora.js)
  * [mp3.js ^0.1.0](https://github.com/audiocogs/mp3.js)
  * [Long.js ^2.0.0](https://github.com/dcodeIO/Long.js)
  * [ByteBuffer.js ^3.2.0](https://github.com/dcodeIO/ByteBuffer.js)
  * [ProtoBuf.js ^3.3.0](https://github.com/dcodeIO/ProtoBuf.js)

It also needs to be able to make cross-domain requests to the following address spaces:

  * https://android.clients.google.com/upsj/*
  * https://uploadsj.clients.google.com/uploadsj/rupio
  * https://music.google.com/music/*

You also need a valid Google OAuth 2.0 token with a scope of `https://www.googleapis.com/auth/musicmanager`; see [here](https://developers.google.com/accounts/docs/OAuth2) for instructions on how to obtain one. You may set up a [Google Cloud Console](https://console.developers.google.com) project to gain the necessary credentials, or just borrow the ones belonging to Google's official Music Manager:
```
client_id:     652850857958
client_secret: ji1rklciNp2bfsFJnEH_i6al
redirect_uri:  urn:ietf:wg:oauth:2.0:oob
```
Note, however, that these only work with the [Using OAuth 2.0 for Installed Applications](https://developers.google.com/accounts/docs/OAuth2InstalledApp) workflow. If you opt to register a new Cloud Console project instead, note that the Music Manager API does not need to be activated in the **APIs & auth** section (in fact, it's not even listed there.)

# Usage #

```
<script src="Long.js"></script>
<script src="ByteBufferAB.js"></script>
<script src="ProtoBuf.js"></script>
<script src="aurora.js"></script>
<script src="mp3.js"></script>
<script src="google-musicmanager.js"></script>

<script>

var client = new gmusicmanager.Client('client id', 'OAuth2.0 token');
var register = client.register('client name');
register.on('error', failure_callback);
register.on('end', success_callback);
register.start();

// with a successfully registered client
var upload = client.upload(filelist);
upload.on('error', failure_callback);
upload.on('progress', progress_callback);
upload.on('end', success_callback);
upload.start();

</script>
```

See the [API reference](INTRO.md) for details.

# Bugs #

Google Play Music requires all songs to be uploaded in MP3 format. Until someone writes a pure-Javascript MP3 transcoding library, `google-musicmanager.js` can only upload MP3 files. For much the same reason, `google-musicmanager.js` can't match songs, as it has no way of extracting 128kbps samples from its input files.

`google-musicmanager.js` relies on [mp3.js](https://github.com/audiocogs/mp3.js) to do its MP3 parsing, and it's not perfect. Notably, if the file contains encoding errors or too much metadata at its start, `mp3.js` throws an asynchronous (and thus uncatchable) `AV.UnderflowError`. `mp3.js` also reports the bitrate of VBR files as simply the bitrate of the first frame it encounters, so `google-musicmanager.js` opts not to report the bitrate at all in order not to lie to the user.

`google-musicmanager.js` can parse album art, but can't upload it.

`google-musicmanager.js` does not offer a high-level interface for listing or downloading songs. Though both are possible through the low-level interfaces, the latter is particularly bothersome: a request for a download link is rejected if it is sent with the `sjsaid` cookie, and `XMLHttpRequest` defaults to sending all cookies with every request. Browser-specific hacks are needed to suppress this behavior.