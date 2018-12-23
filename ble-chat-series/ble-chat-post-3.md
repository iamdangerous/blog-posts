# Bluetooth Low energy Chat Application Series

### Overview of Chat application -

1. How Bluetooth Low energy device works
2. Setting up Gatt Server
3. Setting up Gatt Client
4. Sending Data over Bluetooth Low energy network

#### Post 3 - Setting up Gatt Client

1. Get Bluetooth Manager
2. Start scanning nearby devices
    2.1. Get ScanCallback
3. Add scanned devices
4. Connect with scanned deviced
5. Setup Gatt client callback
5. Manage connection state
6. Discover services offered by client
7. Write/Send First charactericts

Required Objects
val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager

startScanProcess()

override fun startScanProcess() {
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

        mScanCallback = BtleScanCallback()
        mBluetoothLeScanner = mBluetoothAdapter.bluetoothLeScanner
        if (mBluetoothLeScanner == null) {
            sendLog("mBluetoothLeScanner is NULL")
        } else {
            mBluetoothLeScanner?.startScan(filters, settings, mScanCallback)
        }


        mScanning = true
        callback?.onScanStarted(mScanning)
    }
