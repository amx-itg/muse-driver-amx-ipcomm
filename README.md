# AMX-IPComm
The *IP Communications Provider* service provides a suite of IP Communications Drivers enabling the instancing of IP Controlled entities as Devices; allowing access to the Device instance(s) from the Mojo context in a script.  This Driver Provider is distributed as a Muse Extension and the capabilities and configuration options of each of the Drivers is listed below.


## Available Drivers
### Minimal Configuration Drivers
- TCP Client (Persistent) 
  - id:*ipcomm.tcp.client.persistent*
  - Establishes and maintains a persistent TCP client connection at the specified address.
  - since 1.0.0
- TCP/TLS Client (Trusted Persistent)  
  - id:*ipcomm.tls.client.persistent*
  - Establishes and maintains a trusted persistent TLS/TCP client connection at the specified address.
  - since 1.0.0
- UDP Transmitter  
  - id:*ipcomm.udp.client.transmit*
  - Allows for sending UDP Datagram packets to the specified target.
  - since 1.0.0
- UDP Receiver  
  - id:*ipcomm.udp.server*
  - Enables receiving UDP Datagram packets on the specified port.
  - since 1.0.1
- SSH Client (Password Persistent) 
  - id:*ipcomm.ssh.client.persistent*
  - Establishes and maintains a persistent SSH client connection authenticated with user password at the specified address.
  - since 1.0.1
- WS Client (Persistent)
  - id:*ipcomm.ws.client.persistent*
  - Establishes and maintains a persistent WS client connection at the specified address.
  - since 1.0.2
- WSS Client (Trusted Persistent)
  - id:*ipcomm.wss.client.persistent*
  - Establishes and maintains a trusted persistent WSS client connection at the specified address.
  - since 1.0.2 
  
### Advanced Configuration Drivers
- TCP/TLS Client 
  - id:*ipcomm.tcp.client*
  - TCP or TCP/TLS Client connection as per configuration. 
  - The connection can be fully managed from a script. 
  - since 1.0.1
- UDP Client  
  - id:*ipcomm.udp.client*
  - UDP Client connection as per configuration.  
  - The connection can be fully managed from a script.
  - Supports bound or unbound receiver port.
  - since 1.0.1
- SSH Client
  - id:*ipcomm.ssh.client*
  - SSH Client connection as per configuration.  
  - Supports password and/or passkey user authentication.
  - The connection can be fully managed from the a script.
  - since 1.0.1
- WS/WSS Client
  - id: *ipcomm.ws.client*
  - WS or WSS Client connection as per configuration.
  - The connection can be fully managed from a script.
  - since 1.0.2


### Configuration
#### Common Parameters
- Host
  - The IP address/hostname of the destination.
- Port
  - The port number of the target service.
- Timeout
  - The connection timeout.
- Reconnect Time
  - The time delay before reconnecting after disconnection from target detected or connecting after device start.
  - A value of -1 disables the feature and the connection must be maintained at the script.
- Keep ALive Interval
  - The time interval checking the connection and reconnecting if disconnected.
  - A value of -1 disables the feature and the connection must be maintained at the script.

#### Advanced Configuration Parameters
##### TCP/TLS Client Parameters
- TLS Enable 
  - Enables TLS to target.
- Trusted Certificate 
  - The Trusted Certificate as the alias of the uploaded certificate to the CA-Cert Domain or the certificate as a PEM formated string (64-Bit encoded).

##### SSH Client Parameters
- User Name
  - The username credential for the service.
- Password
  - The password credential for the service.
- Private Key
  - The private key credential for the service.
  - The following encryption algorithms are supported:
   	- AES-256-CBC
   	- AES-192-CBC
   	- AES-128-CBC
   	- DES-EDE3-CBC
   	- DES-CBC
- Key Passphrase
  - The Passphrase for the Private Key credential.
  - __The Passphrase is mandatory and cannot be an empty string.__
- Buffer Size
  - The connection buffer size.
- Legacy Cipher Keys
  - Enables support of legacy cipher keys.
  - __The security posture will be degraded when legacy support enabled.__ 
  - The default (legacy support disabled) cipher keys are:
  	  - ecdh-sha2-nistp521,
	  - ecdh-sha2-nistp384,
	  - ecdh-sha2-nistp256,
	  - diffie-hellman-group-exchange-sha256,
	  - diffie-hellman-group18-sha512,
	  - diffie-hellman-group17-sha512,
	  - diffie-hellman-group16-sha512,
	  - diffie-hellman-group15-sha512,
	  - diffie-hellman-group14-sha256,
	  - ext-info-c
