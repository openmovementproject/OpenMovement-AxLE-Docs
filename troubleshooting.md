# Troubleshooting

## How can I view the commands being sent to the device?

To aid debugging a range of errors with the device we have included a build flag `DEBUG_COMMS`. Use this flag to log all input and output from the serial port to the console.

## I am getting CommandFailedExceptions?

Command failed exceptions are caused when a command times out or the response was not what was expected. Inspect the `DataDump` field on the exception to assist in debugging.

This can also be caused by not properly disconnecting from an AxLE before reconnecting in the future. Always make sure you disconnect.

## I am getting an GATT Error 133 when connecting on Android?

This error is usually caused by the Bluetooth stack being busy or a little bit overloaded when trying to connect. Unfortunately these errors are hard to avoid entirely but make sure you stop scanning before connecting.

## I am getting a BlockSyncFailedException when syncing data?

Block sync failed is caused by a block failing checksum validation. The block is syned twice unless manually syncing each block so it is likely the block you are trying to sync \(or rather one within the range you are syncing\) is corrupt. This is unlikely to happen randomly but can occur due to Epoch configuration being altered mid activity logging, always update to `ActiveBlock` when changing config. You will need to manually sync around these blocks to address this.

## I am getting a DeviceIncompatibleException when connecting?

This exception is thrown when the firmware is unrecognised by the library. This exception can occur when the device interrogation has failed and may be fixed by retrying connection.

