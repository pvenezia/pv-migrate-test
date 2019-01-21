## API Client Flow 

This is the workflow describing how the Connected Radio API (CONRAD) should be used by your client.


1. On startup, a `/devices` registration or `/config` request is made, followed by `/genres` request
2. A `/broadcasts` query is made using GPS coordinates. (Frequency and DAB tuning data queries are also possible)
3. The API responds with a JSON array of stations containing full station data records
4. The client can now create a Station Guide List with logos/album art, callsign, slogan, genre, etc
5. If the user selects a station, that station is tuned in and the station logo/slogan/callsign can be displayed
6. The client then subscribes to message queue for the station to receive live data push notifications, if available. This is indicated by the `hasLiveData` attribute within the broadcast object for that station.
7. The broadcast remains tuned in, push notifications cause dynamic data to display album art, song, program, ad info as it changes
8. If the station list is selected again, a new query is made to `/broadcasts` with the current coordinates and the process repeats.
