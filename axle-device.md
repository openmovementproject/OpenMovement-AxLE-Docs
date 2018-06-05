# AxLE Device

## Working with the AxLE device

Now we have connected to an AxLE device we can now start interacting directly with the device. We can vibrate the device, flash its LEDs, stream accelerometer data, sync step and Epoch data and configure other device settings.

### Authenticating with the device

The functionality available to you before authenticating with an AxLE device is limited. To authenticate with a blank device you can use the last 6 digits of the device serial number. You can use the `Authenticate` endpoint to start:

```csharp
var pass = device.SerialNumber.Substring(device.SerialNumber.Length - 6);
if (await device.Authenticate(pass))
{
    // Horay!!
}
else
{
    // Oh no!
}
```

The `Authenticate` function will return `true` or `false` depending on the result of the auth. If you are unable to access your device you will need to reset it.

{% hint style="info" %}
At this point you should consider how you are going to manage passwords for your devices. You can leave the password as its default though this leaves the band open to others to sync with and otherwise manipulate.
{% endhint %}

#### Resetting an AxLE device \(THIS WILL ERASE ALL DATA\)

To reset the password and erase the device you can use the following:

```csharp
await device.ResetPassword();
```

The device will now be reset to its default username and password.

### Device State

Once authenticated with the device the device state information can now be read and the relevant properties of the AxLE device object populated with data. To read config and battery information etc. call the following:

```csharp
await device.UpdateDeviceState();
```

This runs a series of commands on the device and reads state information.

### Configuring an AxLE device

The AxLE device has a range of settings and parameters for you to configure. Step goals, Epoch storage intervals and Cueing with the vibration motor for example. Many of these settings can be controlled by simply setting the required property on the device. **Again, ensure** `UpdateDeviceState();` **has been called before attempting to read properties of the device.**

### Syncing Data

AxLE devices can log activity data for up to a month. Sync data from this can be synced from the band in blocks. There are a few bits of pre-configuration required to start tracking activity data from a user.

#### Setting up a band for tracking

To begin tracking user activity you first need to configure the devices `EpochPeriod`. This value represents the interval in seconds that Epoch samples are recorded. The significance of this is in the duration of logging and the granularity of the data you would like to log.

* A longer period results in more compacted activity data resulting enabling a device to log for much longer and much shorter sync operations for example a days worth of data.
* A shorter period results in more detailed activity data, the device will only be able to log data for a shorter period of time and sync operations for a similar days worth of data will take longer.

Once you have decided on a period that suits, you can begin collecting data. You need to record the starting point for the data for your sync operation. This can be done by reading the block details from the device:

```csharp
BlockDetails blockDetails = await device.ReadBlockDetails();
blockDetails.ActiveBlock // Store this for the first sync
```

As we are setting up the activity tracking for the first time we can just look at the `ActiveBlock` number to work out where to start. This is the block that the device is currently storing samples to, if you were to sync this block it would be incomplete. When you come back to this device in the future and attempt to sync with it you will use this block number as your starting point. You also need to store the device's time and a reference time \(**Remember to call** `UpdateDeviceState();` **before reading the device time**\).

To summise here is the data you need:

* StartingBlock \(usually `ActiveBlock`\)
* DeviceTime
* LastSyncTime \(usually `DateTime.NowUtc`\)

#### Syncing

Now that we have a starting point for our data, the device can now be left to log data. When you are ready to sync this activity data you can do the following:

```csharp
// lastBlock for initial sync is StartingBlock
// lastRtc is the DeviceTime (update each time)
// lastSync is LastSyncTime
EpochBlock[] blocks = await device.SyncEpochData(lastBlock, lastRtc, lastSync);
blocks.Last() // Contains partial ActiveBlock (this is your new StartingBlock going forward), discard or store as appropriate
```

Thats it!! You have now synced data from your device. Managing where to sync from and too is up to you. One of the `SyncEpochData` overloads also takes a `syncTo` block number parameter, this can be used to control how much data is synced. You can also check the `ActiveBlock` before syncing with `ReadBlockDetails` to have more control over the sync and to check how much data is available. Each block is 512 bytes, consider the sync time when transferring large amounts of data \(on average this is ~500 milliseconds a block\).

### Accelerometer Stream

The AxLE device can also stream its acclerometer data in raw from the device. To get started you need to attach to the `AccelerometerStream` event like so:

```csharp
device.AccelerometerStream += (sender, accBlock) =>
{
    Console.WriteLine($"x:{accBlock.Samples[0][0]}, y:{accBlock.Samples[0][0]}, z:{accBlock.Samples[0][0]}");
};
```

Now we can start the stream with:

```csharp
await device.StartAccelerometerStream();
```

Stop the accelerometer stream with:

```csharp
await device.StopAccelerometerStream();
```

{% hint style="info" %}
Note, there are 25 samples per block. To show a smooth stream of data on a UI you need to stagger each sample.
{% endhint %}



