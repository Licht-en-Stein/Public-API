#EyeEm API V3
***

This is a living document to describe the general API changes, as well as list endpoints that need to be broken up, removed, updated, etc. Some points in here are suggestions or open questions.

For reference, check out the list of [API Endpoints being actively called from clients] (api-used-endpoints.md)

##General Remarks
***

- API Versioning. Introduced in V2.1.0 (Requests should now always contain the X-Api-Version header).
- URL/Path Changes:
  - main API path is now https://api.eyeem.com/v3. API will reject non-https requests
  - images are now found under http://cdn.eyeem.com
- API responses should include a "meta" object by default. That object contains the following:
  - a notice if the endpoint is deprecated
  - http response code (optional)
  - human-readable error message
  - notice for native clients (new app version, message from eyeem, etc)
  - rate-limit information
  - human-readable message (optionally w/ link) to popup for native clients [ex: new android version available, update now!]
- introduce a maintenance mode for API
- always return a resource representation to UPDATE/CREATE requests
- introduce rate-limiting for non-native clients
- obfuscate user/photo ids

##Changes to Parameters (AKA params gone wild) 
***

Request parameters need to be standardized. V3 will introduce the following (optional) parameter:
- `sort`: for album photos, this could be something like `chronological`, or `top`. General rule is that this returns the same content, just ordered differently.
- `filter`: returns a subset of the resultset. The value is typically comma-separated, and the results are the union of all filters. For discover, `filter=trending,nearby,favorited` returns all discover items that match any/all of the three filters.
- `type`: for endpoints that span multiple result sets - an example would be `/albums`. `type` in that case could be `ids`, `tags`, `city`, `country`. **stupid?**
- `fields`: This parameter will replace all the stupid `detailed=1,includeAlbums=1,includeLikers=1` etc... it defines what additional object info should be returned. For example, on `GET` `/photos/{id}` `fields=user,albums,likers` would additionally include the photographer object, the album details and the photo's likers. **how can we add more details here? for example, the number of likers to return?**
- `subFields`: In some endpoints, we can specify the fields required for child objects. For example, in `/albums/{id}` we can request that photos be included with certain fields. At the moment, we pass fields like `includePhotoLikers`. That's stupid. **what should we replace this with?**
- `lat` & `lng`: Standardized naming for passing latitude and longitude throughout the API.
    - use `ll={latitude},{longitude}`` (a-la foursquare)?
- extend parameter retrieval functions to also validate the passed params. ex: $this->getNumber('lat') actually checks that 'lat' is a number....


Additionally, default values need to be normalized across all endpoints. Any endpoint that returns people (photo likers, album contributors, friends) should for example return the same number of people by default, same applies to pagination limits, etc.

###Pagination parameters
***
Pagination needs a facelift. Offset/Limit doesn't scale, and it can't reliably return all new items. Already in v2.2.0, some endpoints work with `before` and `after`. We should consider switching to either that, or possibly some timestamp based pagination.

###Features
***
- return photog's twitter handle with photo data if available (for native clients, to make twitter sharing more interactive)
- store and use venue's twitter handle (if provided by foursquare)
- regular text + link in news (w/ targeting, available through ccc)
- regular text + link as push notification (w/ targeting, available through ccc)
- friendly urls for albums (eyeem.com/a/berlin)
- API apps: new state (lightweight delete): allowing users to fave/unfave, comment/uncomment, follow/unfollow.
- new format for validation error:

```json
{ "code" : 1024, "message" : "Validation Failed", "errors" : [ { "code" : 5432, "field" : "first_name", "message" : "First name cannot have fancy characters" }, { "code" : 5622, "field" : "password", "message" : "Password cannot be blank" } ] }
```

###Sample API Response header
***

```json
"system": {
  "type": "notice"|"warning"|"error"|"info" (type of message... developer can decide what to do),
  "code": http error code (?)
  "message": optional, it's recommended to display the content to the users,
  "url": optional, in addition to the message, could be the link to the latest app version, for example
  "timestamp": needed??
}
"data": {
  ... whatever else the response contains. (doesn't even need to be called data - "user","photos"...
}
```