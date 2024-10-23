# YouTube


## PO Token Guide

Proof of Origin (PO) token is a parameter that YouTube requires to be sent with video playback requests from some clients. Without it, requests for the affected clients' format URLs may return HTTP Error 403, or result in your account or IP address being blocked.

This token is generated by BotGuard (Web) / DroidGuard (Android) to attest the requests are coming from a genuine client. This guide will be assuming the use of a PO Token generated by BotGuard, for use with the `web` client.

> For PO Tokens generated by BotGuard, when logged out, the PO Token is bound to a Visitor ID. This Visitor ID is found in the `VISITOR_INFO1_LIVE` cookie, or in the `visitorData` value which is sent with Innertube API requests. When logged in, the PO Token is bound to the account Session ID (part of the Data Sync ID), rather than the Visitor ID.


### Manually acquiring a PO Token from a browser for use when logged out

This process goes through manually obtaining a PO Token generated on YouTube in a web browser, and then manually passing that to yt-dlp via the `po_token` extractor argument.

Steps:

1. Open a browser and go to any video on YouTube Music or YouTube Embedded (e.g. https://www.youtube.com/embed/aqz-KE-bpKQ). **Make sure you are not logged in to any account!**
2. Open the developer console (F12), then go to the "Network" tab and filter by `v1/player`
3. Click the video to play and a player request will appear in the network tab
4. In the request payload JSON, find the PO Token at `serviceIntegrityDimensions.poToken` and save that value
5. Export cookies from the browser
6. Pass the PO Token to yt-dlp using `--extractor-args "youtube:player-client=web,default;po_token=web+PO_TOKEN_VALUE_HERE"` with cookies (`--cookies COOKIES_FILE`)

Addendum:
- You can also get the PO Token from any of the `videoplayback` URLs (it is the `pot` query parameter).
- You can also use `--cookies-from-browser` if that is pointed to the browser session with the same logged-out session you extracted the PO Token from.

#### Passing Visitor Data without cookies

In some cases, you may not want to use cookies and instead pass Visitor Data to use in Innertube API requests. 

> [!WARNING]
> This method is **not recommended** for most cases. It requires skipping webpage requests so that the `VISITOR_INFO1_LIVE` cookie does not interfere. This results in more requests needing to be sent as well as less stable extraction.

You can do this with:

    --extractor-args "youtube:player-client=web,default;player-skip=webpage,configs;po_token=web+PO_TOKEN_VALUE_HERE;visitor_data=VISITOR_DATA_VALUE_HERE"


### Manually acquiring a PO Token from a browser for use when logged in

The process for obtaining a PO Token for use when yt-dlp is logged into an account is similar to the above. The PO Token obtained should work with either cookies or [OAuth](#logging-in-with-oauth).

Steps:

1. Open YouTube Music in a browser, and log in with the user you are using with yt-dlp
2. Open any video on YouTube Music
3. Follow steps 2-4 [above](#manually-acquiring-a-po-token-from-a-browser-for-use-when-logged-out)
4. Pass the PO Token to yt-dlp using `--extractor-args "youtube:player-client=web,default;po_token=web+PO_TOKEN_VALUE_HERE"` with your method of auth (cookies or OAuth)

If you are using cookies, see [Exporting YouTube Cookies](#exporting-youtube-cookies) on how to export cookies without them being invalidated.

> As of writing, it appears that when logged in, YouTube Embedded *does not* bind the PO Token to the Data Sync ID. This will mean PO Tokens from it will not work with the `web` client when using auth, which requires it is bound to the Data Sync ID. We can use YouTube Music instead to fetch the PO Token which does this.


### Other methods of acquiring a PO Token

#### Plugins
- [yt-dlp-get-pot](https://github.com/coletdjnz/yt-dlp-get-pot) by [coletdjnz](https://github.com/coletdjnz)
  - A plugin framework for yt-dlp to support fetching PO Tokens from external providers

## Exporting YouTube cookies

YouTube rotates cookies frequently on open YouTube browser tabs as a security measure.
To export cookies that will remain working with yt-dlp, you will need to export cookies in such a way that they are never rotated. 

One way to do this is through a private browsing/incognito window:
1. Open a new private browsing/incognito window and log into YouTube
2. Open a new tab and **close the YouTube tab**
3. Export cookies from the browser then **close the private browsing/incognito window** so the session is never opened in the browser again.

## Logging in with OAuth

An alternative to using cookies is to log in with OAuth. This method avoids the need to export cookies.

> [!CAUTION]
> Only use one method of authentication at a time. Using both cookies and OAuth at the same time may cause issues.

Enable logging in with OAuth by passing `--username=oauth --password=""` to yt-dlp. You can add this to your [yt-dlp config file](https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#configuration), but this will result in yt-dlp trying to use this username/password combination for all sites. If you only want to apply this for the YouTube extractors, you can add the following to a [.netrc file](https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#authentication-with-netrc):

 ```
machine youtube login oauth password ""
```

You can optionally supply a custom profile name that the account credentials will be saved under with `--username=oauth+MY_PROFILE`. This allows you to switch between multiple accounts. Otherwise, the default profile is used.

A refresh token can optionally be supplied as the password. This may be useful if cache is disabled, or you want to initialize the cache from the refresh token.

### Logging in 

On first run you will be prompted to authorize yt-dlp to access your YouTube account.
> [youtube] oauth: Initializing authorization flow
> 
> [youtube] oauth: To give yt-dlp access to your account, go to  https://www.google.com/device  and enter code  XXX-YYY-ZZZ

Open the link in your browser and enter the code. It will say the request is for YouTube on TV -- this is expected, since we are using the YouTube on TV client for oauth.

The token data is saved in the yt-dlp cache (you can specify a custom cache location with `--cache-dir`).

If you encounter issues, run yt-dlp with verbose logging (`-v`). If you see `oauth: Logged in using profile "XXX"`, then yt-dlp is attempting to use oauth.