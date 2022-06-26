---
title: Spotify
description: Instructions on how to integrate Spotify into Home Assistant.
ha_category:
  - Media Player
ha_release: 0.43
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_quality_scale: silver
ha_codeowners:
  - '@frenck'
ha_domain: spotify
ha_zeroconf: true
ha_platforms:
  - media_player
ha_integration_type: integration
---

The Spotify media player integration allows you to control [Spotify](https://www.spotify.com/) playback from Home Assistant.

Once an output [source]](#selecting-output-source) is selected you will be able to play content from Spotify using the standard Media Browser UI or `media_player` service.

## Prerequisites

- Spotify account
- (Optional) Spotify developer application configured for Home Assistant (see [below](#create-a-spotify-application))

<div class='note'>
  Spotify integrated media controls (pause, play, next, etc.) require a Premium account.
  If you do not have a Premium account, the frontend will not show the controls.
</div>

{% include integrations/config_flow.md %}

Unless configured otherwise, Home Assistant will use account linking provided by
Nabu Casa for authenticating with Spotify. If this is not working or you don't
want to use it, follow the steps for configuring a [developer application](#create-a-spotify-application)
before configuring Spotify.

<div class='note'>

  If you receive an `INVALID_CLIENT: Invalid redirect URI` error while trying to
  authenticate with your Spotify account, check the Redirect URI in
  the address bar after adding the new integration. Compare this value with the
  Redirect URI defined in the Spotify Developer Portal.

</div>

### Create a Spotify application

- Login to [Spotify Developer](https://developer.spotify.com) via Dashboard.
- Visit the [My Applications](https://developer.spotify.com/my-applications/#!/applications) page.
- Select **Create An App**. Enter any name and description.
- Once your application is created, view it and copy your **Client ID** and **Client Secret**, which are used in the Home Assistant [configuration file below](#configuration).
- Enter the **Edit Settings** dialog of your newly-created application and add a *Redirect URI*:
  `https://my.home-assistant.io/redirect/oauth`.
  Note: Spotify does a case-sensitive match of the fields above, as such ensure the Redirect URI is all lower case.
- Click **Save** after adding the URI.

{% details "I have manually disabled My Home Assistant" %}

If you don't have [My Home Assistant](/integrations/my) on your installation,
you can use `<HOME_ASSISTANT_URL>/auth/external/callback` as the redirect URI
instead.

The `<HOME_ASSISTANT_URL>` must be the same as used during the configuration/
authentication process.

Internal examples: `http://192.168.0.2:8123/auth/external/callback`, `http://homeassistant.local:8123/auth/external/callback`."

{% enddetails %}

See [Application Credentials](/integrations/application_credentials) for instructions on how to configure your *Client ID* and *Client Secret*.

## Using multiple Spotify accounts

This integration supports multiple Spotify accounts at once. You don't need to
create another Spotify application in the Spotify Developer Portal and no
modification to the `configuration.yaml` file is needed. Multiple Spotify
accounts can be linked to a _single_ Spotify application. You will have to add those accounts into the **Users and Access** section of your application in the Spotify Developer Portal.

To add an additional Spotify account to Home Assistant, go to the Spotify website and log out, then repeat _only_ the steps
in the [Configuration](#configuration) section.

## Playing Spotify playlists

You can send playlists to Spotify using the `"media_content_type": "playlist"`, which is part of the
[media_player.play_media](/integrations/media_player/#service-media_playerplay_media) service, for example:

```yaml
# Example script to play playlist
script:
  play_jazz_guitar:
    sequence:
      - service: media_player.play_media
        target:
          entity_id: media_player.spotify
        data:
          media_content_id: "https://open.spotify.com/playlist/5xddIVAtLrZKtt4YGLM1SQ?si=YcvRqaKNTxOi043Qn4LYkg"
          media_content_type: playlist
```

The `media_content_id` value can be obtained from the Spotify desktop app by clicking on the more options ("...") next to the album art picture, selecting "Share" and then "Copy Spotify URI" or "Copy Playlist Link" (also available in the Spotify phone and web app).

The `media_content_id` can be a URL (e.g. `https://open.spotify.com/playlist/37i9dQZF1DWVFeEut75IAL`) or the Spotify URI string (e.g. `spotify:playlist:37i9dQZF1DWVFeEut75IAL`). (play_media)[https://www.home-assistant.io/integrations/media_player/#service-media_playerplay_media] requires the correct corresponding `media_content_type`.

## Selecting output source

To play media Spotify needs a device selected for audio output known as the `source`. Currently, this integration cannot initiate playback to a device not already available from the Spotify API. You need to first use Spotify on your smart speaker or phone to populate the source list.

```yaml
# Example code to select an AV receiver as the output device
service: media_player.select_source
entity_id: media_player.spotify
data:
  source: "Denon AVR-X2000"
```

The device name must be known to Spotify and therefore is often different from names given to entities in Home Assistant.

The source list of available devices can be found in the Details section of the Spotify Media Player Control and the `source_list` attribute in the Developer tools States or templates using:

```jinja
{{ state_attr('media_player.spotify', 'source_list') }}
```

<div class='note'>
  A known limitation with Spotify is that devices disappear from the source list over time. Many Spotify Connect devices tend to persist, other devices such as Amazon Alexa speakers may only be remembered for a while, and phones and Google Cast devices disappear as soon as playback stops.
</div>

## Unsupported Devices

- **Sonos**: Although Sonos is a Spotify Connect device, it is not supported by the official Spotify API.
