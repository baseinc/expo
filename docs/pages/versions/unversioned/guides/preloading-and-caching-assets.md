---
title: Handling Assets
---

This section covers all things related to handling assets with Expo here including bundling, caching, pre-loading and publishing.

### Bundling Assets

Bundling assets into your binary will provide for the best user experience as your assets will be available immediately. Instead of having to make a network request to the CDN to fetch your published assets, your app will fetch them from the local disk resulting in a faster, more efficient loading experience. Bundling assets also allows offline functionality.

To bundle assets in your binary, use the [assetBundlePatterns](../../workflow/configuration/) key in `app.json` to provide a list of paths in your project directory:

```
"assetBundlePatterns": [
  "assets/images/*"
],
```

Images with paths matching the given patterns will be bundled into your native binaries next time you run `expo build`.

> **Note**: If your app contains an abnormal amount of assets or assets that are abnormally large in size, asset bundling may not be the best solution as it will cause your application size to bloat. If this is the case, be selective and bundle those assets that are essential and store the rest on the CDN.

### Pre-loading and Caching Assets

Assets are cached differently depending on where they're stored and how they're used. This guide offers best practices for making sure you only download assets when you need to. In order to keep the loading screen visible while caching assets, it's also a good idea to render [AppLoading](../../sdk/app-loading/#app-loading) and only that component until everything is ready.

For images that saved to the local filesytem, use [`Asset.fromModule(image).downloadAsync()`](../../sdk/asset/) to download and cache the image. There is also a [loadAsync()](../../sdk/asset/#expoassetloadasyncmodules) helper method to cache a batch of assets.

For web images, use `Image.prefetch(image)`. Continue referencing the image normally, e.g. with `<Image source={require('path/to/image.png')} />`.

Fonts are pre-loaded using `Font.loadAsync(font)`. The `font`
argument in this case is an object such as the following: `{OpenSans:
require('./assets/fonts/OpenSans.ttf')}`. `@expo/vector-icons` provides a helpful shortcut for this object, which you see below as `FontAwesome.font`.

```javascript
import * as React from 'react';
import { View, Text, Image } from 'react-native';
import { AppLoading } from 'expo';
import * as Font from 'expo-font';
import { Asset } from 'expo-asset';
import { FontAwesome } from '@expo/vector-icons';

function cacheImages(images) {
  return images.map(image => {
    if (typeof image === 'string') {
      return Image.prefetch(image);
    } else {
      return Asset.fromModule(image).downloadAsync();
    }
  });
}

function cacheFonts(fonts) {
  return fonts.map(font => Font.loadAsync(font));
}

export default class AppContainer extends React.Component {
  state = {
    isReady: false,
  };

  async _loadAssetsAsync() {
    const imageAssets = cacheImages([
      'https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png',
      require('./assets/images/circle.jpg'),
    ]);

    const fontAssets = cacheFonts([FontAwesome.font]);

    await Promise.all([...imageAssets, ...fontAssets]);
  }

  render() {
    if (!this.state.isReady) {
      return (
        <AppLoading
          startAsync={this._loadAssetsAsync}
          onFinish={() => this.setState({ isReady: true })}
          onError={console.warn}
        />
      );
    }

    return (
      <View>
        <Text>Hello world, this is my app.</Text>
      </View>
    );
  }
}
```

See a full working example in [github/expo/new-project-template](https://github.com/expo/new-project-template/blob/master/App.js).


### Publishing Assets

When you publish your project, it will upload your assets to the CDN so that they may be fetched when users run your app. However, in order for assets to be uploaded to the CDN they must be explicitly required somewhere in your application's code. Conditionally requiring assets will result in the packager being unable to detect them and therefore they will not be uploaded when you publish your project.

### Optimizing Assets

You can manually optimize your assets by running the command `expo optimize` which will use the [sharp](https://sharp.pixelplumbing.com/en/stable/) library to compress your assets. You can set the quality of the compression by passing the `--quality [number]` option to the command. For example, to compress to `90%` you would run `expo optimize --quality 0.9`.
