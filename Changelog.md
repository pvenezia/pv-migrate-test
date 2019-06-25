### Changes in CONRAD API version 1.1.4

* Documentation consolidation and cleanup 

### Changes in CONRAD API version 1.1.3

* Fixed some cosmetic display issues with escaped quotes
* Added revised C++ and Python AMQP code samples and guide text
* Added `disp` parameter for `/broadcasts/{broadcastId}/live` queries

### Changes in CONRAD API version 1.1.2

* Updated URLs for new development sandbox
* Removed long-deprecated `/feedback` endpoint

### Changes in CONRAD API version 1.1.1

* `/reports` for broadcasts and satellite listening events now contain `volEvents` arrays for volume adjustments. We have also added `errorStatusCode` to streaming report events to capture problems with stream playback.

* We have added `sfOnIconURL`, `sfOffIconURL`, `sfOnIconURLWide`, and `sfOffIconURLWide` as possible attributes within a `/broadcasts` response to facilitate branding of service following/streaming functions. If present, they should be used in the display to indicate service following streaming status.

### Changes in CONRAD API version 1.1

We have made some additions to the CONRAD API since the initial release that are detailed here.

* To address requests for enhanced image support, we have added an `images` array in the `/broadcasts/{broadcastId}/live` response. This array contains images with explicitly stated types if known, such as `artist` for artist image, `logo` for station logo that may or may not be unique for this program, and other possible types as detailed in the documentation. We have added a boolean `default` which designates that image as the default image if no user preference has been made for that particular view, We are deprecating the original `imageURL` and `imageURLHiRes` attributes.

* In order to collect sufficient signal strength data to maintain and improve location-based queries, we have introduced a new path, `/reports/signal` that is used to collect signal strength data on the currently tuned-in broadcast. The reporting interval for this new path is in `/config` as `signalReportInterval`.
 
* We have added several new action types to `/reports/actions` to capture user initiated events from metadata delivered by the API such as SMS messages and navigation events.
 
* As part of that, the `/feedback` endpoint has been deprecated and the capture of thumbs up/down feedback has moved to `/reports/actions` as type `thumbs`.

* In order to address possible interaction issues with certain metadata, we have added `enableThumbs` and `enableShare` as booleans to the `/broadcasts/{broadcastId}/live` result object. This will determine whether the client should show the `Share` and `Rate` or `Feedback` thumbs up/down buttons.
 
* Directly related to the previous item, we have added `enableThumbs` and `enableShare` as booleans to the `/broadcasts` response object under `broadcastData`. This will determine the default action per broadcast if not overridden within the `/broadcasts/{broadcastId}/live` response.

* The `/broadcasts/{broadcastId}/live` path now requires `lat` and `lng` coordinates.
 
* We have explicitly stated the meaning of a `404` to the `/broadcasts/{broadcastId}/live` path. Some broadcasts have time periods where live metadata is not desired. Thus a `404` denotes that there is no live metadata currently available at the time of the query and the client should display the broadcast logo and broadcast metadata and if connected for push notifications, to remain connected for further notifications that may result in fresh data. 
 
* A new attribute called `content` has been added to `adData` in order to permit live metadata to specify the content of an SMS message that could be sent to the specified contact number in the ad via a button displayed on the screen. Multiple buttons could be displayed via multiple instances of an SMS type.

