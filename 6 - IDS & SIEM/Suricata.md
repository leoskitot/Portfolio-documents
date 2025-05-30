# Scenario

In this scenario, you’re a security analyst who must monitor traffic on your employer's network. You’ll be required to configure Suricata and use it to trigger alerts.

Here’s how you'll do this task: First, you'll explore custom rules in Suricata. Second, you'll run Suricata with a custom rule in order to trigger it, and examine the output logs in the fast.log file. Finally, you’ll examine the additional output that Suricata generates in the standard eve.json log file.

For the purposes of the tests you’ll run in this lab activity, you’ve been supplied with a sample.pcap file and a custom.rules file. These reside in your home folder.

Let’s define the files you’ll be working with in this lab activity:

* The ```sample.pcap``` file is a packet capture file that contains an example of network traffic data, which you’ll use to test the Suricata rules. This will allow you to simulate and repeat the exercise of monitoring network traffic.

* The ```custom.rules``` file contains a custom rule when the lab activity starts. You’ll add rules to this file and run them against the network traffic data in the sample.pcap file.

* The ```fast.log`` file will contain the alerts that Suricata generates. The fast.log file is empty when the lab starts. Each time you test a rule, or set of rules, against the sample network traffic data, Suricata adds a new alert line to the fast.log file when all the conditions in any of the rules are met. The fast.log file can be located in the /var/log/suricata directory after Suricata runs.The fast.log file is considered to be a depreciated format and is not recommended for incident response or threat hunting tasks but can be used to perform quick checks or tasks related to quality assurance.

* The ```eve.json``` file is the main, standard, and default log for events generated by Suricata. It contains detailed information about alerts triggered, as well as other network telemetry events, in JSON format. The eve.json file is generated when Suricate runs, and can also be located in the /var/log/suricata directory.

When you create a new rule, you'll need to test the rule to confirm whether or not it worked as expected. You can use the fast.log file to quickly compare the number of alerts generated each time you run Suricata to test a signature against the sample.pcap file.

### Examine a custom rule in Suricata

The ```/home/analyst``` directory contains a custom.rules file that defines the network traffic rules, which Suricata captures.

In this task, you’ll explore the composition of the Suricata rule defined in the custom.rules file.

Use the cat command to display the rule in the custom.rules file:

```cat custom.rules```

The command returns the rule as the output in the shell:

![image](https://github.com/user-attachments/assets/d31e6ef6-7706-4ced-b31c-a88dc46d6557)

This rule consists of three components: an **action**, a **header**, and **rule options**.

```alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"GET on wire"; flow:established,to_server; content:"GET"; http_method; sid:12345; rev:3;)```

### Action

The **action** is the first part of the signature. It determines the action to take if all conditions are met.

Actions differ across network intrusion detection system (NIDS) rule languages, but some common actions are alert, drop, pass, and reject.

Using our example, the file contains a single alert as the action. The alert keyword instructs to alert on selected network traffic. The IDS will inspect the traffic packets and send out an alert in case it matches.

Note that the drop action also generates an alert, but it drops the traffic. A drop action only occurs when Suricata runs in IPS mode.

The pass action allows the traffic to pass through the network interface. The pass rule can be used to override other rules. An exception to a drop rule can be made with a pass rule. For example, the following rule has an identical signature to the previous example, except that it singles out a specific IP address to allow only traffic from that address to pass:

![image](https://github.com/user-attachments/assets/f90a6361-be30-4001-93e0-b2c553543815)

The reject action does not allow the traffic to pass. Instead, a TCP reset packet will be sent, and Suricata will drop the matching packet. A TCP reset packet tells computers to stop sending messages to each other.

You’ll most often use the alert rule in this lab activity.


### Header

![image](https://github.com/user-attachments/assets/0fa38a0b-3b57-4b84-8544-80a5aee535a5)


The next part of the signature is the header. The header defines the signature’s network traffic, which includes attributes such as protocols, source and destination IP addresses, source and destination ports, and traffic direction.

The next field after the action keyword is the protocol field. In our example, the protocol is http, which determines that the rule applies only to HTTP traffic.

The parameters to the protocol http field are $HOME_NET any -> $EXTERNAL_NET any. The arrow indicates the direction of the traffic coming from the $HOME_NET and going to the destination IP address $EXTERNAL_NET.

$HOME_NET is a Suricata variable defined in /etc/suricata/suricata.yaml that you can use in your rule definitions as a placeholder for your local or home network to identify traffic that connects to or from systems within your organization.

In this lab $HOME_NET is defined as the 172.21.224.0/20 subnet.

The word any means that Suricata catches traffic from any port defined in the $HOME_NET network.

So far, we know that this signature triggers an alert when it detects any http traffic leaving the home network and going to the external network.

### Rule options

![image](https://github.com/user-attachments/assets/4bf64a8a-f900-490e-9b68-e2a06c0d9a32)

The many available rule options allow you to customize signatures with additional parameters. Configuring rule options helps narrow down network traffic so you can find exactly what you’re looking for. As in our example, rule options are typically enclosed in a pair of parentheses and separated by semicolons.

Let's further examine the rule options in our example:

* The ```msg:``` option provides the alert text. In this case, the alert will print out the text ```“GET on wire”```, which specifies why the alert was triggered.
  
* The ```flow:established,to_server``` option determines that packets from the client to the server should be matched. (In this instance, a server is defined as the device responding to the initial SYN packet with a SYN-ACK packet.)
* The ```content:"GET"``` option tells Suricata to look for the word ```GET``` in the content of the ```http.method``` portion of the packet.
* The ```sid:12345``` (signature ID) option is a unique numerical value that identifies the rule.
* The ```rev:3``` option indicates the signature's revision which is used to identify the signature's version. Here, the revision version is 3.
  
To summarize, this signature triggers an alert whenever Suricata observes the text GET as the HTTP method in an HTTP packet from the home network going to the external network.


### Trigger a custom rule in Suricata


1. List the files in the /var/log/suricata folder:
   ```ls -l /var/log/suricata```
   ![image](https://github.com/user-attachments/assets/6326641c-314b-4a29-ad53-8449f71637d2)
   
2. Run suricata using the custom.rules and sample.pcap files:
   ```sudo suricata -r sample.pcap -S custom.rules -k none```
![image](https://github.com/user-attachments/assets/5981f426-48d8-4245-bef4-e0fe4661ff1b)

This command starts the Suricata application and processes the sample.pcap file using the rules in the custom.rules file. It returns an output stating how many packets were processed by Suricata.

Now you’ll further examine the options in the command:

* The ```-r sample.pcap``` option specifies an input file to mimic network traffic. In this case, the ```sample.pcap``` file.
* The ```-S custom.rules``` option instructs Suricata to use the rules defined in the custom.rules file.
* The ```-k none``` option instructs Suricata to disable all checksum checks.

As a refresher, checksums are a way to detect if a packet has been modified in transit. Because you are using network traffic from a sample packet capture file, you won't need Suricata to check the integrity of the checksum.

Suricata adds a new alert line to the ```/var/log/suricata/fast.log``` file when all the conditions in any of the rules are met.

3. List the files in the /var/log/suricata folder again:
   ```ls -l /var/log/suricata```
![image](https://github.com/user-attachments/assets/84ba72ff-f0c8-4186-8db4-4cc6519c0553)

4. Use the cat command to display the fast.log file generated by Suricata:

   ```cat /var/log/suricata/fast.log```
   
![image](https://github.com/user-attachments/assets/6c0a0fb4-9ede-4179-8cf9-a89777eec11f)

Each line or entry in the fast.log file corresponds to an alert generated by Suricata when it processes a packet that meets the conditions of an alert generating rule. Each alert line includes the message that identifies the rule that triggered the alert, as well as the source, destination, and direction of the traffic.

###  Examine eve.json output

The eve.json file is the standard and main Suricata log file and contains a lot more data than the fast.log file. This data is stored in a JSON format, which makes it much more useful for analysis and processing by other applications.

1. Use the cat command to display the entries in the eve.json file:
   ```cat /var/log/suricata/eve.json```
   ![image](https://github.com/user-attachments/assets/e8e3b787-384a-42fd-bdd4-86b23bab5249)
   
The output returns the raw content of the file. You'll notice that there is a lot of data returned that is not easy to understand in this format.
   
2. Use the jq command to display the entries in an improved format:
   
   ![image](https://github.com/user-attachments/assets/3079fd8b-bee6-40e4-a33f-98d61df96893)
   
3. Press Q to exit the less command and to return to the command-line prompt.
   
4. Use the jq command to extract specific event data from the eve.json file:
  ```jq -c "[.timestamp,.flow_id,.alert.signature,.proto,.dest_ip]" /var/log/suricata/eve.json```

![image](https://github.com/user-attachments/assets/ca8f8449-dccb-4734-bc2b-679e649de3e2)

5. Use the jq command to display all event logs related to a specific flow_id from the eve.json file. The flow_id value is a 16-digit number and will vary for each of the log entries. Replace X with any of the flow_id values returned by the previous query:
   ```jq "select(.flow_id==X)" /var/log/suricata/eve.json```
![image](https://github.com/user-attachments/assets/7a6637b2-cb0b-4210-a590-99ba0c174d7d)

   


