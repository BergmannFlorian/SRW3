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