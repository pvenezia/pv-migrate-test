## Welcome
Welcome to the DTS Connected Radio API Documentation Portal. We're glad you're here. This API is where you'll find detailed metadata on radio broadcasts all over the world.

Connected Radio is a global solution, but we do not offer data on all countries on Earth in this developer sandbox. For development and testing, it's best to work with data from the United Kingdom, United States, Germany, France, Japan, and Australia.

This guide provides all the information necessary to access Connected Radio including the full API spec. You'll even be able to use a sandboxed version of the API to retrieve data and test your code.

Each endpoint described in the [API Reference](/api-reference) has a built-in query tool at the bottom where you can set your authentication headers as environment variables and begin querying the API right away. 

Note that the keys you use to access the API through the sandbox will not work in production.

**Note:** We have recently released v1.2 of the CONRAD API, including two new endpoints, `/ipc` for IP-based content such as podcasts and Internet streams, and `/localevents` for information on live concerts, shows, and general events local to a given area.


See the [API Guides](/guides) and [API Reference](/api-reference) for more information on these new endpoints.

If you have any questions, please contact [connected.radio@dts.com](mailto:connected.radio@dts.com).

## Steps to Connected Radio API Mastery

1. Take a look at our [API Guides](/guides) for API flow diagrams, authentication walkthroughs, AMQP push notification information, and other how-to guides. **The [AMQP push notification guide](/guides/amqp-push-notifications-guide) is a must-read**, and will save time on implementation.
2. Read the [authentication guide](#authentication) to understand how you can connect your app to our sandbox.
3. Give the API a go with the [API Reference, including live query tools](/api-reference).
4. Roll up your sleeves and read the full API spec.
5. If you use the [Postman API development tools](https://www.getpostman.com/), download the [Postman collection for the DTS Connected Radio API](https://s.cnrd.io/other/DTS_Connected_Radio_API.postman_collection-1.2.zip)
6. **Note that in these docs the example API JSON responses can be shown or hidden by clicking the Example section**.
7. Start coding!

## API Conventions

API interaction is available via SSL/TLS encryption only, and the use of HTTP compression is highly encouraged to reduce data consumption. 

The developer's sandbox API base URL is `https://api.sandbox.cnrd.io`.

If a key specified in the API spec is missing from a response, the value should be assumed `null`. All booleans will have default values specified in the documentation if they are not present in a response.

