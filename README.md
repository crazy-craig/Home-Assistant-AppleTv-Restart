## Home Assistant - AppleTv - Restart / Reboot Apple Tv and select something to watch by default
    
## Description

Home Assistant - Reboot Apple Tv's so they return to their main menu every morning and then start live news consistently. This method for Home Assistant will reboot the Apple TV at 5:30am and set it to the live news consistently.

![After Complete Restart Automation](https://github.com/crazy-craig/Restart-Reboot-Apple-Tv/assets/153091268/54a8c4d1-5aa2-4454-abf1-480a72c06f1d)


## Why I did it in this method

The problems with AppleTV are the following:
1. The main problem is you never know where the focus is in an app so using the Left / Right / Up / Down / Select commands fail because you never know where the app when opened will be focused on. So rebooting the Apple Tv puts the apple back to the same place every time.
2. Calling an app directly in a script can give you a blank screen some times that you can not recover from it. Doing it once is fine but switching back and forth confuses the Apple Tv especially if you hit menu while it is loading.
3. So by restarting the Apple Tv will reset all apps to their default focus location and then you can navigate using Left / Right / Up / Down / Select or Call an app with a playlist directly while you are sleeping so it will have time to start properly

⠀
## Main Issues to over come

1. First issue is rebooting the Apple Tv outside of home assistant can cause the Home Assistant Device Integration to be "unavailable" so we will have to check for "unavailable" and reload it if necessary.
2. Issue two really comes down to timing. Since I am rebooting at 5:30am, while I am asleep, I have put in long pauses ({}Waits) so it has plenty of time to recover.

⠀
## My Configuration for this

My configuration is 3 - 4K Gen-2 AppleTv's, running YouTubeTv. I want Apple Tv 1 to start up playing the live news, Apple Tv 2 to start up playing Weather using Fox Weather App, and Apple Tv 3 to start at the main menu.

## This is my process for achieving a smooth reboot

( Currently I have all 3 Apple Tv's plugged into 1 Lutron Plug-In Wall Switch. In the future I plan to separate them if I am pulling an all nighter I can either disable 1 from rebooting or if it is in the middle of playing a movie to not reboot)

## My Script Process - ( script: MediaScript-AppleTv1-Reboot-Sources )

1. Turn off Lutron Switch
2. Wait 1 minute
3. Turn On Lutron Switch
4. Wait a minute
5. Turn on Apple 1
6. Turn on Apple 2
7. Turn on Apple 3
8. Wait 30 seconds
9. Start the news on Apple 1 (This assumes YouTubeTV app is first app on menu
	1. Send Commands to Apple 1 in 1 service
	2. - select
	3. - up
	4. - select
	5. Have a 6 second delay between commands
10. Wait 10 Seconds - for New to come on live with menu up
11. Send 1 more command to get full screen video
	1. Send Commands to Apple 1
	2. - select
12. Start the weather on Apple 2 (I have using FOX Weather because I am a boater)
	1. Open FOX Weather using service: media_player.play_media.

⠀
## Automation for each Apple Tv make sure Apple Tv reloads properly
1. Trigger (When) - state is unavailable
2. Call a service "Home Assistant Core Integration: Reload config entry" for your AppleTv

⠀
## Automation to reload device/integration if needed

```
alias: MediaAuto-AppleTv1-Reload-If-Unavailable
description: ""
trigger:
  - platform: state
    entity_id:
      - media_player.apple_1
    to: unavailable
condition: []
action:
  - service: homeassistant.reload_config_entry
    data: {}
    target:
      entity_id: media_player.apple_1
mode: single
```

## Automation to start my reboot script**

```
alias: MediaAuto-1-Reboot-Apples-and-Set-First-Play
description: ""
trigger:
  - platform: time
    at: "05:30:00"
condition: []
action:
  - service: script.mediascript_appletv1_reboot_sources
    metadata: {}
    data: {}
mode: single
```

## Here is my Script for rebooting 3 Apple Tv's and setting them to New and Weather

```
alias: MediaScript-AppleTv1-Reboot-Sources
sequence:
  - type: turn_off
    device_id: 11184c54311314ab0a630539cfb6025c
    entity_id: e2f9cb3c89314fc13e3b25b7b4c08ce5
    domain: light
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - type: turn_on
    device_id: 11184c54311314ab0a630539cfb6025c
    entity_id: e2f9cb3c89314fc13e3b25b7b4c08ce5
    domain: light
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - type: turn_on
    device_id: 2d2ec7255402b0b763366f5afb44b3a5
    entity_id: 6a21902627b4c2410070dbeb9ab02d46
    domain: remote
    enabled: true
  - type: turn_on
    device_id: 82d8d0a456c33399ae3d33d731da6af1
    entity_id: e6692e9b46afef6040aef50b1965a951
    domain: remote
  - type: turn_on
    device_id: fde0e3d2c718c83f0a04f40414bd0c61
    entity_id: 7cd4497a650c2919c8eeebd6b48003ff
    domain: remote
  - delay:
      hours: 0
      minutes: 0
      seconds: 30
      milliseconds: 0
  - service: remote.send_command
    data:
      delay_secs: 6
      command:
        - select
        - up
        - select
    target:
      device_id:
        - 2d2ec7255402b0b763366f5afb44b3a5
  - service: media_player.play_media
    target:
      entity_id: media_player.apple_2
    data:
      media_content_id: com.fox.weather
      media_content_type: app
    metadata:
      title: FOX Weather
      thumbnail: null
      media_class: app
      children_media_class: null
      navigateIds:
        - {}
        - media_content_type: apps
          media_content_id: apps
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
  - service: remote.send_command
    data:
      delay_secs: 6
      command:
        - select
    target:
      device_id:
        - 82d8d0a456c33399ae3d33d731da6af1
mode: single
```

## RELATED LINKS FOR RESEARCH

This link has information checking status of multiple entities that looks really cool
[https://github.com/home-assistant/core/issues/90166#issuecomment-1482487713](https://github.com/home-assistant/core/issues/90166#issuecomment-1482487713)
