## Authentication
The Connected Radio API is an authenticated API. This means that every call to the API must contain valid authentication credentials to retrieve or submit data. 

Prior to using the API, you will be provided with two data elements:
- `mfgId` - Manufacturer Id which uniquely identifies you.
- `mfgSecret` - A key secret used to sign all requests (but never transmitted). Connected Radio's API will use this key to validate the signature used in all requests.

For the purposes of this sandbox, there are two components to authentication:
1. While in the API Explorer, you must populate a `mfgId` header with your assigned `mfgId` for each request. 
2. You'll need to register a `device` using the `/devices` endpoint and use the returned `deviceId` for all subsequent requests. That `deviceId` typically lasts for the life of the device and, therefore, **must be stored in non-volatile memory on the device**. This ID is a string that can be between 6 and 32 characters in length.

### deviceId
For your first request to the API, you'll need to request a `deviceId`. To do this, you must construct an `Authorization` header using the `mfgId` and `mfgSecret` as follows:

```
mfgId: {mfgId in plain-text}
Authorization: {mfgId},{SHA256 HMAC(mfgSecret, mfgId)}
```

This `Authorization` header construct is unique to the `/devices` path, because the device does not have an assigned `deviceID` yet. 

Now run the query:

```
curl --request POST \
  --url https://api.sandbox.cnrd.io/v1/devices \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --header 'Authorization: {mfgId},{hash}' \
  --header 'mfgId: {mfgId}' \
  --data '{
    "lat":43.094,
    "lng":-71.730,
    "mfg":"mfg name",
    "brand":"brand name",
    "device":"device name",
    "model":"model name",
    "serial":"1234",
    "version":"1.0"
    }'
```

You'll be returned a unique `deviceId`. See the [Devices documentation](/api-reference/devices/postdevices) for specific parameters. You can view [the visual guide on device registration](/guides/device-registration) for a walkthrough of this process.

For subsequent requests to the API, you'll provide `deviceId` and `Authorization` headers as follows:

```
mfgId: {mfgId in plain-text}
Authorization: {deviceId},{SHA256 HMAC(mfgSecret, deviceId)}
```
The `mfgSecret`, `mfgId`, and `deviceId` string encodings are expected to be in UTF-8. Different encoding will result in different hash values. The hash is hex digest, truncated to 24 characters. A good resource to quickly generate these hashes for testing is at [Code Beautify](http://codebeautify.org/hmac-generator).

## Putting it all together
Here's a sample request to the `/broadcasts` endpoint. Let's pretend that our `mfgId` is `D0K101`, our `deviceId` is `aBc8deFG9` and our `mfgSecret` is `secret`:

```
curl --request GET \
  --url https://api.sandbox.cnrd.io/v1/broadcasts/1234/ \
  --header 'Accept: application/json' \
  --header 'Authorization: aBc8deFG9,b212d5b76e23db5794cb56b9' \
  --header 'mfgId: D0K101'
```

This request will return the relevant data for `broadcastId` 1234. 

