#node-ipc
*a nodejs module for local and remote Inter Process Communication*

**npm install node-ipc**

*this is a new project so more documentation will come*

----
#### Types of IPC Sockets

1. ``Local IPC`` Uses ***Unix Sockets*** to give lightning fast communication and avoid the network card to reduce overhead and latency. [Local Unix Socket examples ](https://github.com/RIAEvangelist/node-ipc/tree/master/example/unixSocket/ "Unix Socket Node IPC examples")  
2. ``IPC over TCP`` Uses ***TCP Sockets*** to give the most reliable communication across the network. Can be used for local IPC as well, but is slower than #1's Unix Socket Implementation because TCP sockets go through the network card while Unix Sockets do not. [Local or remote network TCP Socket examples ](https://github.com/RIAEvangelist/node-ipc/tree/master/example/TCPSocket/ "TCP Socket Node IPC examples")
3. ``Remote IPC over TLS`` ***coming soon...***
4. ``Remote IPC over UDP`` Uses ***UDP Sockets*** to give the **fastest network communication**. UDP is less reliable but much faster than TCP. It is best used for streaming non critical data like sound, video, or multiplayer game data as it can drop packets depending on network connectivity and other factors. UDP can be used for local IPC as well, but is slower than #1's Unix Socket Implementation because UDP sockets go through the network card while Unix Sockets do not. [Local or remote network UDP Socket examples ](https://github.com/RIAEvangelist/node-ipc/tree/master/example/UDPSocket/ "UDP Socket Node IPC examples")

----

#### Basic Server Example for Unix Sockets & TCP Sockets 
The server is the process keeping a socket for IPC open. Multiple sockets can connect to this server and talk to it. It can also broadcast to all clients or emit to a specific client. This is the most basic example which will work for both local Unix Sockets and local or remote network TCP Sockets.

    var ipc=require('node-ipc');

    ipc.config.id   = 'world';
    ipc.config.retry= 1500;
    
    ipc.serve(
        function(){
            ipc.server.on(
                'message',
                function(data,socket){
                    ipc.log('got a message : '.debug, data);
                    socket.emit(
                        'message',
                        data+' world!'
                    );
                }
            );
        }
    );
    
    ipc.server.start();

#### Basic Client Example for Unix Sockets & TCP Sockets 
The client connects to the servers socket for Inter Process Communication. The socket will recieve events emitted to it specifically as well as events which are broadcast out on the socket by the server. This is the most basic example which will work for both local Unix Sockets and local or remote network TCP Sockets.

    var ipc=require('../../../node-ipc');

    ipc.config.id   = 'hello';
    ipc.config.retry= 1500;
    
    ipc.connectTo(
        'world',
        function(){
            ipc.of.world.on(
                'connect',
                function(){
                    ipc.log('## connected to world ##'.rainbow, ipc.config.delay);
                    ipc.of.world.emit(
                        'message',
                        'hello'
                    )
                }
            );
            ipc.of.world.on(
                'disconnect',
                function(){
                    ipc.log('disconnected from world'.notice);
                }
            );
            ipc.of.world.on(
                'message',
                function(data){
                    ipc.log('got a message from world : '.debug, data);
                }
            );
        }
    );
    
#### Basic Server & Client Example for UDP Sockets 
UDP Sockets are different than Unix & TCP Sockets because they must be bound to a unique port on their machine to recieve messages. For example, A TCP or Unix Socket client could just connect to a seperate TCP or Unix Socket sever. That client could then exchange, both send and recive, data on the servers port or location. UDP Sockets can not do this. They must bind to a port to recieve or send data.  

This means a UDP Client and Server are the same thing because inorder to recieve data, a UDP Socket must have its own port to recieve data on, and only one process can use this port at a time. It also means that inorder to ``emit`` or ``broadcast`` data the UDP server will need to know the host and port of the Socket it intends to broadcast the data to.

This is the most basic example which will work for both local Unix Sockets and local or remote network TCP Sockets.

##### UDP Server 1 - "World" 

    var ipc=require('../../../node-ipc');

    ipc.config.id   = 'world';
    ipc.config.retry= 1500;
    
    ipc.serveNet(
        'udp4',
        function(){
            console.log(123);
            ipc.server.on(
                'message',
                function(data,socket){
                    ipc.log('got a message from '.debug, data.from.variable ,' : '.debug, data.message.variable);
                    ipc.server.emit(
                        socket,
                        'message',
                        {
                            from    : ipc.config.id,
                            message : data.message+' world!'
                        }
                    );
                }
            );
            
            console.log(ipc.server);
        }
    );
    
    ipc.server.define.listen.message='This event type listens for message strings as value of data key.';
    
    ipc.server.start();
    
##### UDP Server 2 - "Hello"
*note* we set the port here to 8001 because the world server is already using the default ipc.config.networkPort of 8000. So we can not bind to 8000 while world is using it.

    ipc.config.id   = 'hello';
    ipc.config.retry= 1500;
    
    ipc.serveNet(
        8001,
        'udp4',
        function(){
            ipc.server.on(
                'message',
                function(data){
                    ipc.log('got Data');
                    ipc.log('got a message from '.debug, data.from.variable ,' : '.debug, data.message.variable);
                }
            );
            ipc.server.emit(
                {
                    address : 'localhost',
                    port    : ipc.config.networkPort
                },
                'message',
                {
                    from    : ipc.config.id,
                    message : 'Hello'
                }
            );
        }
    );
    
    ipc.server.define.listen.message='This event type listens for message strings as value of data key.';
    
    ipc.server.start();

----

#### IPC Default Variables
Set these variables in the ipc.config scope to overwrite or set default values.

    {
        appspace        : 'app.',
        socketRoot      : '/tmp/',
        id              : os.hostname(),
        networkHost     : 'localhost',
        networkPort     : 8000,
        encoding        : 'utf8',
        silent          : false,
        maxConnections  : 100,
        retry           : 500
    }

* ``appspace`` used for Unix Socket (Unix Domain Socket) namespacing. If not set specifically, the Unix Domain Socket will combine the socketRoot, appspace, and id to form the Unix Socket Path for creation or binding. This is available incase you have many apps running on your system, you may have several sockets with the same id, but if you change the appspace, you will still have app specic unique sockets.
* ``socketRoot`` the directory in which to create or bind to a Unix Socket
* ``id`` the id of this socket or service
* ``networkHost`` the local or remote host on which TCP, TLS or UDP Sockets should connect
* ``networkPort`` the default port on which TCP, TLS, or UDP sockets should connect
* ``encoding`` the default encoding for data sent on sockets
* ``silent`` turn on/off logging default is false which means logging is on
* ``maxConnections`` this is the max number of connections allowed to a socket. It is currently only being set on Unix Sockets. Other Socket types are using the system defaults.
* ``retry`` this is the time in milliseconds a client will wait before trying to reconnect to a server if the connection is lost. This does not effect UDP sockets since they do not have a client server relationship like Unix Sockets and TCP Sockets.

----

#### IPC Methods
documentaion coming soon...
