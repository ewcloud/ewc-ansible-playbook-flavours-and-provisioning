The entries to the log file for the tellicast client (/var/log/tellicast-client/recv_ter-1.log) is by default buffered.

In order to get real-time log events:

1.  ssh into the VM

2. Alter the file **/etc/cast-client_ter-1.ini**, removing the “>>” in front of the log file path:
```bash
log_file=/var/log/tellicast-client/recv_ter-1.log
```

3. After altering the file, you have to restart the service running the following command:
```bash
sudo systemctl restart tellicast-client
```