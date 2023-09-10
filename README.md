<h1>Expamine alerts, logs and rules with Suricata  </h1>


<h2>Description</h2>
In thia lab I explored suricata alerts and logs, including the general process of rule creation.
 
The Suricata tool monitors network interfaces and applies rules to the packets that pass through the interface. Suricata determines whether each packet should generate an alert and be dropped, rejected, or allowed to pass through the interface.

Source and destination networks must be specified in the Suricata configuration. Custom rules can be written to specify which traffic should be processed.

<h2>Scenario</h2>
In this scenario, I'm a security analyst who monitors traffic on my employer's network. I’ll be required to configure Suricata and use it to trigger alerts.

First, I'll explore custom rules in Suricata. Second, I'll run Suricata with a custom rule in order to trigger it, and examine the output logs in the fast.log file. Finally, I'll examine the additional output that Suricata generates in the standard ```eve.json``` log file.
<br>
For the purposes of the tests in this lab activity, I’ve been supplied with a ```sample.pcap``` file and a ```custom.rules``` file. These reside in your home folder.
<br>
Below I defined the files I was working with in this lab activity:<br>

The ```sample.pcap``` file is a packet capture file that contains an example of network traffic data, which I used to test the Suricata rules. This allowed me to simulate and repeat the exercise of monitoring network traffic.<br>

The ```custom.rules``` file contains a custom rule. I added rules to this file and run them against the network traffic data in the ```sample.pcap``` file.<br>
<br>
The ```fast.log ```file contains the alerts that Suricata generates. Each time I tested a rule, or set of rules, against the sample network traffic data, Suricata adds a new alert line to the ```fast.log``` file when all the conditions in any of the rules are met. The ```fast.log``` file can be located in the ```/var/log/suricata``` directory after Suricata runs. ```The fast.log``` file is considered to be a depreciated format and is not recommended for incident response or threat hunting tasks but can be used to perform quick checks or tasks related to quality assurance.

The ```eve.json``` file is the main, standard, and default log for events generated by Suricata. It contains detailed information about alerts triggered, as well as other network telemetry events, in JSON format. The ```eve.json``` file is generated when Suricate runs, and can also be located in the ```/var/log/suricata``` directory.

When creating a new rule, I needed to test the rule to confirm whether or not it worked as expected. I used the ```fast.log``` file to quickly compare the number of alerts generated each time I ran Suricata to test a signature against the ```sample.pcap``` file.
<br>

<h2>Task 1.Examine a custom rule in Suricata  </h2>

The ```/home/analyst``` directory contains a ```custom.rules``` file that defines the network traffic rules, which Suricata captures.<br>

In this task, I explored the composition of the Suricata rule defined in the ```custom.rules``` file.<br>
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/b1abf5cf-4ecc-46e4-8618-0ad0b85d34b1" height="90%" width="90%" alt="Suricata"/><br>
The command returns the rule as the output in the shell.This rule consists of three components: an <b>action</b>, a <b>header</b>, and <b>rule options</b>.
Below I examined each component in more detail
<br>
<B>Acion</b>
<br>
The action is the first part of the signature. It determines the action to take if all conditions are met.
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/f0cde3e2-154d-486f-93c1-cb9e4a8b3fa9" height="80%" width="80%" alt="Active Directory Lab"/><br>
Actions differ across network intrusion detection system (NIDS) rule languages, but some common actions are alert, drop, pass, and reject.

Using the example, the file contains a single alert as the action. The alert keyword instructs to alert on selected network traffic. The IDS will inspect the traffic packets and send out an alert in case it matches.
Note that the drop action also generates an alert, but it drops the traffic. A drop action only occurs when Suricata runs in IPS mode.
<br>
The pass action allows the traffic to pass through the network interface. The pass rule can be used to override other rules. An exception to a drop rule can be made with a pass rule. For example, the following rule has an identical signature to the previous example, except that it singles out a specific IP address to allow only traffic from that address to pass.
<br>
The reject action does not allow the traffic to pass. Instead, a TCP reset packet will be sent, and Suricata will drop the matching packet. A TCP reset packet tells computers to stop sending messages to each other.
<br>

<b>Header</b><br>
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/bb586b41-582f-4189-999c-32faa9489b50" height="80%" width="80%" alt="Active Directory Lab"/><br>
The next part of the signature is the header. The header defines the signature’s network traffic, which includes attributes such as protocols, source and destination IP addresses, source and destination ports, and traffic direction.

The next field after the action keyword is the protocol field. In our example, the protocol is http, which determines that the rule applies only to HTTP traffic.

The parameters to the protocol ```http``` field are ```$HOME_NET any -> <>$EXTERNAL_NET</i> any```. The arrow indicates the direction of the traffic coming from the ```$HOME_NET``` and going to the destination IP address ```$EXTERNAL_NET```.

```$HOME_NET``` is a Suricata variable defined in ```/etc/suricata/suricata.yaml``` that you can use in your rule definitions as a placeholder for your local or home network to identify traffic that connects to or from systems within your organization.
<br>
In this lab ```$HOME_NET``` is defined as the ```172.21.224.0/20``` subnet. <br>
```any``` means that Suricata catches traffic from any port defined in the ```$HOME_NET``` network.<br>

<b><i>Note:</b>The ```$``` symbol indicates the start of a variable. Variables are used as placeholders to store values.<br>
So far, we know that this signature triggers an alert when it detects any http traffic leaving the home network and going to the external network.

<b>Rule options</b><br>
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/0c16f6a8-c08f-44f6-8b04-96ca68441bc4" alt="Active Directory Lab"/><br>
The many available rule options allow me to customize signatures with additional parameters. Configuring rule options helps narrow down network traffic so I can find exactly what I'm looking for. As in the example, rule options are typically enclosed in a pair of parentheses and separated by semicolons.

 Further examining the rule options in the example:<br>
