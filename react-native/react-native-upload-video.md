## Aim

- To learn more about React Native components, specifically interaction with the camera roll.
- Upload videos to cloud storage (eg s3)
- Explore transcoding options
- Understand what video formats are suitable for what browsers

## Prerequisites / Assumptions

- using a Mac
- iOS first (developing with the simulator)
- `react-native-cli` is installed and ready to go
- Xcode is installed (and command line tools)

### Create a vanilla React Native app

```bash
$ react-native init vanilla
```

To start the web server side of things:

```bash
$ react-native start
```

To run the app in the iOS simulator:

```bash
$ react-native run-ios
```

### Version control

Now might be a good time to init a git repo and make your initial commit.

```bash
$ git init
$ git add .
$ git commit -m "Initial commit of vanilla React Native app."
```

## Camera roll

So you want to access the camera roll.  There is an API in `react-native`, [CameraRoll](https://facebook.github.io/react-native/docs/cameraroll.html).

In App.js you will want to include `CameraRoll` from `react-native`.

```javascript
import { ..., CameraRoll, ... } from 'react-native'
```

You won't be able to use this straight away as you need to do a couple of things first.

1. The `CameraRoll` API requires the `RCTCameraRoll` library to be linked.  Read more about [linking](https://facebook.github.io/react-native/docs/linking-libraries-ios).
1. Your app needs to ask for the user's permission to access the camera roll.

### Linking RCTCameraRoll

1. Find `~/project-dir/node_modules/react-native/Libraries/CameraRoll/RCTCameraRoll.xcodeproj`
1. In your open project in Xcode, expand Libraries in the Project navigator
    - <img alt="libs" width=200 src="libs.png">
1. Drag and drop `RCTCameraRoll.xcodeproj` to the list of Libraries
1. In the top menubar go to Build Phases
1. Expand "Link Binary With Libraries"
1. Click the + to add a new one
1. Select `libRCTCameraRoll.a`

### Permissions

1. In the top menubar go to "Info"
1. Expand "Custom iOS Target Properties"
1. Hover over any of them and click the little +
1. Select "Privacy - Photo Library Usage Description"
1. Add a description as to why your app requires camera roll permissions

And we're done with Xcode!

## Add videos to Photos app

1. Open Photos in the iOS Simulator
1. Drag and drop a .MOV file (or 3)
    - <img alt="videos" width=200 src="videos.png">
1. We can now browse videos in the camera roll via `getPhotos()`

## Browsing videos on device

In order to select a video to upload we need to create some sort of picker to browse videos.  `getPhotos()` takes some config params, and returns a promise with two keys, `page_info`, and `edges`.  It is not unlike a paginated response from an API.  `page_info` will tell us if there are more pages or not.

```javascript
  getMedia (assetType = 'All') {
    CameraRoll.getPhotos({
      first: 20,
      assetType
    })
      .then(r => {
        this.setState({ media: r.edges })
      })
      .catch((err) => {
        console.error(err)
      })
```

We could set asset type to just "Videos".

An example of (one of the) edges:

```javascript
[
  {
    "node": {
      "timestamp": 1544695544,
      "group_name": "Camera Roll",
      "type": "ALAssetTypeVideo",
      "location": {},
      "image": {
        "playableDuration": 8,
        "isStored": true,
        "filename": "IMG_9220.MOV",
        "width": 1080,
        "height": 1920,
        "uri": "assets-library://asset/asset.MOV?id=8C3DBB4A-2248-45D8-82D8-B279C2F374E4&ext=MOV"
      }
    }
  }
]
```

An example of page_info:

```havascript
{
  "has_next_page": false,
  "start_cursor": "assets-library://asset/asset.MOV?id=8C3DBB4A-2248-45D8-82D8-B279C2F374E4&ext=MOV",
  "end_cursor": "assets-library://asset/asset.MP4?id=A59B71C6-A427-47E0-8D1C-5C14B32D6347&ext=MP4"
}
```

Would we want users to be able to play the videos while they're going through them in the video picker?
Would we want users to be able to share via Photos (and the Android equivalent)?

For our picker, we would grab X at a time (where X is a nicely displayable amount) and have an `onPress` handler for each.  We do this by using the `TouchableOpacity` component to wrap around the video.
