# Bluetooth Low energy Chat Application Series

### Overview of Chat application:

1. How Bluetooth Low energy device works [#Post 1](http://www.rahullohra.com/p/5c02b6a0b5a04022765673bc)
2. Setting up Gatt Server [#Post 2](http://www.rahullohra.com/p/5c0c2264e7319d147ec27b63)
3. Setting up Gatt Client
4. Sending Data over Bluetooth Low energy network

#### Post 3 - Setting up Gatt Client

Things todo:

0. Declare global variables
1. Get Bluetooth Manage
2. Start scanning nearby devices
3. Get ScanCallback
4. Connect with scanned devices
5. Setup Gatt ClientCallback
6. Manage onConnectionStateChange
7. Discover Services of Server(a.k.a Bluetooth Gatt server)
8. Write your very first Descriptor
9. Handle BluetoothGattCallback.onDescriptorWrite(…)
10. Write your very first message from Gatt Client
11. Initialise mConnectedDevices: Inside Gatt server setup


## 0. Declare global variables
````
var mGattConnectedMap = HashMap<BluetoothGatt, Boolean>()
val mDeviceGattMap = HashMap<BluetoothDevice, BluetoothGatt>()
val mGattDeviceMap = HashMap<BluetoothGatt, BluetoothDevice>()
var mScanning = false
val mScannedDeviceIds = HashSet<String>()
val mConnectedDevices: ArrayList<BluetoothDevice> = ArrayList()
val mGattClientCallbackMap = HashMap<BluetoothDevice, GattClientCallback>()
var mClientScanCallback: ScanCallback? = null
var mBluetoothLeScanner: BluetoothLeScanner? = null
val mScannedDevices = HashSet<BluetoothDevice>()
````
## 1. Get Bluetooth Manager
````
val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
````

## 2. Start scanning nearby devices
````
fun startScanning(){
if (mScanning) {
    return
}

val filters = ArrayList<ScanFilter>()
val settings = ScanSettings.Builder()
        .setScanMode(ScanSettings.SCAN_MODE_LOW_POWER)
        .build()
filters.add(ScanFilter.Builder()
        .setServiceUuid(ParcelUuid(SERVICE_UUID))
        .build())

mClientScanCallback = BluetoothScanCallback() // Will setup this in 3rd step
mBluetoothLeScanner = mBluetoothAdapter.bluetoothLeScanner

if (mBluetoothLeScanner == null) {
      sendLog("mBluetoothLeScanner is NULL")
} else {
    mBluetoothLeScanner?.startScan(filters, settings, mScanCallback)
}


mScanning = true
}
 ````

 ## 3. Get ScanCallback
 ````
 private inner class BluetoothScanCallback : ScanCallback() {
    override fun onScanResult(callbackType: Int, result: ScanResult)  
     {
        addScannedDevice(result)
     }

    override fun onBatchScanResults(results: List<ScanResult>) {
        for (result in results) {
            addScannedDevice(result)
        }
    }

    override fun onScanFailed(errorCode: Int) {
        Log.d("Scanning Failed with error code: $errorCode")
    }


    private fun addScannedDevice(result: ScanResult) {
        val device = result.device
        if (device != null && device.address != null) {
            mScannedDeviceIds.add(device.address)
        }

        if (!mScannedDevices.contains(device)) {
            mScannedDevices.add(device)
            Handler().postDelayed({ connectWithScannedDevice(device) }, 1000)
        }
    }
}
````

## 4. Connect with scanned devices
````
fun connectWithScannedDevice(device: BluetoothDevice) {
    Log.d("connectDevice")
    var mGattClientCallback = mGattClientCallbackMap[device]
    if (mGattClientCallback == null) {
        mGattClientCallback = GattClientCallback()
        mGattClientCallbackMap[device] = mGattClientCallback
    }
    val mGatt = device.connectGatt(context, false, mGattClientCallbackMap[device])
    mDeviceGattMap[device] = mGatt
    mGattDeviceMap[mGatt] = device
}
````

## 5. Setup Gatt client callback
````
inner class GattClientCallback : BluetoothGattCallback() {

    //SENDER
    override fun onCharacteristicWrite(gatt: BluetoothGatt?, characteristic: BluetoothGattCharacteristic?, status: Int) {
        super.onCharacteristicWrite(gatt, characteristic, status)
        handleOnCharacteristicWriteOfSender(gatt, characteristic, status)
    }

    
    override fun onDescriptorWrite(gatt: BluetoothGatt?, descriptor: BluetoothGattDescriptor?, status: Int) {
        super.onDescriptorWrite(gatt, descriptor, status)
        handleDescriptorWriteOfSender(gatt, descriptor, status)
    }

   
    override fun onConnectionStateChange(gatt: BluetoothGatt, status: Int, newState: Int) {
        super.onConnectionStateChange(gatt, status, newState)
        handleOnConnectionStateChange(gatt, status, newState)
    }

   
    override fun onServicesDiscovered(gatt: BluetoothGatt?, status: Int) {
        super.onServicesDiscovered(gatt, status)
        handleOnServicesDiscovered(gatt, status)
    }

   
    override fun onCharacteristicChanged(gatt: BluetoothGatt?, characteristic: BluetoothGattCharacteristic?) {
        super.onCharacteristicChanged(gatt, characteristic)
        handleOnCharacteristicChanged(gatt, characteristic)

    }

}
````

## 6. Manage onConnectionStateChange
````
fun handleOnConnectionStateChange(gatt: BluetoothGatt, status: Int, newState: Int){
if (status == BluetoothGatt.GATT_FAILURE) {
       disconnectFromGattServer(gatt)
       return
   } else if (status != BluetoothGatt.GATT_SUCCESS) {
       disconnectFromGattServer(gatt, true)
       return
   }
   if (newState == BluetoothProfile.STATE_CONNECTED) {
       handleConnectedStateOfClient(gatt)
   } 
      else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
       disconnectFromGattServer(gatt, true)
   }
}
fun handleConnectedStateOfClient(bluetoothGatt: BluetoothGatt){
   mGattConnectedMap[bluetoothGatt] = true
   mDeviceGattMap[bluetoothGatt.device] = bluetoothGatt
   mGattDeviceMap[bluetoothGatt] = bluetoothGatt.device
   bluetoothGatt.discoverServices()
}
fun disconnectFromGattServer(gatt: BluetoothGatt, retry: Boolean = false){
   mGattConnectedMap[gatt] = false
   if (retry) {
       gatt.connect()
   } else {
       gatt.disconnect()
       gatt.close()
   }
}
````

>bluetoothGatt.discoverServices will call ***BluetoothGattCallback.onServicesDiscovered(…)*** . We have to handle this now
## 7. Discover Services of Server(a.k.a Bluetooth Gatt server)
````
fun handleOnServicesDiscovered(gatt: BluetoothGatt?, status: Int){
   if (status != BluetoothGatt.GATT_SUCCESS) {
       return
   }
   if (gatt != null) {
      writeFirstDescriptor(gatt)
   }
}
````
## 8. Write your very first Descriptor
````
fun writeFirstDescriptor(gatt: BluetoothGatt){
   val service = gatt.getService(SERVICE_UUID)
   var metaDataCharacteristic = 
       service.getCharacteristic(USER_META_DATA_UUID)
   if (metaDataCharacteristic != null) {
       metaDataCharacteristic.writeType =     
       BluetoothGattCharacteristic.WRITE_TYPE_NO_RESPONSE
var descriptor =  
   metaDataCharacteristic.getDescriptor(USER_META_DATA_DESCRIPTOR_UUID)
   descriptor?.value = 
   BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE
  if (descriptor != null) {
      val descriptorWriteStartedSuccess = 
            gatt.writeDescriptor(descriptor)
        //check whether write is success or not
    } else {
        Log.d("DESCRIPTOR is null")
   }
  }
}
````
>The important thing about writing first descriptor is basically that, the client will now receive a callback on ***BluetoothGattCallback.onDescriptorWrite(…)***
If we get a successful then it means two devices can now talk

## 9. Handle BluetoothGattCallback.onDescriptorWrite(…)
````
fun handleDescriptorWriteOfSender (gatt: BluetoothGatt?, descriptor: BluetoothGattDescriptor?, status: Int){
    if (status == BluetoothGatt.GATT_SUCCESS) {
       sendYourVeryFirstMessage(gatt)
   } else {
       if (gatt != null && mGattDeviceMap[gatt] != null) {         
     handleFailureOfDescriptionWrite(mGattDeviceMap[gatt]!!.address)
    }
  }
}
````
>If you want to send messages in queue fashion use a proper data structure for that. You also have to customise sendYourVeryFirstMessage(gatt). Remember the ***MTU(Maximum transfer unit)*** size for BLE will range from **23** bytes till **512** bytes

>And you can definitely request to increase but I won’t recommend that because it will vary from device to device and OS to OS

## 10. Write your very first message from Gatt Client
````
fun sendYourVeryFirstMessage(gatt: BluetoothGatt?, characteristicUuid: UUID = USER_META_DATA_UUID, bytes: ByteArray? = null){
try {
    if (gatt != null) {
        val mGattConnected = mGattConnectedMap[gatt]
        if ((mGattConnected == null || !mGattConnected) && !mConnectedDevices.contains(gatt.device)) {
            handleFailureOfSendingMessage()
            return
        }
        val service = gatt.getService(SERVICE_UUID)
        if (service != null) {
            val characteristic = service.getCharacteristic(characteristicUuid)
            characteristic?.value = bytes
            characteristic?.writeType = BluetoothGattCharacteristic.WRITE_TYPE_NO_RESPONSE
            val success = gatt.writeCharacteristic(characteristic)
            //check for success writing
            //You will get the message on    
            //GattServerCallback.onCharacteristicWriteRequest(..)
        } else {
            handleFailureOfSendingMessage()
        }
    } else {
        handleFailureOfSendingMessage()
    }
} catch (E: NullPointerException) {
    handleFailureOfSendingMessage()
}
}
````
>***mConnectedDevices***: A new variable, it should be defined in Gatt Server setup. I will show you the code below. handleFailureOfSendingMessage(): This functions basically handles the failed messages. For now we will do nothing

## 11. Initialise mConnectedDevices: Inside Gatt server setup

````
inner class GattServerCallback : BluetoothGattServerCallback() {
...
override fun onConnectionStateChange(device: BluetoothDevice?, status: Int, newState: Int){
    super.onConnectionStateChange(device, status, newState);
    handleOnConnectionStateChangeServer(device, status, newState)
}
...
}//end of inner class GattServerCallback
fun handleOnConnectionStateChangeServer(bleDevice: BluetoothDevice?, status: Int, newState: Int){
if (bleDevice != null) {
    if (newState == BluetoothProfile.STATE_CONNECTED) {
          mConnectedDevices.add(bleDevice)
          connectNewDevice(bleDevice)
      } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
          mConnectedDevices.remove(bleDevice)
      }
   }
  }

fun connectNewDevice(device:BluetoothDevice){
  var mGattClientCallback = mGattClientCallbackMap[device]
  if (mGattClientCallback == null) {
  // Means a new device, so add it
      mGattClientCallback = GattClientCallback()
      mGattClientCallbackMap[device] = mGattClientCallback
      val mGatt = device.connectGatt(context, false,  
                  mGattClientCallbackMap[device])
      mDeviceGattMap[device] = mGatt
      mGattDeviceMap[mGatt] = device
  }
}

````

#### Important Notes:
> Our device is acting as both client and server, that’s why we have to save device’s reference from ***handleOnConnectionStateChangeServer(..)***

At last, setting up Gatt client is completed 😌

>Code: https://github.com/iamdangerous/Ble-Chat-Android-app/blob/post_3_setup_gatt_client/app/src/main/java/com/rahullohra/blechatapp/BluetoothController.kt

## References
- https://www.bignerdranch.com/blog/bluetooth-low-energy-part-1/
- https://www.bignerdranch.com/blog/bluetooth-low-energy-part-2/

