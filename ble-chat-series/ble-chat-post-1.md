# Bluetooth Low energy Chat Application Series

### Overview of Chat application -

1. How Bluetooth Low energy device works
2. Setting up Gatt Server [#Post 2](http://www.rahullohra.com/p/5c0c2264e7319d147ec27b63)
3. Setting up Gatt Client [#Post 3](http://www.rahullohra.com/p/5c227ada01d2df134d78d45e)
4. Sending Data over Bluetooth Low energy network

#### Post 1 - How Bluetooth Low energy device works in general

Key components

**Ble** - Bluetooth Low energy device

**GATT** - Defines protocol(set of rules) on how data should be exchanged

**GATT Server** - Provides functionality to create **Services** and broadcast itself so other **Gatt Client** can connect to it. (can be slightly confusing, I know but don't worry, this will be cleared later). Treat it as a web server.

**Gatt Client** - It is responsible for scanning nearby **GATT Server**, connecting to it and exchanging data(in terms of **Characteristics**).

**Service** - A service is a collection of characteristics. For example, you could have a service called "Heart Rate Monitor" that includes characteristics such as "heart rate measurement."

**Characteristics** - It contains a value and a list of **descriptors**

**Descriptors** - It contains a value and generally it is used as attributes of a **Characteristics**

**UUID** - Just a unique identifier, on how to uniquely identify/create a Service or a characteristics or a descriptor.

For exchanging data over http network, we need two devices one will act as a server and another will act as client. Eg - that's how website work

This same thing applies in Ble network, we need two devices one will act as a **Gatt server** and another will act as **Gatt client** for exchanging of data.

And a device can be both Gatt Server and Gatt client.

There are still some concepts left, I will teach them when they are used.

For Android chat application, we will have two devices and they both will act as a Gatt Server and Gatt Client.

Reference - https://developer.android.com/guide/topics/connectivity/bluetooth-le
