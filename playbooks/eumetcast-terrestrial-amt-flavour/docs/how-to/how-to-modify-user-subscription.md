User management and subscription is controlled entirely via the [EUMETSAT User Portal](https://user.eumetsat.int/). Users wishing to add or remove data products from their subscription should do so in the EUMETSAT User Portal where they will find all available products for the EUMETCast Terrestrial service.

  

By default, all products in a user’s subscription are accepted by the TelliCast Client, as shown by the uncommented **`name=*`** section of the channel configuration file (see image below).

However, if a user has altered the channel configuration file and commented out the “name=\*” section in favour of storing their data per channel, the user must ensure that their chosen channel is uncommented in the configuration file _**/etc/cast-client-channels\_ter-1.ini**._

  

vi /etc/cast-client-channels\_ter-1.ini

![](../images/image-2023-4-3_12-54-10.png "European Weather Cloud Knowledge Base > EUMETCast Terrestrial on AMT - How to modify user subscription > image-2023-4-3_12-54-10.png")