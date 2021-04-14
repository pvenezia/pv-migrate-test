# DTS Connected Radio Info Endpoint

The DTS Connected Radio API offers an endpoint that delivers data related to artists, tracks (or recordings), and albums. These may include artist biographies and images, lyrics to songs, information about a particular song or recording, or a particular album.

To use the endpoint, a client must have some sort of descriptive data to use for a query. If the client is querying for information based on local metadata, such as a MP3 tag, this could be an artist name, an artist name and a track name, or an artist and album name. If the client has used another CONRAD API endpoint to retrieve artist and track information, such as via the `/acr` (automatic content recognition) endpoint, or retrieval of live data for a particular broadcast, then the client should use the CONRAD IDs present in that metadata to retrieve further data from the `/info` endpoint. 

The three IDs used to navigate the `/info` endpoint are

* `conradArtistID` - A unique ID assigned by CONRAD for a particular artist, performer, creator, or group.
* `conradTrackId` - A unique ID assigned by CONRAD for a specific audio track or recording. This does not need to be music or a song, though that is the primary type of track. Note that different edits or performances of the same song will each have unique IDs.
* `conradAlbumID` - A unique ID assigned by CONRAD for a particular released album. 

_Note that not all `/info` endpoints are available as of this writing._

## Client Flow with known CONRAD IDs
An example client flow using CONRAD track metadata would look like this:

1. The client retrieves `/live` data for a broadcast, and there is a `conradTrackId` and `conradArtistId` present in the response JSON object.
2. The client uses this `conradTrackId` to retrieve lyrics for the song using the `/info/track/{conradTrackId}/lyrics` endpoint. 
3. The client can use the `conradArtistId` to retrieve the artist's biography and images from `/info/artists/{conradArtistId}/img`, or `/info/artists/{conradArtistId}/bio`.

## Client flow with search
An example client flow using the `/info/search` endpoint would look like this:

1. The client uses a text search with an artist name and track name to the `/info/search` endpoint.
2. The API returns a JSON object with `conradTrackId` and `conradArtistId`, and optionally links to the artist images.
3. The client uses this `conradTrackId` to retrieve lyrics for the song using the `/info/track/{conradTrackId}/lyrics` endpoint. 
4. The client can use the `conradArtistId` to retrieve the artist's biography from `/info/artists/{conradArtistId}/bio`.
