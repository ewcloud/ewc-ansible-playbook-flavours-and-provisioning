Users can check their availability by searching the TelliCast Client log files for missing files. You can run the following command:

```bash
grep –I “missed” /var/log/tellicast-client/recv_ter-1.log | wc –l
```

> 💡 If a user is experiencing a high data loss, please contact the EUMETSAT User Helpdesk (ops@eumetsat.int)