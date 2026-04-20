# General

  

|     |     |     |     |
| --- | --- | --- | --- |
| Terrestrial Service | **Total Bandwidth (Mbps)** | Data | Default |
| **ter-1** | 240 | **EPS, MSG, Sentinel-3A/B, Sentinel-5P, Third Party data, MTG** | yes |
| **ter-2** | 168 | **Sentinel-6 data** |     |
| **ter-3** | 230 | **Sentinel-5P L1B, Sentinel-3A/B OLCI L1 FR, Sentinel-3A/B SLSTR L1B, FY3 HIRAS, FY4 GIIRS, GOSAT, MTG HRFI-FD (4 high-res full-disk bands)** |     |

Note: Additional terrestrial services will be created in the future for various missions. If unsure of what services you need to enable on your station, please contact the EUMETSAT User Helpdesk ([ops@eumetsat.int](mailto:ops@eumetsat.int)).

  

1.  Update user subscription in EO Portal
2.  Log into Reception station (e.g. ssh into it)
3.  Open the file called **`tellicast-client.cfg`**

 vi /etc/tellicast-client.cfg

4\. Update the following parameter and save the file.

INSTANCE\_START\_ORDER=ter-1,ter-2

**![](../images/image-2023-4-3_9-48-46.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to add an additional service > image-2023-4-3_9-48-46.png")  
**

5\. Open the file called `cast-client_ter-2.ini`

  

 vi /etc/cast-client\_ter-2.ini

  

6\. Add credentials and tunnel IP to ter-2 configuration file and save the file

user\_name=<username>
user\_key=<password>
interface\_address=10.0.0.11

![](../images/image-2023-4-3_9-50-31.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to add an additional service > image-2023-4-3_9-50-31.png")

![](../images/image-2023-4-3_9-50-37.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to add an additional service > image-2023-4-3_9-50-37.png")

  

7\. OPTIONAL: Update storage path for new data, modifying `cast-client-channels_ter-2.ini`  file (See also [EUMETCast Terrestrial on AMT - How to change file storage path](https://confluence.ecmwf.int/display/EWCLOUDKB/EUMETCast+Terrestrial+on+AMT+-+How+to+change+file+storage+path))

vi /etc/cast-client-channels\_ter-2.ini

![](../images/image-2023-4-3_9-51-36.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to add an additional service > image-2023-4-3_9-51-36.png")

  

8\. OPTIONAL: Configure IP for NAKs (if enabled) using the following commands (See also [EUMETCast Terrestrial on AMT - How to enable NAKs](https://confluence.ecmwf.int/display/EWCLOUDKB/EUMETCast+Terrestrial+on+AMT+-+How+to+enable+NAKs))

iptables -t nat -A POSTROUTING --destination 193.17.9.7/32 -j SNAT --to-source <internal-IP of station>

  

iptables -t nat –L

Ensure the rule persists after reboot by saving the iptables rules:

sudo apt-get install iptables-persistent

  

iptables-save > /etc/iptables/rules.v4

  

9\. Restart tellicast client software

sudo systemctl restart tellicast-client

  

Log entries for each terrestrial service can be found in **_/var/log/tellicast-client/recv\_ter-x.log_** where x indicates the service, i.e. **/var/log/tellicast-client/recv\_ter-1.log** for ter-1 service.

  

# Terrestrial Service Ter 3 (NEW)

EUMETCast Terrestrial will undergo reorganisation on 11 September 2023. This change is intended to optimise the bandwidth usage on the EUMETCast Terrestrial service.

The announcement channel for TER-3 (232.223.224.1:4711, source 193.17.9.4) is now active. Users now launch the new TER-3 instance on their reception station, as specified in the announcement.

**A new EUMETCast Terrestrial instance (TER-3) will be created to disseminate data that is exclusively available on EUMETCast Terrestrial and not on EUMETCast Satellite.**

For more information you can refer to the official page from EUMETCast team: [https://eumetsatspace.atlassian.net/wiki/spaces/DSEC/pages/1989640193/EUMETCast+Terrestrial+Re-organisation](https://eumetsatspace.atlassian.net/wiki/spaces/DSEC/pages/1989640193/EUMETCast+Terrestrial+Re-organisation)

  

**For users using EUMETCast Terrestrial on the EWC prior to the 11 of September**

  

Users in the EWC with existing EUMETCast Terrestrial machines can include the new service in the following ways:

**OPTION 1: Add configuration files to existing machine  
**

Add the configuration files for TER-3 as described in the table below:

|     |     |     |
| --- | --- | --- |
| configuration file name | configuration file | location in your machine |
| cast-client-channels\_ter-3 | [cast-client-channels\_ter-3.ini](https://confluence.ecmwf.int/download/attachments/327648093/cast-client-channels_ter-3.ini?version=1&modificationDate=1693992679353&api=v2) | /etc |
| cast-client\_ter-3 | [cast-client\_ter-3.ini](https://confluence.ecmwf.int/download/attachments/327648093/cast-client_ter-3.ini?version=1&modificationDate=1693992694935&api=v2) | /etc |

  

**OPTION 2: Redeploy a new machine from Morpheus**

Check Provision Client VM here: [EUMETCast Terrestrial on AMT](https://confluence.ecmwf.int/display/EWCLOUDKB/EUMETCast+Terrestrial+on+AMT)