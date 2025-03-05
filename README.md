# f5-apm-google-2fa
How to use Google 2FA on F5 APM

I struggle a bit when trying to find examples on how to use Google 2FA with APM. There were a few examples on the web, but most missing the important bits. 
The best reference I could find is in https://loadbalancing.se/2016/07/09/setting-up-apm-with-google-authenticator/ - yes, almost 10 years old...!
The required iRules are presented in the above tutorial, but they are outdated in that the QR code for enrolment wasn't being displayed. 
This repo is an update of the tutorial found at https://loadbalancing.se/2016/07/09/setting-up-apm-with-google-authenticator/. It also has the required iRules, updated by me. 

From https://loadbalancing.se/2016/07/09/setting-up-apm-with-google-authenticator/:

Steps to get this to work:

1) On the APM BIG-IP, create the required iRules (DON'T CHANGE THEIR NAMES!)

2) To generate the Google authenticator tokens, create the Virtual Server used to generate Google authenticator tokens:
  - Go to Local Traffic, Virtual servers
  - Click on Create
  - Give the virtual server a name, ie generate_ga_token
  - Give the virtual server an IP on your local subnet
  - Assign an HTTP profile (could be the default)
  - Assign an SSL profile (if you want to use SSL)
  - Assign <b>generate_ga_code</b> irule to the virtual server
  - Click on Finished

3) Generate a token:
  - From a browser, access the IP address of the Virtual Server, ie http://192.168.1.50 (You should then see the token generator page)
  - Enter the username of your user (exactly as it exists in whatever authentication option you are using).
  - Click on “Submit” - you should see the QR code and the secret key.
  - Open up your Google Authenticator app and touch the “plus sign”, select scan barcode and scan the QR code.
  - Copy the secret, we will need it next.

4) Save the key in a data group list
  - Go to Local Traffic, iRules, Data Group Lists
  - Click on Create
  - Give the data group list the name <b>google_auth_keys</b> and select type "string". Then, add the username and the secret generated earlier as string and value, respectively. 
  - Click Finished
  - This will have to be repeated for every user (this solution has no self-enrolment neither does it present an automated way of going about the process).

5) Update the Access Policy (you can see details and screenshots here: https://clouddocs.f5.com/training/community/iam/html/archived/class9/module5/lab1.html)
  - Go to Access Policy, Access Profiles
  - Click on Edit in the Access Policy column
  - In the Visual policy editor, click the plus sign to the right of the LocalDB Auth block (or whatever Auth option you are using) 
  - In the Logon tab, choose Logon Page and then Add Item
  - Change the text as per the picture https://clouddocs.f5.com/training/community/iam/html/_images/image1194.png
  - Click Save
  - Click on Add New Macro
  - Name it and click Save
  - Now click on Edit Terminals in the Macro settings
  - Click on Add Terminal
  - Name the terminal “Failure”
  - Rename the terminal called “Out” to successful
  - Click on the Set default tab and set the default to Failure.
  - Click on save
  - Edit the new macro by clicking on the plus sign in the macro settings
  - Go to the General Purpose Tab, click on iRule event and then Add Item
  - Name: Google Auth verification
  - ID: ga_code_verify
  - Then click on Branch rules, Add Brand Rule
  - Name and change the expression according to the following image (https://clouddocs.f5.com/training/community/iam/html/_images/image1315.png):
  - Then click Save
  - Click on the terminals and set Successful to Successful and the rest to Failure terminals
  - Now we’re going to insert the Macro in the main policy. Click on the plus sign to the right of the Get GA Token block
  - Click on the Macro tab and select your Verify Google Token macro. click “Add Item”
  - Now click on Apply Access Policy
  - Finally, apply the new / updated policy to the Virtual Server in use for the VPN (follow https://clouddocs.f5.com/training/community/iam/html/archived/class9/module5/lab1.html for more details)
