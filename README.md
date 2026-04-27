###### Memo

# JavaScript fetch: CORS errors

[CORS](#mdn-cors) errors are handled differently in different JavaScript runtimes and environments, resulting in pitfalls for catching them.

Note that the following just shortly clarifies what to be aware of, but does not offer a ready-to-use solution.


## Node.js vs. Browsers

Node's "fetch" implementation [via undici](#node-fetch-based-on-undici) does explicitly [not implement CORS](#node-undici-no-cors) since it is targeting "server-side environments where CORS restrictions are typically unnecessary".

This is a problem if you develop in a local Node.js environment without problems and the code crashes in browsers due to CORS.


## Browsers: console vs. JavaScript

Browsers give very technically, but rich information on CORS errors **in their DevTools consoles** like

  * Chrome
    ```console
      Access to fetch at 'https://example.com/' from origin '...'
      has been blocked by CORS policy: No 'Access-Control-Allow-Origin'
      header is present on the requested resource.
      GET https://example.com/ net::ERR_FAILED 200 (OK)
    ```
  * Firefox
    ```console
      Cross-Origin Request Blocked: The Same Origin Policy disallows
      reading the remote resource at https://example.com/. (Reason:
      CORS header ‘Access-Control-Allow-Origin’ missing).
      Status code: 200.
    ```

but are [**intentionally "opaque" on what is catchable by JavaScript means**](#cors-error-security-no-javascript), i.e. just a TypeError with a message like

  * Chrome
    ```console
      TypeError: Failed to fetch
    ```
  * Firefox
    ```console
      TypeError: NetworkError when attempting to fetch resource.
    ```

i.e. **for JavaScript a fetch with CORS problems just results in an Error object of type "TypeError", but without any [Response object](#mdn-fetch-response)** that could deliver an HTTP status, headers, contents or anything.

This is a problem since you may not be able even to prove the existence of a target URL.


## Reference code

Use this to compare behavior of runtimes:
  ```js
    const fetch_test = ( url ) => {
      let fetch_res = { url: url, status: 'N/A', statusText: 'N/A', error: '(none)' };
      fetch( url )
      .then( ( res ) => {
        fetch_res.status = res.status;
        fetch_res.statusText = res.statusText;
      })
      .catch( err => {
        fetch_res.error = err;
      })
      .finally( () => {
        Object.keys( fetch_res ).forEach( value =>
          console.log( value + ':', fetch_res[ value ] )
        );
      })
    }
    
    // HTTPS target
    fetch_test( 'https://example.com/' )

    // HTTP target
    fetch_test( 'http://neverssl.com/' )
  ```



## References

###### cors-error-security-no-javascript
  * [MDN: "For security reasons, specifics about what went wrong with a CORS request are not available to JavaScript"](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS/Errors#:~:text=For%20security%20reasons%2C%20specifics,%20JavaScript)

###### mdn-cors
  * [MDN: Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)

###### mdn-fetch-response
  * [MDN: fetch: Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)

###### node-fetch-based-on-undici
  * [NodeNode.js Documentation: fetch](https://nodejs.org/docs/latest/api/globals.html#:~:text=The%20implementation%20is%20based%20upon%20undici)

###### node-undici-no-cors
  * [Node.js Undici: CORS](https://undici.nodejs.org/#/?id=cors)