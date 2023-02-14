﻿
# openHASP Sonos group player - multiple plates documentation v1.03

![T3E Sonos media player plate](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image.png) ![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image6_320x480.png)

### Revision:
- **1.03** (2023-02-11)
	 Added 480x800 res. display layout (Sunton 5 and 7" devices)
	 Master speaker entity is now dynamic and can be altered directly from plate

- **1.02** (2023-01-30)
	Removed `zoom` service calls as Custom Component now support image upscaling. Requires openHASP Custom Component from `main` branch commit `b2ed186` or newer

-	**1.01** (2023-01-28)
Added revised layout (**portrait** mode) for 320x480 resolution devices

-	**1.00** (2023-01-25)
Removed all hardcoded media player entities from automations- and config yaml files. Designated master speaker `entity_id` now only need to be entered in `group.sonos_all`
Cleaned up the hardcoded openHASP plate entities as well. Some nine entities are left in total, where only five of these are actually needed.
Configuration prepared to run either multiple 320x480 or 480x480 res. devices without further alterations (besides the device entity_id)

### Prerequisities:
- openHASP 0.6.4-dev
	- commit 7a83367 or newer (January 18th, 2023)
- Home Assistant with Sonos integration installed
	- Tested working with Home Assistant 2022.10.4
- openHASP Custom Component installed in HA
	- `main` branch commit `b2ed186` or newer required (January 30th, 2023)
- Sonos speakers + active Sonos app
	- Tested working with S1 (legacy) and S2 version devices
	- Setup with both S1 & S2 devices will work as well. But S1 devices can't join S2 groups and vice versa 
-   GS-T3E / WT32-SC01 (plus) / Sunton ESP32-48S035 / Sunton ESP32-8048S050 / Sunton ESP32-8048S070 or similar devices **with** PSram
	- Configuration can dynamically handle **both** multiple 320x480, 480x480 or 480x800 resolution displays
	- Devices with different resolutions can even be mixed, as configuration is almost fully dynamic now. Currently only pop-up info is not yet fully dynamic. Pop-up's are at present only dynamic in that sense that first plate device entry in group determines position on display for all device types.
	
With this openHASP Sonos plate configuration, you’ll be able to control single or grouped Sonos speakers from multiple openHASP plates. Plates just have to share identical object mapping (page/object number).

Approach was to make this setup as dynamic as possible. The original design with hardcoded master speaker has now been abandoned and replaced with a fully dynamic group speaker configuration. Now it's possible to both change master- and slave speakers and jump between all active groups or single speakers by changing the Master speaker. 

I also wanted to keep mqtt chatter to a minimum and have mainly used mqtt group topics and not entities for boths triggers and actions. Hence the config is primarily done as automations and not solely as Custom Component configuration. As config is quite elaborate, a multi plate Custom Component configuration for eg. five plates would also have filled more than 4000 lines !

This configuration will only control all speakers volume as a global group. So no individual volume setting is possible from plates. Joining new slave speakers to the group, will also set new speakers volume to match master speakers volume - before it’s joined.

Config template will split all Sonos Favourites in two groups, but will need a little help from you to determine what belongs to which group. All ‘non sources’ (playlists, songs, podcasts etc. from eg. Spotify) should be renamed (in the Sonos app or via web interface) and prefixed with an asterix ‘*’ (No extra spaces, just an asterix)

Sources (radio stations) unfortunately doesn’t follow any strict rules on how to present media meta data. So some radio stations will unfortunately appear on plate with artist/title mixed up. Source in Sonos Favourites can be renamed and suffixed with a ‘|’ pipe symbol (No extra spaces, just a pipe symbol). Templates in config will seek out this pipe symbol and swap title/artist on plate display. Symbol will also be hidden in dropdown list, pop-up info etc.
Special fix for handling danish national radio stations is hardcoded in templates:

- Template search for specific string in `media_title` attribute is done as well. This is needed as quite a few danish broadcasters meta data has both title and artist combined in the `media_title` attribute - only separated with a ' [SPACE]/[SPACE]'. 

Example below:

    media_title: Elephant Woman / Blonde Redhead
 
### Keywords:
- **Source/playlist album image** is displayed as background for the entire page. All other overlaying objects are drawn with varying opacity on top of the source/album image
 - **Next/previous buttons**
	 - **In active playlist:** Select next/previous song
	- **When source is selected:** Skip to next/previous source in list. Will also skip to first source, if at last source or vice versa (loop through)
- **Play/pause pop-up info** overlay (5 sec.) shows info about playlist- or source name, origin (TuneIn (somewhat irregular) and Spotify source detection) and playing status
- **Source/playlist dropdown** list selection via image object button
	- Press and **hold** image object for tabview to appear
	- If you regret to change, then exit via either of the dropdown lists ‘arrow down’ symbols
- **Master speaker selection**
	- ***480 x 800 resolution. layout***
		- Press and **hold** Master speaker button until new button matrix appears
		- Button matrix with all available Sonos speakers shown in **blue**, will replace the slave speaker button matrix
		- Choose new master speaker or exit via **EXIT** button shown in **white**
	
	-  ***320 x 480 and 480 x 480 resolution layouts***
		-  Found on **tabview** mentioned under Source/playlist dropdown
		- All available Sonos speakers are listed in **blue** (current master speaker excluded from list)
		- After selection, the entire dropdown menu and tabview will be closed automatically

- **Slave speaker selection**  
	- ***480 x 800 resolution. layout***
		-   Available slave speakers are listed in the button matrix displayed under the Master speaker button
		-  Already grouped slave speakers are listed in **green**
		- Non grouped, available speakers are listed in **red**
		- Not available speakers will not be shown at all (dynamic group)
		- **Toggle** buttons to **join/unjoin** speakers

	-  ***320 x 480 and 480 x 480 resolution layouts***
		- on **tabview** mentioned under Source/playlist dropdown
		- Already grouped slave speakers are listed in **green**
		- Non grouped, available speakers are listed in **red**
		-  Not available speakers will not be shown at all (dynamic group)
		- **Toggle** buttons to join/unjoin speakers
		- **Exit** selection tab via either of the dropdown lists **‘arrows’**.

- **Shuffle/repeat** buttons and **progress bar** are **disabled** playing sources
- Automation will update progress bar every 5 seconds while playing non sources. Progress bar is shown between artist and title objects
- **Play/pause** button will play/pause (Sonos don’t use stop function. Only when idle)
- **Volume mute** button will mute/unmute whole group of speakers
- **Volume slider** will change volume setting for the whole group of speakers. Volume level indicated on the right - inside the slider. At high volume (above 75 on plate = 'real' volume at 60) indicator will be placed on the left side instead
	- **Bezier algorithm** has been implemented. This makes it much easier to adjust slider at **low level volume** compared to a 'standard' linear slider
		- Slider value 10, equals volume 5. Slider value 20, equals volume 11. Slider value 30, equals volume 18 etc. 
- **Title/artist** are displayed
	- **Album name** is displayed as suffix to title if it's available and not identical with title
	- Supports **swap of title/artist** by suffixing source in Sonos Fovourites with a ‘|‘ symbol
- Primary- and active slave speakers friendly names are also displayed
	- Active speakers are displayed instead of artist, when **tabview is active**
	- On 480 x 800 resolution devices both master- and slave speakers are displayed at all times

#### Config consists of four elements:
HA groups, configuration (both support sensors and CC plate config), automation and the plate jsonl.

### Following needs to be done for config to work !!

**Changes in Sonos app:**
- **Mandatory:** Revise your Sonos Favourites so all playlists, songs, podcast etc. are **prefixed** with an asterix **‘*’** character. Config will **need** this as key to sort favourites into sources / non sources groups
- **Optional:** Revise Sonos sources that need media title/artist to be swapped, by adding a pipe symbol **‘|’** as **suffix** to source name in Sonos Favourites

**Examples:**

    FV:2/44: '*New Music Friday Denmark'
    FV:2/47: '*The Ultimate Hit Mix'
    FV:2/49: Classic FM
    FV:2/54: Classic Rock|
    FV:2/63: DR Nyheder

**Changes in configuration files:**
- Copy all page 2 objects from either `sunton_jsonl`, `gs_t3e.jsonl` or `wt32_01_plus.jsonl` to your existing openHASP jsonl file. I've included my page 0 objects as well in the files, so you can copy the entire page layout if you want.
	- **480 x 800 pixels:** Use `sunton_jsonl` file 
	- **480 x 480 pixels:** use `gs_t3e.jsonl` file
	- **320 x 480 pixels (portrait mode):** use `wt32_01_plus.jsonl` file
- Populate `group.sonos_all` in `groups.yaml` file with **your** typical master speaker `entity_id` only. Remaining speaker entities will be populated dynamically `group.sonos_all_speakers` should be left untouched and unpopulated. It will be populated upon HA start and dynamically upon speaker availability change
- Populate `group.hasp_sonos_devices`in `groups.yaml` file with **your** primary openHASP plate `entity_id`. Group will be dynamically expanded and populated with all available openHASP entities - sorted in alphabetical order
- In both `config.yaml` and `automations.yaml` files, search for `t3e_02` (my plate name) and replace with **your** plate name.
Currently there are nine hardcoded openHASP entities left in the Custom Component part of the`config.yaml` file
	-	The Custom Component configuration **slug entry**
	-	**Four entries** in the custom component **plate configuration**
		- These are needed to be kept unique for each plate configuration as they control local events on the plate being operated
	-	Additional **four entries** in the **page 0 objects** (not mandatory)
	- No entries left in the remaining `config.yaml`or `automations.yaml` files

**Changes in Home Assistant:**

- The Sonos integration has the favorites sensor disabled by default. You need to **enable** `sensor.sonos_favorites` for the configuration to work. Check [HA Sonos integration](https://www.home-assistant.io/integrations/sonos/) to find details on this
- Local HA IP address sensor is needed for this configuration
	- Please add [this](https://www.home-assistant.io/integrations/local_ip/) simple integration in HA to expose needed `sensor.local_ip`
	- If you prefer to use hardcoded static IP address instead, then search for `{{states('sensor.local_ip')}}` in `automations.yaml` and replace with **your** HA local IP address

**General:**

- Merge the four needed configuration parts into your existing files.
	- Note that I’ve used **plate page 2** for this configuration !
- Upload the `openhasp.png` file to plate. Reboot HA and plate(s) and you’re ready ! 🙂

#### openHASP design:

- **UI Theme:** Hasp Dark
- **Primary color:** R:222, G:113, B:16
- **Secondary color:** Default (untouched)


#### ’To do’ list:

- 250 millisecond mandatory delay auto opening the dropdown menus is currently needed. Waiting for an openHASP fix by @fvanroie 
- General config clean-up (templates)
- Revise last bits to make config fully dynamic with multiple devices using different display screen resolutions
- Add Spotify / TuneIn logo’s as small png overlays
- Populate and update plate completely on plate reconnect and/or HA restart.

Suggestions, improvements, error reporting etc. are very welcome ! 🙂

February, 2023 @htvekov

### Below, various images showing the tab views and the pop up info overlay object:

![T3E Source dropdown list](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image1.png)
##### T3E Source dropdown list

![T3E Playlist dropdown list](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image2.png)
##### T3E Playlist dropdown list

![T3E Slave speaker selection buttons](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image3.png)
##### T3E Slave speaker selection buttons

![T3E Slave speaker selection buttons](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image4.png)
##### T3E Pop up info overlay object

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image6_320x480.png)
##### WT32_SC01 Plus 320 x 480 res. layout

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image7_320x480.png)
##### WT32_SC01 Plus Source dropdown list

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/image8_320x480.png)
##### WT32_SC01 Plus Slave speaker selection buttons

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/sunton_openhasp_sonos_01.png)
##### 480 x 800 res. layout

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/sunton_openhasp_sonos_02.png)
##### Pop up info overlay object

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/sunton_openhasp_sonos_03.png)
##### Source dropdown list

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/sunton_openhasp_sonos_04.png)
##### Master speaker button and Master speaker button matrix

![](https://github.com/htvekov/openHASP-Sonos-media-player/blob/main/images/sunton_openhasp_sonos_05.png)
##### Pop up info overlay object





