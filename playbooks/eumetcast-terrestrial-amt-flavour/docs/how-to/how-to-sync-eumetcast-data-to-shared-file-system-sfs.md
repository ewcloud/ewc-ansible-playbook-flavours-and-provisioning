You can sync data received by EUMETCast Terrestrial client to shared file system (SFS) with following steps: 

1.  Setup the file share, and mount it to the EUMETCast Terrestrial VM in preferred folder (here: /mountpoint) following: [EUMETSAT - Shared File System (SFS) usage in tenants](https://confluence.ecmwf.int/display/EWCLOUDKB/EUMETSAT+-+Shared+File+System+%28SFS%29+usage+in+tenants) (EUMETSAT side only)
2.  Create directory with preferred name, e.g.  
      
    
    mkdir /mountpoint/eumetcast-data
    
3.  Modfiy crontab  
      
    
    sudo crontab -e
    
    by adding following line (note that if you have modified the default location of the data, this have to be modified respectively):  
      
    \*/1 \* \* \* \* rsync -rt --delete /root/data/eumetcast/ter-1/default/ /mountpoint/eumetcast-data
    
    Note that you may need to set folder permissions to enable data usage in your applications. The above command takes care to always sync back changes done locally to the remote. That means it takes care to synchronize _local_ and _remote_ such that ( _local → /root/data/eumetcast/ter-1/default/_  _remote → /mountpoint/eumetcast-data_ )
    
    1.  files **added** in _local_ are added to _remote_
    2.  files **removed** from _local_ are removed from _remote_
    3.  files **added** in _remote_ are removed
    4.  files **removed** from _remote_ are restored from _local_ if they exist, else ignored