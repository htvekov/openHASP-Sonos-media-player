# openHASP-Sonos-media-player
openHASP Sonos group media player - Multiple plate configuration support

openHASP Sonos group player - multiple plates
documentation
Requirements:
- openHASP 0.6.4-dev 7a83367 or newer (February 2nd, 2022)
- openHASP custom component installed
- HA with Sonos integration installed - Tested working with Home Assistant 2022.10.4
- Sonos speakers + active Sonos app. Tested working with S1 (legacy) & S2 versions.
- GS-T3E plate or similar (my config is formatted for 480x480 res. displays)
With this openHASP Sonos plate configuration, you’ll be able to control single or grouped
Sonos speakers from multiple plates. Plates just have to share identical object mapping
(page/object number).
Approach was to make this as dynamic as possible. But as HA needs a hardcoded trigger
entity (templating not allowed), and to make things less complicated, a designated
permanent master speaker will have to be chosen and hardcoded for the config to work.
I also wanted to keep mqtt chatter to a minimum and have mainly used mqtt group topics
and not entities for boths triggers and actions. Hence the config is primarily done as
automations and not solely as CC configuration. As config is quite elaborate, a multi plate
CC configuration for eg. five plates would also have filled more than 4000 lines !!
This config will only control all speakers volume as a global group. So no individual volume
setting is possible from plates. Joining new slave speakers to the group, will also set
speakers volume to match master speakers volume - before it’s joined :-)
Config template will split all Sonos Favourites in two groups, but will need a little help from
you to determine what belongs to which group. All ‘non sources’ (playlists, songs, podcasts
etc. from eg. Spotify) should be renamed (in Sonos app or via web interface) and prefixed
with an asterix ‘*’ (No extra spaces, just an asterix)
Sources (radio stations) unfortunately doesn’t follow any strict rules on how to present media
meta data. So some radio stations will unfortunately appear on plate with artist/title mixed
up. Source in Sonos Favourites can be renamed and suffixed with an ‘|’ pipe symbol (No
extra spaces, just a pipe symbol). Templates in config will seek out this pipe symbol and
swap title/artist on plate display. Symbol will also be hidden in dropdown list, pop-up info etc.
Special fix for handling danish national radio stations is hardcoded in templates.
Keywords:
- Source/playlist album image displayed. Zoom function used to resize TuneIn’s source
images, as they are only 145 x 145 pixels max.
- In active playlist: Next/previous button will select next/previous song
- When source is selected: Next/previous button will skip to next/previous source in
list. Will also skip to first source, if at last source or vice versa (loop through)
- Play/pause pop-up info overlay (5 sec.) shows info about playlist- or source name,
origin (TuneIn (somewhat irregular) and Spotify source detection) and playing status
- Source/playlist dropdown lists selection via image object button (press and hold
image for tabview to appear). If you regret change, exit via either of the dropdown
lists ‘arrow down’
- Slave speaker selection on above mentioned tabview as well. Already grouped slave
speakers are green, non grouped, available speakers are red. Toggle buttons to
join/unjoin speakers. Exit selection tab via dropdown lists ‘arrows’ as well. Not
available speakers will not be shown at all (dynamic group)
- Shuffle/repeat buttons and progress bar are disabled playing sources
- Automation will update progress bar every 5 seconds while playing non sources
- Play/pause button will play/pause (Sonos don’t use stop function. Only when idle)
- Volume mute button will mute/unmute whole group of speakers
- Volume slider will change volume for whole group of speakers. Volume level
indicated on the right - inside the slider
- Title/artist and album name (if available and not identical with title) displayed.
Supports swap of title/artist by suffixing source in Sonos Fovourites with a ‘| symbol
- Hardcoded primary- and selected, active slave speakers friendly names displayed as
well
Config consists of four elements: HA groups, config (both support sensors and CC plate
config), automation and the plate jsonl.
Following needs to be done for config to work !!
- Revise your Sonos Favourites so all playlists, songs, podcast etc. are prefixed with
an asterix ‘*’ character. Config will need this as key to sort favourites into sources /
non sources groups
- Revise sources that need swap of media title/artist as well, by adding a pipe symbol
‘|’ as suffix to source name in Sonos Favourites.
- Search for media_player.kokken in all files and replace with your master speakers
entity_id. Remember to also change within the templates, where entity_id is just a
part of the full string !)
- Populate group.sonos_all with your designated master speaker entity_id only.
Remaining entities will be populated dynamically. group.sonos_all_speakers will
also be completely populated as well upon HA start and dynamically upon speaker
availability change.
- In config and automation files, search for wt32_0 and replace with your plate name
or names (if you’ve multiple plates). If you’ve only one plate, then remove all
wt32_02 entities in config files. In config files I’ve both wt32_01 and wt32_02
entities.
- In automation.yaml search for xxx.xxx.x.xx and replace with your Home
Assistant’s IP address
- Merge the four needed configuration bits into your existing files. Note that I’ve used
plate page 2 for this configuration !
- Upload the openhasp.png file to plate. Reboot HA and plate(s) and you’re ready ! :-)
On the ‘to do’ list:
- General config clean-up
- Add Spotify / TuneIn logo’s as small png overlays
- Populate and update plate completely on plate reconnect and/or HA restart
Suggestions, improvements, error reporting etc. are very welcome !!! 🙂
February, 2020 @htvekov
