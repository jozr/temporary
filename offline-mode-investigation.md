# SALTY-76: Investigate Offline Mode

## Current Native Implementation

### Data

Overview of data stored:
- Basic user information
  - UUID (?)
  - (?)
- Graph of courses and corresponding lessons (without the actual lesson content)
- Lesson content
- Lesson completion
- The active course
- Review content (?)
- Review completion

### Syncing

Data required to be synced:
- The active course
- Lesson completion
- Review completion

If the sync is not successful, it's stored locally until the next opportunity.

### Offline functionality

Copied directly from [Confluence](https://confluence.internal.babbel.com/wiki/display/MOB/Offline+functionality)

#### Review Sessions
- Review sessions are downloaded automatically (either on WiFi or cellular - depending on users settings) on the following triggers:
  - iOS
    - app start 
    - app resume
    - when the review session is finished
    - after the log in
    - when the language combination is changed
  - Android
    - app start 
    - sometime after a review or lesson was completed (18 hours max)
    - when the last downloaded review was done.
- A user can download the sessions manually. 
- The download is performed for the first 5 review sessions independently of what is there (a user could have the sessions up-to-date already but the app is now aware about that).
- A review session cannot be started while the download is in progress. 

#### Vocabulary items

Vocabulary items are cached (200 items only) when a user accesses the screen of the review manager.

#### Lessons

Lessons are downloaded manually by a user.

The upcoming lesson is downloaded automatically when a user starts the current lesson (Android only).

### iOS specificities

- Data is stored in JSON files
  - No third party library used
- Syncing
  - When the app starts
  - Managed by [Background App Refresh](https://developer.apple.com/documentation/uikit/core_app/managing_your_app_s_life_cycle/preparing_your_app_to_run_in_the_background/updating_your_app_with_background_app_refresh) (?)
- Auth token
  - Using the expiration time from the token, refresh token before call if expired
  - No retry for invalid token if it gets invalidated server-side before expiration
  - Some tricks about _server time_ not matching _system time_
    - Checking headers and having an "adjusted" system time (?)
      - Most likely it is because the auth mechanism used is based on [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm) (?)
        - Some implementations rely, as well, on the client clock being synced with the server
- Pre-fetching
  - Next 5 review sessions
  - When the lesson is started, download the next one
    - Unless it's the last in a course because it's impossible to know the next course (and therefore the lesson)

### Android specificities

- Data stored in a database (?)
- Scheduled jobs (?)

### Notes for Salt from Mobile

- Caching is fine only _if_ you absolutely don't need the images
- They prefer Salt to handle the syncing of data separately. It's possible to bridge the data, but it may introduce risks for them.

## Offline Libraries for React Native

<!-- ### redux-persist

https://github.com/rt2zz/redux-persist

- Requires use of [redux](https://redux.js.org/)
-->

### AsyncStorage

https://facebook.github.io/react-native/docs/asyncstorage

"AsyncStorage is a simple, unencrypted, asynchronous, persistent, key-value storage system that is global to the app. It should be used instead of LocalStorage."

#### NetInfo

https://facebook.github.io/react-native/docs/netinfo

NetInfo is a built-in module that exposes information about the offline and online status. Much of the following information is taken from the documentation.

Cross platform values for `ConnectionType`:
- `none` - device is offline
- `wifi` - device is online
- `cellular` - device is connected via Edge, 3G, WiMax, or LTE
- `unknown` - error case and the network status is unknown

Android-only values for `ConnectionType`:
- `bluetooth` - device is connected via Bluetooth
- `ethernet` - device is connected via Ethernet
- `wimax` - device is connected via WiMax

### react-native-offline

https://github.com/rgommezz/react-native-offline

"Handy toolbelt to deal nicely with offline/online connectivity in a React Native app. Smooth redux integration ✈️"

- Helps alleviate issues with NetInfo
  - "[NetInfo] allows you to detect network connectivity, but it does not guarantee you can exchange data with a remote server even though it reports you are connected."

