# OpenMovement AxLE

This is the sync client for the OpenMovement AxLE Band. This client will handle communication with the AxLE bands and sync all data to the OpenMovement repository for retrieval by one of our partner services.

Functionality:

* Syncing and Viewing of Epoch data from AxLE band.
* Goal settings and configuration.
* General band configuration.
* Data syncing service.
* Firmware Updater.

This application is available on the [iOS](https://itunes.apple.com/us/app/openmovement-axle/id1298548301?ls=1&mt=8) and [Android](https://play.google.com/store/apps/details?id=uk.ac.ncl.OpenLab.OpenMovement.AxLE.App) app stores.

## Getting Started

The OpenMovement AxLE Band API is a set of C\# libraries with pre-built support for Xamarin \(iOS, Android & UWP\) and Windows 10 APIs.

#### Xamarin & Windows

To get started with a Xamarin or Windows application create a new project in Visual Studio. Run the following command to add the OpenMovement AxLE respository as a sub-module.

```bash
$ git submodule add -b production https://github.com/digitalinteraction/OpenMovement-AxLE
```

{% hint style="info" %}
The OpenMovement AxLE library has been migrated to NuGet packages which are available for each platform \(still in beta\):

* OpenMovement AxLE Comms
* OpenMovement AxLE Comms iOS
* OpenMovement AxLE Comms Android
* OpenMovement AxLE Service
{% endhint %}

{% hint style="warning" %}
When using a non-Universal Windows application you must manually add the `Windows.Devices` namespace as outlined [here](http://kiewic.com/2015-11-24/how-to-use-windows-10-runtime-store-universal-apis-in-desktop-console-apps). Ensure the Build flags `WIN_CONSOLE` or `WINDOWS_UWP` are set appropriately.
{% endhint %}

Once added as a submodule include the `OpenMovement.AxLE.Comms` and `OpenMovement.AxLE.Service` as referenced shared projects in your new solution.

#### Other Platforms

If you would like to use another platform to communicate with the OpenMovement AxLE devices we have simplified this process for you.

You just need to implement the following interfaces for the nativave Bluetooth interfaces you are using:

* OpenMovement.AxLE.Comms.Bluetooth.Interfaces.IBluetoothManager
* OpenMovement.AxLE.Comms.Bluetooth.Interfaces.IDevice
* OpenMovement.AxLE.Comms.Bluetooth.Interfaces.IService
* OpenMovement.AxLE.Comms.Bluetooth.Interfaces.ICharacteristic

You can use the existing `Mobile` and `Windows` implementations as guides both in the `Bluetooth` namespace in the comms library.

### Scanning for AxLE Bands

All interactions with the AxLE devices start with the `AxLEManager`. This manager is responsible for scanning and monitoring, connecting, disconnecting and firmware updating devices.

To begin we need to create an instance of the manager:

```csharp
IAxLEManager manager = new AxLEManager(nativeBluetoothManager, [nearbyTimeout = 30000]);
```

Now we need to set our relevant callbacks for when a new device is found:

```csharp
manager.DeviceFound += (sender, serial) =>
{
    // Do something amazing...
};
```

Finally we call `manager.StartScan();` to begin scanning for AxLE devices and trigger our `DeviceFound` callback when we have found devices nearby. To stop scanning call `manager.StopScan();`.

### Connecting to an AxLE device

We again use the manager to connect to an AxLE device. The `AxLEManager` returns a serial number string for devices it has found. We shall use this as follows:

```csharp
try
{
    manager.ConnectDevice(serial);
}
catch (DeviceNotInRangeException ex)
{
    // Called if the device has not yet been discovered
    // by the manager or is no longer nearby
}
catch (Exception ex)
{
    // Depending on platform other Bluetooth Comms
    // exceptions may be thrown by the stack.
    // Android Gatt errors for example which can
    // usually be ignored and tried again.
}
```

{% hint style="warning" %}
On some platforms you should call `manager.StopScan();` before connecting to a device. Android is particularly sensitive to this and will result in a higher likely hood of Gatt errors if the manager is still scanning.
{% endhint %}

### Disconnecting from an AxLE device

Disconnecting from an AxLE once you are finished with the device is very important, reconnecting to an already connected device may cause serial port issues and commands may fail. Disconnect from devices like this:

```csharp
await manager.DisconnectDevice(device);
```

### Using the Devices Nearby monitor

The AxLE manager can monitor for devices nearby and keep you up to date as devices appear and disappear from nearby your device.

There are two properties for you to configure in this instance, the `nearbyTimeout` parameter on the constructor and the `RssiFilter` \(lower is closer\) property on the manager.

{% hint style="danger" %}
On Windows the Nearby monitor may not work as expected. The Windows Bluetooth APIs are incomplete and lack the functionality required to implement a complete nearby monitor.
{% endhint %}

Once the nearby monitor is configured you can start using the relevant callbacks to monitor devices:

```csharp
manager.DeviceFound += (sender, serial) =>
{
    // Called each time a device that was once lost is found.
    // Like the ring of power...
};

manager.DeviceLost += (sender, serial) =>
{
    // Called each time a device is considered lost.
};
```

