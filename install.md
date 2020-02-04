### Config VM

Langague : english-US
Time format : US
Keyboard : Swiss French
Windows Key :  WC2BQ-8NRM3-FDDYY-2BFGV-KHKQY
System : Windows server 2016 (Desktop Experience)
Installation : Custom

### Install IIS 10

On dashboard, choose "Add roles and features"
Select server roles "Web Server (IIS)"
No change Features
On Role Services, choose in Application Developement
    screen 3
And install

### Config sites

Go on Internet Information Services (IIS) Manager
Disabled all active site
"WIN-XXX..." -> "Sites" -> "Default Web Site" -> right clic -> "Manage" -> "Stop"
Add new default document iis.html
Next go on "C:\iis_www" and add new file iis.html and paste "MON Site IIS" into

### Install PHP

Create folder "PHP7.4.2" in "C:\"
Go on "https://windows.php.net/download/" and download PHP 7.4.2 in zip format
Extract content of zip in "C:\PHP7.4.2\" folder
Go on "https://aka.ms/vs/16/release/vc_redist.x64.exe" and execute the exe file
Next go back on Internet Information Services (IIS) Manager -> "WIN-XXX..." and choose "Handler Mappings"
Add new mapping
Fill content with the same configuration
Restart site

### Add Auth

Define static ip adress
Install Server Roles "Active Directory Domain Services"
Show warning message et clic on "Promote this ..."
Add a new forest "srw3.local"
Check form is same and add Pa$$w0rd password
After just go of end of config and finish installation
Add "Windows Authentification" on Manage -> Add Roles and Features
Next Go on Internet Information Services (IIS) Manager -> "WIN-XXXX..." and select "Authentification" on IIS
Disabled all and Enable "Windows Authentification"

### FTP

