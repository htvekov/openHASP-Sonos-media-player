---

# openHASP Sonos group player - multiple plates documentation v1.1

---

![T3E Sonos media player plate](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/image.png)


### Prerequisities:
- openHASP 0.6.4-dev
	- commit 7a83367 or newer (January 18th, 2023)
- Home Assistant with Sonos integration installed
	- Tested working with Home Assistant 2022.10.4
- openHASP Custom Component installed in HA
	- v0.6.6 or newer
- Sonos speakers + active Sonos app
	- Tested working with S1 (legacy) and S2 version
-   GS-T3E plate or similar (This config is formatted for 480x480 res. displays)

With this openHASP Sonos plate configuration, you’ll be able to control single or grouped Sonos speakers from multiple openHASP plates. Plates just have to share identical object mapping (page/object number).

Approach was to make this as dynamic as possible. But as HA needs a hardcoded trigger entity (templating not allowed), and to make things less complicated, a designated permanent master speaker will have to be chosen and hardcoded for the config to work.

I also wanted to keep mqtt chatter to a minimum and have mainly used mqtt group topics and not entities for boths triggers and actions. Hence the config is primarily done as automations and not solely as Custom Component configuration. As config is quite elaborate, a multi plate Custom Component configuration for eg. five plates would also have filled more than 4000 lines !

This configuration will only control all speakers volume as a global group. So no individual volume setting is possible from plates. Joining new slave speakers to the group, will also set speakers volume to match master speakers volume - before it’s joined.

Config template will split all Sonos Favourites in two groups, but will need a little help from you to determine what belongs to which group. All ‘non sources’ (playlists, songs, podcasts etc. from eg. Spotify) should be renamed (in the Sonos app or via web interface) and prefixed with an asterix ‘*’ (No extra spaces, just an asterix)

Sources (radio stations) unfortunately doesn’t follow any strict rules on how to present media meta data. So some radio stations will unfortunately appear on plate with artist/title mixed up. Source in Sonos Favourites can be renamed and suffixed with an ‘|’ pipe symbol (No extra spaces, just a pipe symbol). Templates in config will seek out this pipe symbol and swap title/artist on plate display. Symbol will also be hidden in dropdown list, pop-up info etc. Special fix for handling danish national radio stations is hardcoded in templates.

### Keywords:
- **Source/playlist album image** is displayed as background for the entire page. All other overlaying objects are drawn with varying opacity on top of the source/album image
	- **Zoom** function used to e.g. resize TuneIn’s source images, as they are only 145 x 145 pixels max.
 - **Next/previous buttons**
	 - **In active playlist:** Select next/previous song
	- **When source is selected:** Skip to next/previous source in list. Will also skip to first source, if at last source or vice versa (loop through)
- **Play/pause pop-up info** overlay (5 sec.) shows info about playlist- or source name, origin (TuneIn (somewhat irregular) and Spotify source detection) and playing status
- **Source/playlist dropdown** list selection via image object button
	- Press and **hold** image object for tabview to appear
	- If you regret change, exit via either of the dropdown lists ‘arrow down’ symbol
- **Slave speaker selection** on above mentioned tabview as well
	- Already grouped slave speakers are green. Non grouped, available speakers are red
	- Toggle buttons to join/unjoin speakers
	- Exit selection tab via dropdown lists ‘arrows’ as well. Not available speakers will not be shown at all (dynamic group)
- **Shuffle/repeat** buttons and **progress bar** are disabled playing sources
- Automation will update progress bar every 5 seconds while playing non sources. Progress bar is shown between artist and title objects
- **Play/pause** button will play/pause (Sonos don’t use stop function. Only when idle)
- **Volume mute** button will mute/unmute whole group of speakers
- **Volume slider** will change volume setting for the whole group of speakers. Volume level indicated on the right - inside the slider. At high volume (above 80 on plate) indicator will be placed on the left side instead
	- **Bezier algorithm** has been implemented. This makes it much easier to adjust slider at **low level volume** compared to a 'standard' linear slider
		- Slider value 10, equals volume 5. Slider value 20, equals volume 11. Sliver value 30, equals volume 18 etc. 
- **Title/artist** are displayed
	- **Album name** is displayed as suffix to title if it's available and not identical with title
	- Supports **swap of title/artist** by suffixing source in Sonos Fovourites with a ‘|‘ symbol
- Primary/active slave speakers friendly names are also displayed
	- Active speakers are displayed instead of artist, when tabview is active

#### Config consists of four elements:
HA groups, config (both support sensors and CC plate config), automation and the plate jsonl.

### Following needs to be done for config to work !!

**Changes in Sonos app:**
- **Mandatory:** Revise your Sonos Favourites so all playlists, songs, podcast etc. are prefixed with an asterix ‘*’ character. Config will **need** this as key to sort favourites into sources / non sources groups
- **Optional:** Revise Sonos sources that need swap of media title/artist as well, by adding a pipe symbol ‘|’ as suffix to source name in Sonos Favourites

**Changes in configuration files:**
- Copy all page 2 objects from `sonos.jsonl` to your existing openHASP jsonl file. I've included my page 0 objects as well in the file, so you can copy the entire page layout if you want.  
- Search for `media_player.kokken` in both `automations.yaml` and `config.yaml` and replace with your master speakers `entity_id`. Remember also to change within the templates, where `media_player.kokken` is just a part of the full string
- Populate `group.sonos_all` with your designated master speaker `entity_id` only. Remaining entities will be populated dynamically. `group.sonos_all_speakers` will also be populated as well upon HA start and dynamically upon speaker availability change
- In both `config.yaml` and `automations.yaml` files, search for `t3e_02` and replace with your plate name. If you've multiple Sonos media player plates, then add all plate entities to the automations where multiple `entity_id`'s are used to trigger object events upon plate beboot. Search for `#- openhasp.wt32_01` in `automations.yaml` in order to find current automations with multiple `entity_id` triggers
- In `automations.yaml` search for `xxx.xxx.x.xx` and replace with your Home Assistant’s IP address
- Merge the four needed configuration parts into your existing files. Note that I’ve used **plate page 2** for this configuration !
- Upload the `openhasp.png` file to plate. Reboot HA and plate(s) and you’re ready ! 🙂

#### openHASP design:

- **UI Theme:** Hasp Dark
- **Primary color:** R:222, G:113, B:16
- **Secondary color:** Default (untouched)


#### ’To do’ list:

- One second mandatory delay auto opening the dropdown menus are currently needed. Waiting for an openHASP fix by @fvanroie 
- General config clean-up
- Add Spotify / TuneIn logo’s as small png overlays
- Populate and update plate completely on plate reconnect and/or HA restart.

Suggestions, improvements, error reporting etc. are very welcome ! 🙂

January, 2023 @htvekov



### Below, various images showing the tab views and the pop up info overlay object:

![Source dropdown list](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/image1.png)
##### Source dropdown list

![Playlist dropdown list](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/image2.png)
##### Playlist dropdown list

![Slave speaker selection buttons](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/image3.png)
##### Slave speaker selection buttons

![Slave speaker selection buttons](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/image4.png)
##### Pop up info overlay object


