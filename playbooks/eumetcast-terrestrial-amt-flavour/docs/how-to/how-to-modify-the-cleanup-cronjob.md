# How to modify the cleanup cronjob

The item, automatically cleans up disk space by keeping only the latest 5 minutes of transmitted data by default, under the /home/eumetuser/data subdirectory.

The installation provides under the root user the following cronjobs by default that find all regular files (-type f) whose status was last changed more than 5 minutes ago (-cmin +5) and delete them (-delete). 
You can see them under the root user running `crontab -l` (you will see one for each channel + one for the logs):

```
#Ansible: None
/5 * * * find /root/data/eumetcast/ter-1/default -mmin +5 -type f -delete
#Ansible: None
/5 * * * find /root/data/eumetcast/ter-1/ -mmin +5 -type f -delete
#Ansible: None
/5 * * * find /var/log/amt/ -mtime +7 -type f -delete
```

If you want to modify those, you need to become root and run `crontab -e` to change them.
