# muse-driver-amx-ipcomm
The *IP Communications Provider* service provides a suite of IP Communications Drivers enabling the instancing of IP Controlled entities as Devices; allowing access to the Device instance(s) from the Mojo context in a script.  The capabilities and configuration options of each of the Drivers is listed below.


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
  - since 1.0.1
- UDP Client  
  - id:*ipcomm.udp.client*
  - UDP Client connection as per configuration.  
  - Supports bound or unbound receiver port.
  - since 1.0.1
- SSH Client
  - id:*ipcomm.ssh.client*
  - SSH Client connection as per configuration.  
  - Supports password and/or passkey user authentication.
  - since 1.0.1
- WS/WSS Client
  - id: *ipcomm.ws.client*
  - WS or WSS Client connection as per configuration.
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
  - Manages the connection.
  
#### Commands
- deviceonline
  - sets the device online status.
  
#### Events 
- fault
  - Notification of error event in the Driver.
  
  
## Basic Driver Examples

> Groovy

```
svsiEnc = context.devices.get("tcpclient");

context.log.warn("test start");

svsiEnc.receive.listen({ 
    context.log.info("{}",new String(it.arguments.data));
});

svsiEnc.fault.listen({  
    context.log.info("{}",new String(it.arguments.fault));
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
    context.log.info(String(event.arguments.data));
});

svsiEnc.fault.listen(function(event) {
    context.log.info(String(event.arguments.fault));
});

var t1 = context.services.get("timeline");
t1.start([10000], true);
t1.expired.listen(function(){
    context.log.info("polling");

    var msg = "getStatus\r";
    svsiEnc.send(toUTF8Array(msg));
});

function toUTF8Array(str) {
    var utf8 = [];
    for (var i=0; i < str.length; i++) {
        var charcode = str.charCodeAt(i);
        if (charcode < 0x80) utf8.push(charcode);
        else if (charcode < 0x800) {
            utf8.push(0xc0 | (charcode >> 6), 
                      0x80 | (charcode & 0x3f));
        }
        else if (charcode < 0xd800 || charcode >= 0xe000) {
            utf8.push(0xe0 | (charcode >> 12), 
                      0x80 | ((charcode>>6) & 0x3f), 
                      0x80 | (charcode & 0x3f));
        }
        // surrogate pair
        else {
            i++;
            // UTF-16 encodes 0x10000-0x10FFFF by
            // subtracting 0x10000 and splitting the
            // 20 bits of 0x0-0xFFFFF into two halves
            charcode = 0x10000 + (((charcode & 0x3ff)<<10)
                      | (str.charCodeAt(i) & 0x3ff));
            utf8.push(0xf0 | (charcode >>18), 
                      0x80 | ((charcode>>12) & 0x3f), 
                      0x80 | ((charcode>>6) & 0x3f), 
                      0x80 | (charcode & 0x3f));
        }
    }
    return utf8;
}
```

> Python 

```
def statusResponse(event):
    context.log.info(str(event.__dict__))  

def onError(event):
    context.log.info(str(event.__dict__))  

svsiEnc = context.devices.get("tcpclient")
svsiEnc.receive.listen(statusResponse)
svsiEnc.fault.listen(onError)

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
    context.log.info(str(event.__dict__))  
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


