  

By default, all EUMETCast Terrestrial data is stored in **/_root/data/eumetcast/_**.

In order to change the file storage path:

1.  Open the channel configuration file for the applicable service (e.g. for ter-1)

vi /etc/cast-client-channels\_ter-1.ini 

  

2. Alter the **`target_directory`**  and **`tmp_directory`**  parameters so they point to the desired storage location (absolute path recommended)

![](../images/image-2023-4-3_10-37-2.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to change file storage path > image-2023-4-3_10-37-2.png")

**`name=*`**  indicates that all data regardless of channel will be stored in one directory

  

3\. Alternatively, users can store data depending on their channels by uncommenting the channel file and updating the **`target_directory`**  path

![](../images/image-2023-4-3_10-38-45.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to change file storage path > image-2023-4-3_10-38-45.png")