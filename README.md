# Web Socket Server Side Client

## Description
A web socket server side client for web socket process connector servers. Use this client to connect to a web socket server and send and receive messages.

USE THIS CLIENT TO CONNECT TO A WEB SOCKET SERVER SET WITH 

- [websocket node server](https://www.npmjs.com/package/ws-node-server) 
- [web socket process connector](https://www.npmjs.com/package/web-socket-processes-connector)

## Installation
```bash
npm install --save ws-server-side-client
```
## Connection Setup
```typescript
import { WebSocketServerSideClient } from "ws-server-side-client";

let options = null;
/*  default values
    options = {
        onConnectionErrorReconnect: true,
        authCallbackOnReconnect:true,
        reconnectionTimeout: 2_000
    }
*/

let Logger = null;
/*  default values
    Logger = {
        log: console.log,
        error: console.error,
        warn: console.warn,
        info: console.info,
        debug: console.debug
    }

    use this to log all incoming messages
*/
export const wsClient = new WebSocketBrowserClient(options,Logger);

let authCredentials = {
    //... your credentials
}

// LISTENERS SETUP SECTION
// set what to do if authentication is successful
wsClient.whenConnected = () => {
    console.log('WebSocketClient Connected');
    // ... now you can use the client in other parts of your application
    AfterConnectedProcedure();
};
// set what to do if authentication fails
wsClient.ifAuthenticationFails = (authenticationError) => {
    console.error({authenticationError});
}
// set what to do if connection is lost
wsClient.onConnectionError = (connectionLostError,connectionLostInfo) => {
    console.error({connectionLostError,connectionLostInfo});    
}

wsClient.onConnectionClosed = (connectionCloseError,connectionCloseEvent) => {
    console.log({connectionCloseError,connectionCloseEvent});
}
// execute the connection to the server
wsClient.connectTo('ws://localhost:8080',authCredentials);
```
You can rewrite the `whenConnected`, `ifAuthenticationFails` and `onConnectionError` methods anytime before the `connectTo` method is called.
## API: After Connection Setup
```typescript
interface User { }
interface NewUserResponse { }
let globalUsers: User[] = [];

const AfterConnectedProcedure = () => {
    // after connection is done, you can read de session data return by the server
    let sessionData = wsClient.session
    // send a echo message to the server and wait for a response
    wsClient.echo({msg:'testing connection ...'},(error,response) => {
        console.log({error,response});
    });
    // send a request message to the server and wait for a response to get an array of users
    wsClient.request<User[]>('getUsers',{},(error,users) => {
        if(error) {
            console.log('Error:',error);
            return;
        } else {
            globalUsers = users;
        }
    });
    // join the group1 to receive messages from the server for this group
    wsClient.joinGroup('group1');
    // leave the group1
    wsClient.leaveGroup('group1');
    // leave all groups
    wsClient.leaveAllGroups();
    // close the connection
    wsClient.close();
}

// you can set a onMessageReceived listener, 
// to receive messages from the server 
// before or after the connection setup is done
wsClient.onMessageReceived<User>('newUser', globalUsers.push);
```

## API: Example
As long as the connection is open, you can send messages to the server and receive messages from the server in any part of your application.
```typescript

import http from 'http';
import express from 'express';
import { CreateServerSocket } from '../server.socket';
import { CreateClientSocket } from '../client.socket';
const app     = express();

import { WebSocketServerSideClient } from "ws-ss-client";

export const wsClient = new WebSocketServerSideClient();

let authCredentials = {
    //... your credentials
}

//LISTENERS SETUP SECTION ...

// connection to remote server
wsClient.connectTo('ws://remoteServer:7777',authCredentials);

//... your express app configuration, validations, routes, middleware, etc
app.get('/create-user', (req, res) => {
    let newUser: User = req.body;
    // pass the request to the remote server
    wsClient.request<NewUserResponse,User>('createUser',newUser,(error,response:NewUserResponse) => {
        if(error) {
            res.send({error,response:null});
            return;
        } else {
            res.send({error:null,response});
        }
    });
});

const server = http.createServer(app);
server.listen(8000, () => console.log('Listening on http://localhost:8000'));

```


## READ THE CODE ON

[github: websocket-server-side-client](https://github.com/ceduardorubio/websocket-server-side-client)

## License

[MIT](LICENSE)

## Author

Carlos Velasquez - [ceduardorubio](https://github.com/ceduardorubio)

## Keywords

[websocket](https://www.npmjs.com/search?q=keywords:web%20socket), [websocket client](https://www.npmjs.com/search?q=keywords:websocket%20client), [websocket server client](https://www.npmjs.com/search?q=keywords:websocket%server%20client), [websocket client server](https://www.npmjs.com/search?q=keywords:websocket%20client%20server)

## Change Log

### 0.0.1
    - Initial release
### 0.0.4
    - Fix type module error