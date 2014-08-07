nRF8001-Swift
=============

**nRF8001-Swift** was written by **[Michael Teeuw](https://twitter.com/michmich)**

## What is it?
While working on one of my Arduino projects, I was in the need for a simple wrapper for communication with the [Adafruit Bluefruit LE nRF8001 Breakout](https://www.adafruit.com/products/1697).

Since I was unable to find one (especially one that was Swift-ready) I wrote a Swift-wrapper myself. It is based on a [Bluefruit LE Connect](https://github.com/adafruit/Bluefruit_LE_Connect).

It has all the basic needs for basic two way communication and is easy to implement.

![Bluefruit LE nRF8001 Breakout](https://www.adafruit.com/images/970x728/1697-04.jpg)
*Image Courtesy of [Adafruit](https://www.adafruit.com/products/1697)*

## How to use
Add the [NRFManager.swift](https://github.com/MichMich/nRF8001-Swift/blob/master/nRF8001-Swift/NRFManager.swift) file to your project and use the following implementation:

Use the shared instance:


	let nrfManager = NRFManager.sharedInstance

Or simply create an instance yourself:

	let nrfManager = NRFManager()

The Initializer takes 5 arguments. All of these arguments are optional and can be nil:

	let nrfManager = NRFManager(delegate:NRFManagerDelegate? = nil, onConnect connectionCallback:(()->())? = nil, onDisconnect disconnectionCallback:(()->())? = nil, onData dataCallback:((data:NSData, string:String)->())? = nil, autoConnect:Bool = true)

- The **delegate** argument sets the delegate for the NRFManager instance. The delegate must conform to the `NRFManagerDelegate`. Default is `nil`.

- The **onConnect** argument can take a closure to execute when the NRFManager connects to the nRF8001 module. Default is `nil`.

- The **onDisconnect** argument can take a closure to execute when the NRFManager disconnects from the nRF8001 module. Default is `nil`.

- The **onData** argument can take a closure to execute when the NRFManager receives data from the nRF8001 module. This closure will receive a `data:NSData` and `string:String` object. Default is `nil`.

- The **autoConnect** argument will tell the manager if it has to start searching and connect right away, and if it has to reconnect when the connection is lost. Default is `true`.

###Initialization examples:
	// basic object which won't connect
	let nrfBasicExample = NRFManager(autoConnect:false)

	// using delegates
	let nrfDelegateExample = NRFManager(delegate:self)

	// using closures
	let nrfClosureExample = NRFManager(
       onConnect: {
           println("Connected")
       },
       onDisconnect: {
           println("Disconnected")
       },
       onData: {
           (data:NSData, string:String)->() in
           println("Recieved data - String: \(string) - Data: \(data)")
       }
	)


After initialization, it's possible to (re)set the following properties:

- `nrfManager.delegate` to set the delegate.

- `nrfManager.connectionCallback` to set the onConnect callback.

- `nrfManager.disconnectionCallback` to set the onDisconnect callback.

- `nrfManager.dataCallback` to set the onData callback.

- `nrfManager.autoConnect` to set the auto (re)connection behaviour.

- `nrfManager.verbose` to enable or disable logging. This might be helpfull during debugging.

Additionally, the following properties are read-only:

- `nrfManager.connectionMode` returns an enum: *ConnectionStatus* (*.Disconnected*, *.Scanning*, *.Connected*).

- `nrfManager.connectionStatus` returns an enum: *ConnectionMode* (*.None*, *.PinIO*, *.UART*).

## Instance methods
A NRFManager can receive the following method calls:

- `nrfManager.connect()` to start looking for a connection. Only necessary when `autoConnect` is set to `false`.

- `nrfManager.disconnect()` to disconnect from the nRF8001 device. Note that when `autoConnect` is set to `true`, the NRFManager will be an annoying prick and start looking for an new connection instantly ... 

- `nrfManager.writeString(string:String) -> Bool` to send a String to the nRF8001 device. It will return a Bool to tell you if it was possible to send it. (If there is no current connection, this will return `false` in all other cases this will return `true`.)

- `nrfManager.writeData(data:NSData) -> Bool` to send raw data (NSData) to the nRF8001 device. It will return a Bool to tell you if it was possible to send it. (If there is no current connection, this will return `false` in all other cases this will return `true`.)

## Delegate methods
The following methods are part of the NRFManagerDelegate protocol:

- `nrfDidConnect(nrfManager:NRFManager)` is called when the NRFManager connects to the nRF8001 module. The `nrfManager` variable will contain a reference to the NRFManager instance which connected.

- `nrfDidDisconnect(nrfManager:NRFManager)` is called when the NRFManager disconnect from the nRF8001 module. The `nrfManager` variable will contain a reference to the NRFManager instance which disconnected.

- `nrfReceivedData(nrfManager:NRFManager, data:NSData, string:String)` is called when the NRFManager receives data from the nRF8001 module. The `nrfManager` variable will contain a reference to the NRFManager instance which received data. `data` will contain the raw data, `string` will contain the string representation of the received data.

## Delegates AND Closures?!
The reason I implemented both is because I was unable to decide what the best approach is. Personally I tend to prefer the delegate route. 

Although it is possible to both use the closures and the delegates, it wouldn't be the best idea to do so. That being said: Feel free to go wild! ... Both the delegate method and closure will be called when appropriate.

All closures and delegate methods are optional. 

## TL;DR - Give me the most simple example!
Okay okay, you lazy ass ... Here you go!

Using closures:
	
	class ViewController: UIViewController {

		var nrfManager:NRFManager!
		
		override func viewDidLoad() 
		{
			super.viewDidLoad()

			nrfManager = NRFManager(
			  onConnect: {
			      	println("Connected")
					self.sendData()
			  },
			  onDisconnect: {
			      	println("Disconnected")
			  },
			  onData: {
			      	(data:NSData, string:String)->() in
			      	println("Received data - String: \(string) - Data: \(data)")
			  }
			)
		}

		func sendData()
		{
			let result = self.nrfManager.writeString("Hello, world!")
		}
	}
 
Using delegates:

	class ViewController: UIViewController, NRFManagerDelegate {

		var nrfManager:NRFManager!
		
		override func viewDidLoad() 
		{
			super.viewDidLoad()
			nrfManager = NRFManager(delegate:self)
		}

		func sendData()
		{
			let result = self.nrfManager.writeString("Hello, world!")
		}

		// NRFManagerDelegate methods

		func nrfDidConnect(nrfManager:NRFManager)
		{
			println("Connected")
			self.sendData()
		}
		    
		func nrfDidDisconnect(nrfManager:NRFManager)
		{
			println("Disconnected")
		}
		    
		func nrfReceivedData(nrfManager:NRFManager, data: NSData, string: String) {
			println("Received data - String: \(string) - Data: \(data)")
		}
	}

## Yeah, cool! But how do I connect my Bluefruit Breakout Board?

Check Adafruit's [awesome tutorial](https://learn.adafruit.com/getting-started-with-the-nrf8001-bluefruit-le-breakout) on how to use the Bluefruit Breakout Board. You can use the echo example sketch to test out nRF8001-Swift.

## Disclaimer

This is my fist bluetooth experiment. So be gentle. ;)

## Contributing

Forks, pull requests and other feedback are welcome.