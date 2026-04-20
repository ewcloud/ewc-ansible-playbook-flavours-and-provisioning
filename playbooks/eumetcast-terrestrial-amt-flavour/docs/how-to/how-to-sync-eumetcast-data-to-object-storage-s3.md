You can sync data received by EUMETCast Terrestrial client to object storage with following steps: 

1. Setup s3cmd following [Object Storage: How to use s3cmd and s3fs](https://confluence.ecmwf.int/display/EWCLOUDKB/Object+Storage%3A+How+to+use+s3cmd+and+s3fs)

2. Create bucket with preferred name, e.g.
```bash
s3cmd mb s3://eumetcast-example
```

3. Copy the .s3cfg to /etc
```bash
sudo cp ~./s3cfg /etc/s3cfg
```

4. Modify crontab
```bash
sudo crontab -e
```
by adding following line (note that if you have modified the default location of the data, this have to be modified respectively):

```bash

*/1 * * * * s3cmd -c /etc/s3cfg sync /root/data/eumetcast/ter-1/default s3://eumetcast-example 
```

