The TelliCast Client offers a GUI for users to check their connectivity.You can go to your browser and search for http://<public-ip-station>:8500

![](../images/image-2023-4-11_15-14-1.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to monitor and check connectivity of EUMETCast client > image-2023-4-11_15-14-1.png")

  

The tabs on the left hand side of the page allow the user to:

*   check the overall connectivity status **(Overview)**
*   information on the file availability statistics **(Statistics)**
*   the current active channels **(Active Channels)**
*   the log file entries **(Log File)**
*   License information **(License)**
*   More information on the monitoring **(Help)**

# How to change the port used for monitoring

By default, the port used for this is **8500** which can be changed in the TelliCast Client configuration file.

This can be altered to a port of the user’s choice as follows:

1.  Open the TelliCast Client configuration file

vi /etc/cast-client\_ter-1.ini (ter-1 service)

2\. Update following parameters in the config (e.g. set port to 80)

![](../images/image-2023-4-11_15-8-54.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to monitor and check connectivity of EUMETCast client > image-2023-4-11_15-8-54.png")

  

Users must ensure that the firewall on their station allows HTTP access via their chosen port in order to access the GUI.

Remember you have to create a new rule that allows that port in the security group of the VM, you can follow this guide for more information: [Creating Security Groups in Morpheus](https://confluence.ecmwf.int/display/EWCLOUDKB/Creating+Security+Groups+in+Morpheus)).

Sample security rule can be seen below.

  

![](../images/image-2023-4-11_15-9-14.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to monitor and check connectivity of EUMETCast client > image-2023-4-11_15-9-14.png")