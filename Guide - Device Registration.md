## Device Registration Guide 

In order to fully interact with the Connected Radio API, you must register a device. In the real world, a device might be an individual radio. 

The same holds true with our API Explorer. In order to do anything, you must first register a device and use the returned `deviceId` to generate your `Authorization` string.

Following are the steps to register a device in the API Explorer:

1. Go to the [Devices endpoint reference](/api-reference/devices/postdevices)
2. Scroll to the bottom to show the request generator
3. Click the `Headers` tab
4. Add a the Manufacturer ID provided to you by DTS as the value for the `mfgId` header.
5. Create the HMAC-SHA256 hash of the `mfgId`, and add `mfgId,<hash string>` as the value of the `Authorization` header
6. Enter reasonable values in the JSON object:
   - Brand Name
   - Device Name
   - Model Name
   - Serial Number
   - Version
   - Manufacturer
   - Latitude
   - Longitude
7. Click `Send`

Here's a an animated walkthrough of the process. This walkthrough uses [Code Beautify's HMAC Generator](http://codebeautify.org/hmac-generator).

![Device Request](https://s.cnrd.io/other/device_registration_guide.gif)

You can also follow the steps outlined in the [Authentication Documentation](/#authentication) to generate a `deviceId` via the command line with `curl`.