- Session Idle Time
  - The session idle timeout.
- Authentication Timeout
  - The authentication timeout.
  
##### UDP Client Parameters
- Bound Port
  - The port enabled for receiving data.  
  - A value of 0 binds the receiving port with the transmitting port.  
  - A value of -1 disables the receiver.

##### WS Client Paramaters
- Context Path
  - The path to the WebSocket Context at the target device.
- Custom Headers
  - Custom headers attached to Upgrade Request.
  - Property is set as array and string item: <key>=<value>.
- Proxy 
  - Proxy address for the WebSocket service.

### Common Driver API
#### Commands
- send
  - Sends the provided message (*byte[] data argument*) to the specified target.
  - The optional *cast* argument will send data in an alternate format as specified in the metada description.
  
#### Events  
- receive
  - Message received (*<byte[]>data argument*) from the sending entity (*<String>source argument (when applicable)*).

### Advanced Driver API
#### Parameters
- connect
  - Manage the connection from a script.
  
#### Commands
- deviceonline
  - Sets the device online status when managing the connection from a script.
  
#### Events 
- fault
  - Notification of error event or thrown exception in the Driver.
  - Arguments
    - fault: Error or exception message when thrown. 
    - exception: Exception class when thrown.
   
    | Exception Class                 | Description                                                                                     |
	| ------------------------------- | ------------------------------------------------------------------------------------------------ |
 	| AlreadyConnectedException       | This channel is already connected                                                                |
	| ConnectException                | An error occurred while attempting to connect a socket to a remote address and port              |
 	| ConnectionPendingException      | A non-blocking connection operation is already in progress on this channel                       |
 	| ClosedChannelException          | This channel is closed                                                                           |
 	| UnresolvedAddressException      | The given remote address is not fully resolved                                                   |
 	| UnsupportedAddressTypeException | The type of the given remote address is not supported                                            |
 	| SecurityException 		      | A security manager has been installed and it does not permit access to the given remote endpoint |
 	| IOException                     | Some other I/O error occurs                                                                      | 	
  - since 1.0.4

> [!WARNING] 
> Ungraceful disconnects of a TCP/TLS connection may not register for up to 15-minutes after losing the connection.  
>
> Implementing device polling in the script will expedite the lost connection notification.
  
## Basic Driver Examples

> Groovy

```
svsiEnc = context.devices.get("tcpclient");

context.log.warn("test start");

svsiEnc.receive.listen({ 
    context.log.info("data: {}",new String(it.arguments.data));
});

t1 = context.services.get("timeline");
t1.start([10000], true);
t1.expired.listen({
    context.log.info("polling");

    var msg = "getStatus\r";
    svsiEnc.send(msg.getBytes());
});

```

> JavaScript

```
var svsiEnc = context.devices.get("tcpclient");

context.log.warn("test start");

svsiEnc.receive.listen(function(event) {
    context.log.info("data: "+String(event.arguments.data));
});

var t1 = context.services.get("timeline");
t1.start([10000], true);
t1.expired.listen(function(){
    context.log.info("polling");

    var msg = "getStatus\r";
    svsiEnc.send(msg);
});


```

> Python 

```
def statusResponse(event):
    data = event.arguments['data'].decode('utf-8')
    context.log.info(f"data: {data}")  

svsiEnc = context.devices.get("tcpclient")
svsiEnc.receive.listen(statusResponse)

def getStatus(event):
    context.log.info("polling")
    svsiEnc.send("getStatus\r".encode('utf-8'))

t1 = context.services.get("timeline")
t1.start([1000], True, 0)
t1.expired.listen(getStatus)
```

## Advanced Driver Examples

> Python 

```
def statusResponse(event):
    context.log.info(str(event.__dict__))  
    test.connect = False

def onError(event):
    fault = event.arguments['fault']
    context.log.info(f"fault: {fault}")  
    exception = event.arguments['exception']
    context.log.info(f"exception: {exception}")  
    test.connect = False 

test = context.devices.get("tcpclient")
test.receive.listen(statusResponse)
test.fault.listen(onError)

def onConnect(event):
    context.log.info(str(event.__dict__))
    test.deviceonline(event.newValue)
    
test.connect.watch(onConnect)

def getStatus(event):
    context.log.info("polling")
    test.send("getStatus\r".encode('utf-8'))

test.online(getStatus)

def connect(event):
    context.log.info("connect")
    if(test.connect == False):
        test.connect = True
    else:
        test.send("getStatus\r".encode('utf-8'))

t1 = context.services.get("timeline")
t1.start([10000], True)
t1.expired.listen(connect)
```

