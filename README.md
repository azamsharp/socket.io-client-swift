[![Build Status](https://travis-ci.org/socketio/socket.io-client-swift.svg?branch=master)](https://travis-ci.org/socketio/socket.io-client-swift)

#Socket.IO-Client-Swift
Socket.IO-client for iOS/OS X.

##Example
```swift
let socket = SocketIOClient(socketURL: "localhost:8080", options: [.Log(true), .ForcePolling(true)])

socket.on("connect") {data, ack in
    print("socket connected")
}

socket.on("currentAmount") {data, ack in
    if let cur = data[0] as? Double {
        socket.emitWithAck("canUpdate", cur)(timeoutAfter: 0) {data in
            socket.emit("update", ["amount": cur + 2.50])
        }

        ack?.with("Got your currentAmount", "dude")
    }
}

socket.connect()
```

##Objective-C Example
```objective-c
SocketIOClient* socket = [[SocketIOClient alloc] initWithSocketURL:@"localhost:8080" options:@{@"log": @YES, @"forcePolling": @YES}];

[socket on:@"connect" callback:^(NSArray* data, SocketAckEmitter* ack) {
    NSLog(@"socket connected");
}];

[socket on:@"currentAmount" callback:^(NSArray* data, SocketAckEmitter* ack) {
    double cur = [[data objectAtIndex:0] floatValue];

    [socket emitWithAck:@"canUpdate" withItems:@[@(cur)]](0, ^(NSArray* data) {
        [socket emit:@"update" withItems:@[@{@"amount": @(cur + 2.50)}]];
    });

    [ack with:@[@"Got your currentAmount, ", @"dude"]];
}];

[socket connect];

```

##Features
- Supports socket.io 1.0+
- Supports binary
- Supports Polling and WebSockets
- Supports TLS/SSL
- Can be used from Objective-C

##Installation
Requires Swift 2/Xcode 7

If you need Swift 1.2/Xcode 6.3/4 use v2.4.5 (Pre-Swift 2 support is no longer maintained)

If you need Swift 1.1/Xcode 6.2 use v1.5.2. (Pre-Swift 1.2 support is no longer maintained)

Manually (iOS 7+)
-----------------
1. Copy the SocketIOClientSwift folder into your Xcode project. (Make sure you add the files to your target(s))
2. If you plan on using this from Objective-C, read [this](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html) on exposing Swift code to Objective-C.

Carthage
-----------------
Add this line to your `Cartfile`:
```
github "socketio/socket.io-client-swift" ~> 4.0.2 # Or latest version
```

Run `carthage update --platform ios,macosx`.

CocoaPods 0.36.0 or later (iOS 8+)
------------------
Create `Podfile` and add `pod 'Socket.IO-Client-Swift'`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
use_frameworks!

pod 'Socket.IO-Client-Swift', '~> 4.0.2' # Or latest version
```

Install pods:

```
$ pod install
```

Import the module:

Swift:
```swift
import Socket_IO_Client_Swift
```

Objective-C:

```Objective-C
#import <Socket_IO_Client_Swift/Socket_IO_Client_Swift-Swift.h>
```

CocoaSeeds
-----------------

Add this line to your `Seedfile`:

```
github "socketio/socket.io-client-swift", "v4.0.2", :files => "SocketIOClientSwift/*.swift" # Or latest version
```

Run `seed install`.


##API
Constructors
-----------
`init(var socketURL: String, options: Set<SocketIOClientOption>? = nil)` - Creates a new SocketIOClient. opts is a Set of SocketIOClientOption. If your socket.io server is secure, you need to specify `https` in your socketURL.

`convenience init(socketURL: String, options: NSDictionary?)` - Same as above, but meant for Objective-C. See Options on how convert between SocketIOClientOptions and dictionary keys.

Options
-------
All options are a case of SocketIOClientOption. To get the Objective-C Option, convert the name to lowerCamelCase.

```swift
case ConnectParams([String: AnyObject]) // Dictionary whose contents will be passed with the connection.
case Reconnects(Bool) // Whether to reconnect on server lose. Default is `true`
case ReconnectAttempts(Int) // How many times to reconnect. Default is `-1` (infinite tries)
case ReconnectWait(Int) // Amount of time to wait between reconnects. Default is `10`
case ForcePolling(Bool) // `true` forces the client to use xhr-polling. Default is `false`
case ForceWebsockets(Bool) // `true` forces the client to use WebSockets. Default is `false`
case Nsp(String) // The namespace to connect to. Must begin with /. Default is `/`
case Cookies([NSHTTPCookie]) // An array of NSHTTPCookies. Passed during the handshake. Default is nil.
case Log(Bool) // If `true` socket will log debug messages. Default is false.
case Logger(SocketLogger) // Custom logger that conforms to SocketLogger. Will use the default logging otherwise.
case SessionDelegate(NSURLSessionDelegate) // Sets an NSURLSessionDelegate for the underlying engine. Useful if you need to handle self-signed certs. Default is nil.
case Path(String) // If the server uses a custom path. ex: `"/swift"`. Default is `""`
case ExtraHeaders([String: String]) // Adds custom headers to the initial request. Default is nil.
case HandleQueue(dispatch_queue_t) // The dispatch queue that handlers are run on. Default is the main queue.
```
Methods
-------
1. `on(event: String, callback: NormalCallback)` - Adds a handler for an event. Items are passed by an array. `ack` can be used to send an ack when one is requested. See example.
2. `once(event: String, callback: NormalCallback)` - Adds a handler that will only be executed once.
3. `onAny(callback:((event: String, items: AnyObject?)) -> Void)` - Adds a handler for all events. It will be called on any received event.
4. `emit(event: String, _ items: AnyObject...)` - Sends a message. Can send multiple items.
5. `emit(event: String, withItems items: [AnyObject])` - `emit` for Objective-C
6. `emitWithAck(event: String, _ items: AnyObject...) -> (timeoutAfter: UInt64, callback: (NSArray?) -> Void) -> Void` - Sends a message that requests an acknowledgement from the server. Returns a function which you can use to add a handler. See example. Note: The message is not sent until you call the returned function.
7. `emitWithAck(event: String, withItems items: [AnyObject]) -> (UInt64, (NSArray?) -> Void) -> Void` - `emitWithAck` for Objective-C. Note: The message is not sent until you call the returned function.
8. `connect()` - Establishes a connection to the server. A "connect" event is fired upon successful connection.
9. `connect(timeoutAfter timeoutAfter: Int, withTimeoutHandler handler: (() -> Void)?)` - Connect to the server. If it isn't connected after timeoutAfter seconds, the handler is called.
10. `close()` - Closes the socket. Once a socket is closed it should not be reopened.
11. `reconnect()` - Causes the client to reconnect to the server.
12. `joinNamespace()` - Causes the client to join nsp. Shouldn't need to be called unless you change nsp manually.
13. `leaveNamespace()` - Causes the client to leave the nsp and go back to /
14. `off(event: String)` - Removes all event handlers for event.
15. `removeAllHandlers()` - Removes all handlers.

Client Events
------
1. `connect` - Emitted when on a successful connection.
2. `disconnect` - Emitted when the connection is closed.
3. `error` - Emitted on an error.
4. `reconnect` - Emitted when the connection is starting to reconnect.
5. `reconnectAttempt` - Emitted when attempting to reconnect.

##Detailed Example
A more detailed example can be found [here](https://github.com/nuclearace/socket.io-client-swift-example)

##License
MIT