<br>
The ```msg```: option provides the alert text. In this case, the alert will print out ```“GET on wire”```, which specifies why the alert was triggered.<br>
The ```flow:established,to_server``` option determines that packets from the client to the server should be matched. (In this instance, a server is defined as the device responding to the initial SYN packet with a SYN-ACK packet.)<br>
The ```content:"GET" ```option tells Suricata to look for the word ```GET``` in the content of the ```http.method``` portion of the packet.<br>
The ```sid:12345``` (signature ID) option is a unique numerical value that identifies the rule.<br>
The ```rev:3``` option indicates the signature's revision which is used to identify the signature's version. Here, the revision version is 3.<br>
To summarize, this signature triggers an alert whenever Suricata observes the text ```GET``` as the HTTP method in an HTTP packet from the home network going to the external network.
<br>
<br>
<h2>Task 2.Trigger a custom rule in Suricata  </h2><br>

<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/df55c554-7974-4a13-a649-b0941178af87" height="90%" width="90%" alt="Active Directory Lab"/><br>
1.Before running Suricata, there are no files in the ```/var/log/suricata``` directory. <br>
After starting the Suricata application and processes the ```sample.pcap``` file uses the rules in the ```custom.rules``` file. It returns an output stating how many packets were processed by Suricata.<br>
The ```-r sample.pcap``` option specifies an input file to mimic network traffic. In this case, the ```sample.pcap``` file.<br>
The ```-S custom.rules``` option instructs Suricata to use the rules defined in the ```custom.rules``` file.<br>
The ```-k none``` option instructs Suricata to disable all checksum checks.<br>
<b>Checksums</b> are a way to detect if a packet has been modified in transit. Because I used network traffic from a sample packet capture file, I won't need Suricata to check the integrity of the checksum.<br>
Suricata adds a new alert line to the ```/var/log/suricata/fast.log``` file when all the conditions in any of the rules are met.<br>
<br>
2.Below I listed the files in the ```/var/log/suricata``` folder again:<br>
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/48f39601-00be-45f0-a7bb-09c518869cca" height="60%" width="60%" alt="Active Directory Lab"/><br>
After running Suricata, there are now four files in the ```/var/log/suricata``` directory, including the ```fast.log``` and ```eve.json``` files.<br>
<br>
3.Lets examine these files in more detail:<br>
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/513080d5-ce51-428a-9c14-7cd9d1a980ac" height="90%" width="90%" alt="Active Directory Lab"/><br>
Each line or entry in the ```fast.log``` file corresponds to an alert generated by Suricata when it processes a packet that meets the conditions of an alert generating rule. Each alert line includes the message that identifies the rule that triggered the alert, as well as the source, destination, and direction of the traffic.<br>


<h2>Task 3. Examine eve.json output</h2>

In this task, I examined the additional output that Suricata generates in the ``` eve.json ```
file.<br>
As previously mentioned, this file is located in the ``` /var/log/suricata/ ```
directory.<br>
(screenshot of cat)
1. Using the ``` cat ``` command to display the entries in the ``` eve.json ``` file:
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/51abc9d1-1082-4ccc-9e13-30984905ab16" height="80%" width="80%" alt="Active Directory Lab"/><br>
The output returns the raw content of the file. There is a lot of data returned and is not easy to understand in this format.<br>
2. I then used the ``` jq ``` command to display the entries in an improved format.<br>
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/13dafeea-55bf-490f-9cf1-4f703c837569" height="50%" width="50%" alt="Active Directory Lab"/> <br>
<br>

It is now much easier to read the output as opposed to the ```cat``` command output. <br>
<b>Note:</b>The ```jq``` tool is very useful for processing JSON data, however, a full explanation of its capabilities is outside of the scope of this lab.<br>
<br>
3. Below I used the ```jq``` command to extract specific data from the ```eve.json``` file:
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/13607535-47fc-4fcb-baaa-d389db5dccf2" height="80%" width="80%" alt="Suricata"/> <br>

<b><i>Note:</b> The ``` jq```  command above extracts the fields specified in the list in the square brackets from the JSON payload. The fields selected are the timestamp ``` (.timestamp)``` , the flow id ``` (.flow_id) ```, the alert signature or msg ``` (.alert.signature) ```, the protocol ```(.proto)```, and the destination IP address ```(.dest_ip)```.</i> <br>
The destination IP address is ```142.250.1.102```.<br>

The first event in the ```eve.json``` file has ```."GET on WIRE"```. as the alert signature.<br>

 
4. Using  ```jq``` command to display all event logs related to a specific ```flow_id``` from the ```eve.json``` file. The ```flow_id``` value is a 16-digit number and will vary for each of the log entries. I used the  ```flow_id ``` values returned by the previous query:
<img src="https://github.com/Bridgetanntighe/Suricata/assets/134883216/343a9b39-c30b-4f47-95a1-0bfc8402a232" height="50%" width="50%" alt="Suricata"/><br>

<b><i>Note:</b>A network flow refers to a sequence of packets between a source and destination that share common characteristics such as IP addresses, protocols, and more. In cybersecurity, network traffic flows help analysts understand the behavior of network traffic to identify and analyze threats. Suricata assigns a unique ```flow_id``` to each network flow. All logs from a network flow share the same ```flow_id```. This makes the ```flow_id``` field a useful field for correlating network traffic that belongs to the same network flows.</i>


<h2>Conclusion</h2>

The above activity shows I can use Suricata to trigger alerts on network traffic.<br>
<br>
Running Suricata to:<br>
<br>
- create custom rules and run them in Suricata,<br>
- monitor traffic captured in a packet capture file, and<br>
- examine the ```fast.log``` and ```eve.json``` output.<br>
