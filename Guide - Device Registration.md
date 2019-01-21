## Device Registration Guide 

In order to fully interact with the Connected Radio API, you must register a device. In the real world, a device might be an individual radio. 

The same holds true with our API Explorer. In order to do anything, you must first register a device and use the returned `deviceId` to generate your `Authorization` string.

Following are the steps to register a device in the API Explorer:

1. Go to the [API Explorer](https://developer.conrad.ibapis.com/api-explorer)
2. Select `Devices` from the Resource drop-down
3. Enter reasonable values for:
   - Brand Name
   - Device Name
   - Model Name
   - Serial Number
   - Version
   - Manufacturer
   - Latitude
   - Longitude
4. Click the `Headers` tab
5. Add a the Manufacturer ID provided to you by DTS as the value for the `mfgId` header.
6. Create the HMAC-SHA256 hash of the `mfgId`, and add `mfgId,<hash string>` as the value of the `Authorization` header
7. Click `Send Request`

Here's a an animated walkthrough of the process. This walkthrough uses [Code Beautify's HMAC Generator](http://codebeautify.org/hmac-generator).

![Device Request](https://d16co4vs2i1241.cloudfront.net/uploads/tutorial_image/file/682977402162251499/d30d9ccf6efb95055711b92ae89b95a5332101d4a707c57019fde769a4effb43/column_sized_Device-Request.gif)

You can also follow the steps outlined in the [Authentication Documentation](https://developer.conrad.ibapis.com/docs/versions/1.1.3/authentication) to generate a `deviceId` via the command line with `curl`.


