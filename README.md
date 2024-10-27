# Azure-SIEM-Detection-Lab
Azure Sentinel detection lab with LIVE CYBER ATTACKS!

## Objective
My goal of this lab is to set up a SIEM (Azure Sentinal) and connect it to a live virtual machine posing as a honey pot. I will observe live attacks (RDP Brute Force) from all around the world. I will use a custom powershell script tp look up the attackers geolocation information and plot it on the Azure Sentinel map. 


### Skills Learned

- Developed proficiency in setting up and managing Azure Sentinel as a Security Information and Event Management (SIEM) tool. Gained 
  experience in configuring data connectors to ingest security logs from a virtual machine.
- Learned to identify and analyze security threats, specifically focusing on RDP brute force attacks. This included recognizing patterns 
  and indicators of compromise (IoCs) within the data collected.
- Developed skills in utilizing geolocation services to map the origin of cyber attacks, integrating these insights into Azure Sentinel 
  to visualize threat landscapes and enhance situational awareness.
- Gained insights into cloud security principles, particularly how they apply to Azure and the importance of securing virtual 
  environments against emerging threats.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- Windows 10 Pro virtual machine
- Azure Sentinel 

## Steps

- First, i will construct the Virtual enviornment in Azure. I will start by creating a Windows 10 virtual machine. This will 
  be the "exposed machine" in which attackers will try to gain access to. I will name the resource group "Honeypotlab" and name the 
  VM "Honeypot".

   ![image](https://github.com/user-attachments/assets/4ce8a96b-3a0e-4bd4-a726-29eb4d8aaa4b)

- I will move on to the network configuration and set a new custom inbound firewall rule that essentially allows any source and any 
  port to come into the VM. I will name the rule "DANGER_ANY_IN". The purpose of this is to make the VM discoverable and enticing for 
  people to attack.

   ![image](https://github.com/user-attachments/assets/651e585c-c54c-44f3-b171-1e6fb6b001fd)

- While the virtual machine is creating, i will go ahead and create a log analytics workspace. The purpose of this is to ingest logs 
  from the virtual machine. I will ingest windows event logs and create a custom log that contains geographic information so i can 
  discover where the attackers are coming from. To do this, i will start by heading to the log analytics workspace in Azure. This is 
  where those logs will be stored and where the SIEM will connect to in order to display the geo data on the map. I will add the log 
  analytic workspace to the same resource group as the VM (Honeypotlab) and name the workspace "law-honeypot".

   ![image](https://github.com/user-attachments/assets/730d84bb-79cd-48c9-a4dd-d8c58ca888f6)

- I will then set the data collection settings to make sure all data gets collected. 

   ![image](https://github.com/user-attachments/assets/89cbdc3e-6eed-471f-a31e-48d6259eaf55)

- The next step is to connect the VM as the destination for my logs.

   ![image](https://github.com/user-attachments/assets/1fc9f4dc-9021-43e1-9ded-4982079f8e50)

- From here, i will now setup the SIEM (Sentinel) and add it to my workspace.

   ![image](https://github.com/user-attachments/assets/b547b19e-016d-4062-af55-a7cec25b0587)

- Now that the SIEM is added to the workspace, i will now connect to the VM through Remote Desktop Connection on my local computer by 
  copying the ip address of the VM and logging in.

   ![image](https://github.com/user-attachments/assets/57c21279-e586-4a4a-bad6-e2e1e080f6c5)

   ![image](https://github.com/user-attachments/assets/b1492b2a-9dbd-4fb8-8a9c-7012357268d8)

- Once i get logged in, i will open up the command prompt and ping my VM at 13.88.189.199 to check the firewall status.

   ![image](https://github.com/user-attachments/assets/ed10f32c-2f00-41a9-8bd8-71598296d85b)

- I see that the request has timed out which means that the firewall is on and blocking the ping request. Now i will go ahead and 
  turn the firewall off to let the attacks come in.

   ![image](https://github.com/user-attachments/assets/8c4988db-42e2-45bc-824d-6bb06c8e329e)

- Now i will ping the VM now that the firewall is off and make sure that the ping requests come through. 
   
   ![image](https://github.com/user-attachments/assets/3c85a846-f4d1-4edd-9f2a-125507b07eeb)

- I will then go grab a powershell script from Github. The script has 2 parts to it. First, it will look through the security log 
  from event viewer and export all of the failed logon events. Then, it grabs the geo location data from their ip address and 
  creates a new log file. The script will also send the ip addresses to an ip geolocation API (ipgeolocation.io).

   ![image](https://github.com/user-attachments/assets/a0c5f64d-f288-4fff-bbbd-d346a5e58014)

- I will now run the script in powershell so that the it will start listening for failed logon events and log them to a file once 
  found.
   ![image](https://github.com/user-attachments/assets/94c965c6-946a-4209-8c4b-51af11e14051)

- Almoast immediately, i can see a whole bunch of failed login attemts start to flood in with their associated geo location.

   ![image](https://github.com/user-attachments/assets/3618b3c7-54a2-4f9a-9809-7e93c6326617)

- Now i will head back over to Azure log analytics page to create a custom log. I will copy the log file from the VM to a text file 
  on my local computer and upload it to the custom log in azure to train log analytics for what to look for in the log file. 

  ![image](https://github.com/user-attachments/assets/37dbfa74-decb-4a83-a99f-1a53e3277f52)

- I will then add the log file from the virtual machine to link the custom log from the VM to my log analytic workspace. I will do 
  this by copying the log file path from the VM and pasting it into the custom log in Azure.

   ![image](https://github.com/user-attachments/assets/a66b578e-54a4-4cee-ac22-ab4f29411446)

- Here, we can see that the log analytics work space and the VM have synced up and brought the log into Log analytics workspace.

   ![image](https://github.com/user-attachments/assets/eaefb007-6dd6-40bc-83c7-07653009367c)

- As shown above, the raw data colum contains all of the custom log entry data from the custom log file in the VM (geo location). I 
  want to take the raw data and extract the certain fields from it to show up in the SIEM. 

- To do that, i will head over to Azure Sintenel (SIEM)

   ![image](https://github.com/user-attachments/assets/2cfb587f-6019-4594-b07c-cf45eaae6b29)

- Azure Sentinel allows you to use Kusto Query Language (KQL) to query the logs. I will add a new workbook in Sentinel to add a 
  query to select failed RDP attempts and extract the ip addresses and geo location associated with these attempts. I will use a 
  query from Github that will allow me to extract those fields. Once i input the query, i will change the visualization option to 
  "Map"
   ![image](https://github.com/user-attachments/assets/f78685e1-f0fd-4a77-8f8f-16c2e2e0b8bd)

- I will now run the query and see the map of all of the attacks carried out on the honeypot mapped out across the globe. 

  ![image](https://github.com/user-attachments/assets/a00e91c7-e148-45a5-9545-cb3480500b75)
 WOW!

- Each point represents a failed attempt to breach my VM giving me invaluable insights into the global distribution of these threats. 
- After letting the honeypot collect data for awhile, its time to dive into the analysis.


## Conclusion

- This map is not just a collection of data points, its a window into the tactics and locations of potential attackers. You will 
  notice concentration of attacks emenating from certain regions. This could be indicitive of where cyber threats are most active or 
  where specific vulnerabilities are being exploited on mass. By analyzing the timing of these attacks, we can identify peak periods 
  of activity. This information is crucial for bolstering our defences where we are most vulnerable. Another interesting observation 
  is the frequency of attacks from specific ip addresses or ranges. Identifying repeat offenders can help us understand their tactics 
  and potentially block these persistent threats. Utilizing Azure Sentinel to visualize and analyse security data provides us with 
  the clarity needed to understand and mitigate cyber threats. The honeypot has not only attracted attackers, but has given the 
  isights needed to strengthen our defenses.
    

   




   
  

   

