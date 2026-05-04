# How to check file availability

You can quickly check for missing files by searching the Tellicast client log files.

Run the following command (replace `ter-1` with the service you want to check):

```bash
grep -i "missed" /var/log/tellicast-client/recv_ter-1.log | wc -l
```

> 💡 If you observe a high number of missed files, contact the EUMETSAT User Helpdesk (ops@eumetsat.int) and provide the relevant log files.

**Resources**
- [How to monitor and check connectivity of the EUMETCast client](./how-to-monitor-and-check-connectivity-of-the-eumetcast-client.md)


