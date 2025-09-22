# Extending the Design

1.  **concept** Analytics [ShortUrl]\
    **purpose** Track the number of accesses for a shortUrl\
    **principle** After a shortUrl is registered, viewAnalytics will return its number of accesses

    **state**

        a set of Analytics with
            a shortUrl ShortUrl
            an accessCount number

    **actions**

        initCount(shortUrl: ShortUrl)
            requires: shortUrl is registered and accessCount does not exist
            effects: initializes accessCount to 0

        incrementCount(shortUrl: ShortUrl)
            requires: shortUrl is registered
            effects: increments accessCount by 1

        getCount(shortUrl: ShortUrl): (count: number)
            requires: shortUrl is registered
            effects: returns accessCount of shortUrl

    **concept** Creator [User, ShortUrl]\
    **purpose** Associates shortUrls with their creators\
    **principle** Only the creator of a shortUrl is allowed to view its analytics

    **state**

        a set of Creators with
            an owner User
            a set of ShortUrl

    **actions**

        setOwner(owner: User, shortUrl: ShortUrl)
            requires: owner exists and shortUrl is registered
            effects: set owner as the creator of shortUrl

        getOwner(shortUrl: ShortUrl): (owner: User)
            requires: shortUrl is registered
            effects: returns owner associated with shortUrl

2.  ```
    sync set

        when UrlShortening.register (): (shortUrl)
        then
            Creator.setOwner(user, shortUrl)
            Analytics.initCount(shortUrl: ShortUrl)

    ```

    ```
    sync incrementCount

        when UrlShortening.lookup(shortUrl)
        then Analytics.incrementCount (shortUrl)

    ```

    ```
    sync examineAnalytics

        when Request.viewAnalytics (user, shortUrl)
        where Creator.getOwner (shortUrl) === user
        then Analytics.getCount (shortUrl)
    ```

3.  Feature requests:

- Allowing users to choose their own short URLs

  - I would modify NonceGeneration.generate to include a user-chosen suffix to be returned if it already isn't being used by the context.

- Using the “word as nonce” strategy to generate more memorable short URLs

  - I would use the modified NonceGeneration concept from the third concept question, and keep the _generate_ and _register_ syncs the same.

- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL

  - I would have to modify the Analytics concept to include the targetUrl in the set of Analytics, and modify the action _getCount_ so that it returns the counts of both the given shortUrl and the total count from its associated targetUrl.

- Generate short URLs that are not easily guessed

  - I would modify NonceGeneration to generate longer strings using a larger variety of characters for shortUrls.

- Supporting reporting of analytics to creators of short URLs who have not registered as user

  - This is an undesirable feature since it directly conflicts with the principle of the Creator concept, and it would sacrifice the privacy of a creator's shortUrl analytics if implemented.
