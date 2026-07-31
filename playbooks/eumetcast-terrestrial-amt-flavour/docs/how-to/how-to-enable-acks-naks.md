# How to enable ACKs and NAKs (retransmission feedback)

> ⚠️ The feedback channel required for retransmission is disabled by default during the trial phase. This results in reduced availability with logs reporting "Lost X messages during the last 60 seconds". Please enable ACKs and NAKs in coordination with EUMETSAT for optimal availability.

To enable acknowledgements (which improve file availability through retransmission):

1. Send an email to the EUMETSAT User Helpdesk (ops@eumetsat.int) requesting ACKs  and NAKs for your station. Attach 24 hours of Tellicast client log files for all active services (e.g. `recv_ter-1.log`, `recv_ter-2.log`, `recv_ter-3.log`).

2. Once EUMETSAT verifies good reception and enables the feedback channel on their side, no further changes are needed on your instance.


**Resources**
- [How to check file availability](./how-to-check-file-availability.md)
