Using API
=========
We provide programmatic access to several features of our platform.

1. Log in on https://app.qgraph.io and click your name on the top right
of the screen. In the drop down menu that comes, select "Account Settings".
Note down the "API Token" for your account.

2. You need to set Authorization key as ``"Token: <your API token>"`` in the header of your http requests.
For instance, if using curl, you do it like this::

   curl -H "Authorization: Token abcd" <relevant api url>

Sending notifications
---------------------
First you need to create a campaign. Go to https://app.qgraph.io, log in, go to "Campaigns" tab and create a new campaign. You can fill any value in the campaign, they will be overridden by what you provide in the api call.

Once you have created the campaign, proceed to edit that campaign. URL of the performance page looks like: ``https://app.qgraph.io/#/edit_campaign/<campaign id>``. Note down the campaign id for your campaign.

Next you can make a HTTP POST request at ``https://api.qgraph.io/api/v2/send-notification/``. The POST body is of the following format::

   {
      "cid": <campaign id>,

      "registration_ids": ["regd 1", "regid 2", ..., "regid n" (upto 500 registration ids)]
      or
      "user_ids": ["userid 1", "userid 2", ..., "userid n" (upto 500 userid)]
      or
      "emails": ["email 1", "email 2", ..., "email n" (upto 500 emails)]
      or
      "segment_id": <segment id>
      
      "message": <message in the format described below>
      "os": "android" or "ios-dev" or "ios-prod"
   }

You need to provide one of ``registration_ids``,  ``user_ids``, ``emails`` or ``segment_id``. In case you provide segment id, specified segment id must be a valid segment in your account, and notifications will go to that segment. To find segment id of a given segment, proceed to edit that segment. URL of the segment page is of the format ``https://app.qgraph.io/#/edit_segment/<segment id>``.

If you want to send notification to android devices, use ``android`` for key ``os``. If you want to send notification to ios devices, use ``ios-dev`` or ``ios-prod``, depending on whether you want us to use development profile or production profile. (You should have uploaded the respective .pem file to us)

For a simple android notification, ``message`` is of the following format::

   {
       "type": "basic",
       "title": <title of the notification>,
       "message": <body of the notification>,
       "imageUrl": <url of the icon image> (optional),
       "bigImageUrl": <url of the big image> (optional),
       "deepLink": <deep link of notification> (optional)
   }

For ios notification, ``message`` is of the following format::

   { 
        "aps": {
           "alert": {
               "title": <title of the notification>,
               "body": <body of the notification>
            }
         }
   }

For banner notification (available only in android), ``message`` is of the following format::

   {
       "type": "banner",
       "title": <title of the notification>,
       "message": <body of the notification>,
       "contentImageUrl": <url of banner image>,
       "deepLink": <deep link of notification> (optional)
   }

For animated banner notification (available only in android), ``message`` is of the following format::

   {
       "title": <title of the notification>,
       "message": <body of the notification>,
       "deepLink": <deep link of notification> (optional)
       "type": "animation"
       "animation": {
           "millisecondsToRefresh": <duration between two frames in milliseconds>,
           "images": [url1, url2, ..., url n]
       }
   }

Specifying key value pairs
##########################
You can specify key value pairs in (both android and ios) notifications. To do this, include a key ``qgPayload``
in your ``message`` dictionary. ``qgPayload`` should contain key-value pairs. For example, a sample ``message`` for android
would be::

   {
       "type": "basic",
       "title": <title of the notification>,
       "message: <body of the notification>,
       "imageUrl": <url of the icon image> (optional),
       "bigImageUrl": <url of the big image> (optional),
       "deepLink": <deep link of notification> (optional)
       "qgPayload": {
           "key1": "some value",
           "key2": 123
        }
   }

Key value pairs can then be extracted in your activity as described here: http://docs.qgraph.io/en/latest/integrating-android-sdk.html#receiving-key-value-pairs-in-activity


Getting user profiles
---------------------
Send a GET request to https://app.qgraph.io/api/get-user-profiles/. For instance, if your token is ``abcd``, the relevant call in curl would be::

    curl -H "Authorization: Token abcd" https://app.qgraph.io/api/get-user-profiles/

Specifying start and end dates
##############################
You can optionally provide parameters ``start_date`` and ``end_date`` to the API call. If these parameters are provided, the API fetches
entries only for the users who have installed the app on or after ``start_date``, but on or before ``end_date``. The format of the both the 
arguments is ``yyyy-mm-dd``. A sample call would be::

    curl -H "Authorization: Token <your token>" https://app.qgraph.io/api/get-user-profiles/?start_date=2015-12-22&end_date=2015-12-25

For faster response times, you should retrieve the data for small date ranges.

Specifying OS
#############
You can specify the ios for which you want to retrieve data. You specify this by
providing a query parameter ``os`` whose values can be ``android`` (for android), ``ios-prod`` (for ios using production profile), or ``ios-dev``
(for ios using development profile). Default value for ``os`` is ``android``. Here is an example of using this variable::

    curl -H "Authorization: Token <your token>" https://app.qgraph.io/api/get-user-profiles/?start_date=2015-12-22&end_date=2015-12-25&os=android

Specifying specific fields to retrieve
######################################
You can get following fields using the api:

#. *firstSeen*: Date when the user installed your app
#. *mTime*: Latest date when the user accessed your app
#. *monthlyActivity*: Number of days in last 30 days when the user accessed your app
#. *email*: email of the user, if available
#. *qgCity*: city of the user, if available
#. *uninstallTime*: date when we detected that the user has uninstalled your app
#. *user_id*: the user id set by ``setUserId()`` function of the SDK
#. *qgType*: tells whether the install is a fresh one or a reinstall
#. *qgSrc*: source of the install, if available
#. *gcmId*: gcm registration id of the user in case of android and device token in case of ios
#. *deviceId*: device id of the user
#. *advId*: advertiser id of the user

You can specify what specific fields you want. For instance, if you want to get *firstSeen*, *uninstallTime* and *gcmId* of all the users who installed
your app between December 1, 2015 and December 3, 2015, the relevant curl call would be::

    curl -H "Authorization: Token <your token>" https://app.qgraph.io/api/get-user-profiles/?start_date=2015-12-01&end_date=2015-12-03&fields=firstSeen,uninstallTime,gcmId

For faster response times, you should retrieve only the fields that you need.
