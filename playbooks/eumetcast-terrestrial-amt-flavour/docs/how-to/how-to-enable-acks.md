# How to enable NAKs / ACKs (retransmission feedback)

The EUMETCast Terreistrail on AMT Item automatically configures the necessary NAT rules for the EUMETSAT network. No manual iptables changes are required on the station.

To enable acknowledgements (which improve file availability through retransmission):

1. Send an email to the EUMETSAT User Helpdesk (ops@eumetsat.int) requesting ACKs/NAKs for your station. Attach 24 hours of Tellicast client log files for all active services (e.g. `recv_ter-1.log`, `recv_ter-2.log`, `recv_ter-3.log`).

2. Once EUMETSAT verifies good reception and enables the feedback channel on their side, no further changes are needed on your instance.

> ⚠️ Enabling retransmission will consume a small portion of the forward channel bandwidth.

**Resources**
- [How to check file availability](./how-to-check-file-availability.md)


