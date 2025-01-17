# Network Configuration

## overview

Nodes in the network are configured to receive messages from 2 sources:  

a. From users and applications using a REST API.  
This functionality is enabled by calling the command: ***run rest server***  

b. From peer nodes using the native AnyLog API.  
This functionality is enables by calling the command: ***run tcp server***  

## Determining the IP addresses recognized by a node in the network

When a node starts, it determines the local IP addresses available to the node. These addresses initialize 2 dictionary variables:  

| Variable Name | Explanation   | Retrieve value |
| ------------- | ------------- | ------------- |
| ip            | The Public IP or a Local IP if the node is on a local network | !ip |
| external_ip   | The Public IP | !external_ip |


Some systems fail to identify their IP addresses, and the values can be set by the user.  
To view all IPs recognized by the node issue the following command:
<pre>
get ip list
</pre> 

## Configuring a REST server process
Any node in the network can be configured to receive requests using the REST API.  
Usage:
<pre>
run rest server [ip] [port] where timeout = [timeout] and threads = [threads count] and ssl = [true/false]
</pre>
More details on the REST API are available at [using rest](https://github.com/AnyLog-co/documentation/blob/master/using%20rest.md#using-rest).  
More details on setting a node as a REST server is available at [background processes](https://github.com/AnyLog-co/documentation/blob/master/background%20processes.md#rest-requests).  

## Configuring a TCP server process
This process makes each node a member in the AnyLog Network and allows communication between peers in the network.  

Usage:
<pre>
run tcp server [ip] [port] [threads]
</pre>
   
***[ip] [port]*** - The IP and Port of the socket that is in the listening state and accessible by peer nodes in the AnyLog Network.   
***[local ip] [local port]*** - Optional parameters to indicate an IP and Port that are accessible from a local network.  
***[threads]*** - An optional parameter for the number of workers threads that process requests which are send to the provided IP and Port. The default value is 6.

Additional information is available at [network configuration](https://github.com/AnyLog-co/documentation/blob/master/network%20configuration.md). 

### The Source Address
When the command ***run tcp server*** is initiated, the node dedicates a process to listen to incoming messages on the declared IP and Port.
When the node sends a message to a peer, it requests that the reply is send to the declared IP and Port.  
Therefore, for an outgoing message, the declared IP and Port serve as a Source Address identifying the node that issued the message and an address for a reply message.   
 
A user can configure a node to use a different reply address in one of the following ways:

a) Request peers to reply using a different address.  
b) Request peers to determine the reply address from the message socket.

## Self messaging

For self-messaging, nodes use their configured Local IP address.  
In some setups (i.e., Kubernetes), a self-message does not work as it is needed to assign a different address for self-messaging.  
Using the ***set self ip*** command, a self-message is directed to use a different IP address than the configured address.
In this case, the port remains the same as the port configured for the local address (using the ***run tcp server*** command).
Therefore, this command needs to be set only if the TCP server is configured.  
Using the ***set self ip and port*** command, a self message is directed to use a different address than the configured address.   

if the keyword ***dynamic*** is used, the machine's local IP is used for self-messaging.

usage:
<pre>
set self ip = [ip]
set self ip and port = [IP:Port]
</pre>

Examples:
<pre>
set self ip = dynamic
set self ip = 10.0.0.178
set reply ip and port = 10.0.0.178:4078
set reply ip and port = dynamic:4078
set reply ip and port = !self_ip:!self_port
</pre>

### Reset self messaging
Calling reset will disable the use of self IP.  
Usage:
<pre>
reset self ip
</pre>


## Setting a different IP address for replies 
Using the ***set reply ip*** command, user can direct a node sending a message, to receive the reply on a different IP address.    
Using the ***set reply ip and port*** command, user can direct a node sending a message, to receive the reply on a different IP and port address.  

Usage:
<pre>
set reply ip = [ip]
set reply ip and port = [IP:Port]
</pre>

Examples:
<pre>
set reply ip = !external_ip
set reply ip = 24.23.250.144
set reply ip and port = 24.23.250.144:4078
</pre>

The value assigned to the ***reply ip*** can be retrieved using the following command:
<pre>
get reply ip
</pre>

### Using the message socket to determine the reply IP 
This configuration will retrieve the peer IP from the message socket and use the retrieved IP for the reply message.  
Usage:
<pre>
set reply ip = dynamic
</pre>

### Reset the reply IP to the Source IP 
Calling reset will disable the use of reply IP. Replies will use the Source IP.  
Usage:
<pre>
reset reply ip
</pre>


# Testing the network configuration

Different command calls can view and test the network configuration.  

View active connection using the command:
<pre>
get connections
</pre>

Test the node configuration including the status of the REST server and the TCP server using the command:
<pre>
test node
</pre>
 
Test connection between 2 peers in the network:
<pre>
run client (host:port) get status
</pre>

Test the REST server configuration:
<pre>    
rest get url =  http://ip:port type = info details = "get status"
</pre>  
Example:
<pre>
rest get url =  http://10.0.0.159:2049 type = info details = "get status"
</pre>

DIsplay the IP and Port used in a message:
<pre>
trace level = 1 tcp
</pre>

DIsplay the IP and Port used in a REST message by sending a VIEW request to the REST server.    
Example:
<pre>
curl --location --request VIEW 24.23.250.144:2049
</pre>
