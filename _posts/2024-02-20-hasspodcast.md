---
layout: post
title: "Home Assistant Podcast"
date: 2024-02-02 21:49:00
categores: [HomeAssistant, Podcast]
---

# Home Assistant Podcast

It's been a minute since I've updated my site. I figure it's a great
time to add in some things I've been up to.

Back in September 2023 Phil and Rohan from the [Home Assistant Podcast](https://hasspodcast.io)
had me on as a guest for [episode 118](https://hasspodcast.io/ha118/).
This started off with a [compeition](https://hasspodcast.io/ha092/)
to send in your coolest automation for the Home Assistant Podcast birthday.
I sent in mine and got runner up!
It was great to be on the podcast, and wonderful to meet Phil and Rohan.

## The Submission

The submission revolved around automatically displaying a Unifi
camera I have mounted in my twins room. When nap time starts I wanted
to have that RTSP stream show up on on the TV downstairs
in the living room. This has gone through some iterations
since my discussion on the podcast.
But the latest [iteration](https://github.com/mikeodr/Home-AssistantConfig/blob/22099520908648782ac57cbe3f97ed89b9ef1b53/packages/nursery.yaml#L634-L662)
looks like so:

{% highlight yaml %}
- alias: Show Nursery Camera on TV automatically
  id: nursery_show_camera_automatically
  trigger:
    - platform: state
      entity_id: input_boolean.nap_mode
      to: "on"
  condition:
    not:
      - condition: state
        entity_id: media_player.apple_tv_living_room
        state: "playing"
  action:
    - service: homeassistant.turn_on
      entity_id: remote.apple_tv_living_room
    - service: remote.send_command
      target:
        entity_id: remote.apple_tv_living_room
      data:
        command: "wakeup"
    - delay:
        seconds: 30
    - event: homekit_faux_camera_event
    - delay:
        seconds: 5
    - service: remote.send_command
      target:
        entity_id: remote.apple_tv_living_room
      data:
        command: "home"
{% endhighlight %}

This [fakes a input](https://github.com/mikeodr/Home-AssistantConfig/blob/22099520908648782ac57cbe3f97ed89b9ef1b53/packages/homekit.yaml#L17-L24)
into HomeKit to
[display a linked camera](https://github.com/mikeodr/Home-AssistantConfig/blob/22099520908648782ac57cbe3f97ed89b9ef1b53/packages/homekit.yaml#L53)[^1]
This forces the camera feed to pop up on the AppleTV display, and with 
some some brittle (but fingers cross Apple doesn't break this on me)
timing events to use the
[AppleTV Remote](https://www.home-assistant.io/integrations/apple_tv/#remote)
input to force it to go full screen.

## Previous Version

Before I picked up an AppleTV an old MiBox S was [running the show](https://github.com/mikeodr/Home-AssistantConfig/blob/1c70a4b2712358e7216481b8bf891073b4ba5555/packages/nursery.yaml#L140-L202).
It would used the Android debugger tool `adb` to send the commands.
This would power on the system, set the set the RTSP stream directly.
More direct and straightforward than the AppleTVs timing method.
Hopefully Apple will allow a direct RTSP stream one day for less brittle timing in the future.
It looked like so:

{% highlight yaml %}
- alias: Toggle TV Monitor
  trigger:
    platform: state
    entity_id: input_boolean.nap_mode
  condition:
    - condition: time
      after: "08:00:00"
  action:
    - wait_template: "{{ not is_state('media_player.living_room_mi_box_s', 'unavailable') }}"
      continue_on_timeout: false
      timeout: "00:20:00"
    - choose:
        # Turning nap on
        - conditions:
            - condition: state
              entity_id: input_boolean.nap_mode
              state: "on"
          sequence:
            - choose:
                - conditions:
                    - condition: state
                      entity_id: media_player.living_room_mi_box_s
                      state: "off"
                  sequence:
                    - service: androidtv.adb_command
                      data:
                        entity_id: media_player.living_room_mi_box_s
                        command: "POWER"
                    - service: androidtv.adb_command
                      data:
                        entity_id: media_player.living_room_mi_box_s
                        command: !secret baby_cam_adb_command
                    - service: androidtv.adb_command
                      data:
                        entity_id: media_player.living_room_mi_box_s
                        command: "adb shell media volume --set 0"
              default:
                - service: androidtv.adb_command
                  data:
                    entity_id: media_player.living_room_mi_box_s
                    command: !secret baby_cam_adb_command
        # Turning nap off
        - conditions:
            - condition: state
              entity_id: input_boolean.nap_mode
              state: "off"
          sequence:
            - wait_template: "{{ not is_state('media_player.living_room_mi_box_s', 'unavailable') }}"
              continue_on_timeout: false
              timeout: "00:20:00"
            - choose:
                - conditions:
                    - condition: template
                      value_template: "{{ not is_state('media_player.living_room_mi_box_s', 'off') }}"
                  sequence:
                    - service: androidtv.adb_command
                      data:
                        entity_id: media_player.living_room_mi_box_s
                        command: "HOME"
                    - service: androidtv.adb_command
                      data:
                        entity_id: media_player.living_room_mi_box_s
                        command: "POWER"
{% endhighlight %}

#### Footnotes

[^1]: [Linked Doorbell Sensor](https://www.home-assistant.io/integrations/homekit/#linked_doorbell_sensor){:target="_blank"}.
