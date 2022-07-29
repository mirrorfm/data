# Mirror.FM data 🎵

Anyone can add new YouTube channels or Discogs labels as playlists on www.mirror.fm. 100% automatic

## Submitting guidelines

Please only submit channels or labels following these criteria.

### YouTube channels

 - channel tracks seem to be formatted correctly, for example: `Artist – Track`
 - most channel tracks are single tracks, not "full EP", "full album" or a mix
 - have a decent amount of tracks (~50+) 
 - doesn't belong to a single artist
 - doesn't have a matching record label on [discogs.com](https://www.discogs.com/search/?type=label), in which case it should be added as a [Discogs label](https://github.com/mirrorfm/data#discogs-labels)

### Discogs labels

Most should already follow community-driven standards, feel free to add any!

## How to submit a channel or label 

It's quick and easy:

 - edit [discogs-labels.csv](https://github.com/mirrorfm/data/blob/master/discogs-labels.csv) or [youtube-channels.csv](https://github.com/mirrorfm/data/blob/master/youtube-channels.csv) on Github
 - add the channel or label ID, name and link to the end of the file
   - to find the Youtube channel ID I recommend https://commentpicker.com/youtube-channel-id.php
 - create a pull request
 - once approved your Spotify playlist will sync automatically and will be added to the mirror.fm [Spotify profile](https://open.spotify.com/user/xlqeojt6n7on0j7coh9go8ifd?si=StuR-GbuTeCJUSNzHKN5gg)

## How does it work

 - Submissions in the current repo triggers other automations in https://github.com/mirrorfm/mirrorfm/tree/master/functions
 - YouTube track names are cleaned up using https://github.com/mirrorfm/trackfilter/, feel free to [contribute](https://github.com/mirrorfm/trackfilter/blob/master/tests/test_trackfilter.py#L11)
