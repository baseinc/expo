---
title: Accelerometer
sourceCodeUrl: "https://github.com/expo/expo/tree/sdk-34/packages/expo-sensors"
---

import SnackInline from '~/components/plugins/SnackInline';


Access the device accelerometer sensor(s) to respond to changes in acceleration in 3d space. Note that the accelerometer hardware is [not supported in the iOS Simulator](../../workflow/ios-simulator/#limitations).

## Installation

For [managed](../../introduction/managed-vs-bare/#managed-workflow) apps, you'll need to run `expo install expo-sensors`. To use it in a [bare](../../introduction/managed-vs-bare/#bare-workflow) React Native app, follow its [installation instructions](https://github.com/expo/expo/tree/master/packages/expo-sensors).

## Usage

<SnackInline label="Basic Accelerometer usage" templateId="accelerometer" dependencies={["expo-sensors"]}>

```javascript
import React, {Component} from 'react';
import { StyleSheet, Text, TouchableOpacity, View } from 'react-native';
import { Accelerometer } from 'expo-sensors';

export default class AccelerometerSensor extends React {
  state = {
    accelerometerData: {},
  };

  componentDidMount() {
    this._toggle();
  }

  componentWillUnmount() {
    this._unsubscribe();
  }

  _toggle = () => {
    if (this._subscription) {
      this._unsubscribe();
    } else {
      this._subscribe();
    }
  };

  _slow = () => {
    Accelerometer.setUpdateInterval(1000);
  };

  _fast = () => {
    Accelerometer.setUpdateInterval(16);
  };

  _subscribe = () => {
    this._subscription = Accelerometer.addListener(accelerometerData => {
      this.setState({ accelerometerData });
    });
  };

  _unsubscribe = () => {
    this._subscription && this._subscription.remove();
    this._subscription = null;
  };

  render() {
    let { x, y, z } = this.state.accelerometerData;
    return (
      <View style={styles.sensor}>
        <Text style={styles.text}>Accelerometer: (in Gs where 1 G = 9.81 m s^-2)</Text>
        <Text style={styles.text}>
          x: {round(x)} y: {round(y)} z: {round(z)}
        </Text>
        <View style={styles.buttonContainer}>
          <TouchableOpacity onPress={this._toggle} style={styles.button}>
            <Text>Toggle</Text>
          </TouchableOpacity>
          <TouchableOpacity onPress={this._slow} style={[styles.button, styles.middleButton]}>
            <Text>Slow</Text>
          </TouchableOpacity>
          <TouchableOpacity onPress={this._fast} style={styles.button}>
            <Text>Fast</Text>
          </TouchableOpacity>
        </View>
      </View>
    );
  }
}

function round(n) {
  if (!n) {
    return 0;
  }

  return Math.floor(n * 100) / 100;
}

const styles = StyleSheet.create({
  buttonContainer: {
    flexDirection: 'row',
    alignItems: 'stretch',
    marginTop: 15,
  },
  button: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#eee',
    padding: 10,
  },
  middleButton: {
    borderLeftWidth: 1,
    borderRightWidth: 1,
    borderColor: '#ccc',
  },
  sensor: {
    marginTop: 45,
    paddingHorizontal: 10,
  },
  text:{
    textAlign: 'center'
  }
});
``` 
</SnackInline>

## API

```js
import { Accelerometer } from 'expo-sensors';
```

### `Accelerometer.isAvailableAsync()`

> You should always check the sensor availability before attempting to use it.

Returns whether the accelerometer is enabled on the device.

On **web** this starts a timer and waits to see if an event is fired. This should predict if the iOS device has the **device orientation** API disabled in `Settings > Safari > Motion & Orientation Access`. Some devices will also not fire if the site isn't hosted with **HTTPS** as `DeviceMotion` is now considered a secure API. There is no formal API for detecting the status of `DeviceMotion` so this API can sometimes be unreliable on web.

#### Returns

- A promise that resolves to a `boolean` denoting the availability of the sensor.

### `Accelerometer.addListener(listener)`

Subscribe for updates to the accelerometer.

#### Arguments

- **listener (_function_)** -- A callback that is invoked when an
  accelerometer update is available. When invoked, the listener is
  provided a single argument that is an object containing keys x, y,
  z. Each of these keys represents the acceleration along that particular axis in Gs (A G is a unit of gravitational force equal to that exerted by the earth’s gravitational field (9.81 m s<sup>-2</sup>).

#### Returns

- A subscription that you can call `remove()` on when you
  would like to unsubscribe the listener.

### `Accelerometer.removeAllListeners()`

Remove all listeners.

### `Accelerometer.setUpdateInterval(intervalMs)`

Subscribe for updates to the accelerometer.

#### Arguments

- **intervalMs (_number_)** Desired interval in milliseconds between
  accelerometer updates.

#
