## Getting Started
Create a project with an API Level target of at least 18, or JellyBean 4.3.

To use the Javelin SDK add the following to your project's `build.gradle`:
```
repositories {
    maven { url "https://jitpack.io" }
}
```
In the application build.gradle declare the following dependency:
```
dependencies {
    compile 'com.github.javelin-dev:javelin-sdk-android:v1.0.0'
}
```
Finally, **you must download and install** the [JavelinServiceApp](https://github.com/javelin-dev/javelin-sdk-examples/releases/download/v1.0/JavelinService.apk) application.
This application manages connections between any clients (other apps) wanting to use the
Javelin device at the same time. Your android device must be Bluetooth Low Energy compatible.


You can also [checkout or download the example project](https://github.com/javelin-dev/javelin-sdk-examples) to get you started.

## Connecting to the Javelin Device
### Setting permissions
To be able to connect to the Javelin device, declare the following permissions in the manifest file:
```
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```

Once the permissions are set up, all we need is the device address that you want to connect to. To retrieve this we we can either:

-  [Scan all nearby devices](#nearby) and retrieve the device address of the one you'd like to connect to.
- Let the Javelin SDK [connect to the closest available](#autoconnect) Javelin Device.


### Scanning nearby devices  <a name="nearby"></a>
If you'd like to scan the nearby area for Javelin devices, you can create a Bluetooth scanner activity and select the device you're looking for.

Refer to the [android developer documentation](https://developer.android.com/guide/topics/connectivity/bluetooth-le.html#find) to help you create one of these activities.

In addition, you can also refer the [example](https://github.com/javelin-dev/javelin-sdk-examples/blob/master/app/src/main/java/com/javelindevices/javelinexamples/ScanActivity.java) we've created to make it easier.

### Connecting using the JavelinSensorManager <a name="autoconnect"></a>

If you have the javelin's `device address`, from a scanning activity or a previously address, you can instantiate the `JavelinSensorManager` with an Activity context and the `device address` and initiate a connection through it. Note that if you want to listen to Javelin events -- such as connection events -- the class will need to implement the `JavelinEventListener` interface.

```
public class MainActivity extends Activity implements ISensorManager.JavelinEventListener {
JavelinSensorManager mSensorManager;
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		String mDeviceAddress = "ABCDEF";
		mSensorManager = new JavelinSensorManager(this, mDeviceAddress);
		mSensorManager.setListener(this); // this activity will listen to connection events
		mSensorManager.enable(); // Initiates the connection
	}

	@Override
	public void onSensorManagerConnected() {
		// If this gets called, we've succeeded!
	}
	...
	...
	// Other implemented methods
	...
	...
}
```


> **Note:** The `JavelinSensorManager` will remember any commands sent while the device is not connected, and will attempt to send them once connection is achieved.


###Disconnecting
To disconnect from the Javelin, simply call `mSensorManager.disable()`. You can verify whether the disconnection took place by checking the `onSensorManagerDisconnected()` callback.



## Enabling a sensor and receiving data
Via our `JavelinSensorManager` object, we can register listeners to various Javelin sensors and receive data from them.
```

public class SomeActivity extends Activity implements ISensorManager.JavelinEventListener {
	...
	...
	@Override
	public void onResume() {
		super.onResume();
		// Enables the accelerometer on the Javelin device
		mSensorManager.registerListener(this, ISensor.TYPE_ACCELEROMETER);
	}

	@Override
	public void onPause() {
		super.onPause();
		// Disables the accelerometer
		mSensorManager.unregisterListener(this, ISensor.TYPE_ACCELEROMETER);
	}
}
```
We can then listen for sensor data via our `onSensorChanged(JavelinSensorEvent event)` callback.
```java
	@Override
	public void onSensorChanged(JavelinSensorEvent event) {
		float[] sensorData = event.values;
		switch (event.sensorType) {
			case ISensor.TYPE_ACCELEROMETER:
				// Received some data from the accelerometer!
				int x = sensorData[0]; // Do something with it
				break;
			case ISensor.TYPE_GYROSCOPE:
				...
				...
				// Other sensors
		}
	}
```

## Javelin Settings
### LED and Vibration

There are four LED blinking and vibration settings:

- **rate** of vibration / blinking, in .1Hz values
- **type**, which can be one of the values found in the `JavelinSettings` class.
- **intensity** on a scale of 1-100
- **number of times** to vibrate / blink at the given rate, type and intensity -- from 1 to 254 times. A value of 255 causes it to enable the component indefinitely until the javelin device disconnects or the component is disabled.

When you call`mSensorManager.vibrationEnable(true)` or `mSensorManager.ledEnable(true)`,  the LED or vibrator will vibrate according to the settings that were set. If the vibration or LED are on, and a setting is sent, the Javelin will change the type, intensity, or rate of blinking in real time.


### Sensor Settings
You can change the sample rate (Hz) of the accelerometer and gyroscope by using the `setAcceleromGyroRate(int rate)` method on the `JavelinSensorManager`.

You can also change the full scale range of the accelerometer and gyroscope with the functions 
`setAccelerometerFullScaleRange(int range)` and `setGyroscopeFullScaleRange(int range)`


### Work in Progress APIs
The following methods on the `JavelinSensorManager` are not fully functional yet:


 - createbond()
 - removeBond()
 - setAudioQuality()
 
We also are in progress of adding


 - Additional audio controls (number of samples per packet)
 - Pedometer controls
 - Quaternion controls
 - Sensor data filters and smoothing settings
 - Buffering of data for later retrieval by the android device
 - Ability to create Javelin accounts and register devices
 - Temperature sampling rate controls