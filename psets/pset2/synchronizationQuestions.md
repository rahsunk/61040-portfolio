# Synchronization Questions

1. In the first sync, the action invocation is NonceGeneration.generate, which just returns a string not already used by the given context, so the targetURL is not relevant here.

   In the second sync, the targetURL is included because this sync has the UrlShortening.register action invocation, which is responsible for creating the mapping from the shortURL to the targetURL such that when you look up the shortURL, you get the targetURL.

2. There are cases where the argument/result names and variable names aren't the same, particularly when there are multiple parameters of the same argument/result name. In this case, well-named variables help distinguish between the parameters.

3. The first two syncs take place before a shortURL to targetURL mapping is created, that is, when a request to shorten a URL is made.

   The third sync is only invoked when a shortURL to targetURL mapping is successfully created through the UrlShortening.register action, so the _request_ action is redundant here.

4. If domain name is fixed, then there is no need for the shortUrlBase and context arguments:

   ```
   sync generate
       when Request.shortenUrl ()
       then NonceGeneration.generate ()
   ```

   ```
   sync register
       when
           Request.shortenUrl (targetUrl)
           NonceGeneration.generate (): (nonce)
       then UrlShortening.register (shortUrlSuffix: nonce, targetUrl)
   ```

   ```
   sync setExpiry (same as before)
       when UrlShortening.register (): (shortUrl)
       then ExpiringResource.setExpiry (resource: shortUrl, seconds: 3600)
   ```

5. ```
   sync expireResource
       when ExpiringResource.expireResource () : (resource)
       then UrlShortening.delete (shortUrl: resource)
   ```
