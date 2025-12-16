# Ansible Role: ipa_cert_expiry_exporter

This Ansible role installs a systemd timer and service to periodically export FreeIPA certificate expiration timestamps as Prometheus metrics for monitoring.

## Features

- Runs a custom script to parse `ipa-getcert` certificate expiry info.
- Exports metrics to `/var/lib/node_exporter/ipa_cert_expiration.prom`.
- Uses `systemd` timer for scheduling (recommended over cron).
- Integrates with existing Prometheus `node_exporter`.
- Requires `node_exporter` service installed and running.
- Sends metrics with labels for each certificate request ID.
- Metrics file ownership is set to `node-exp` user.

## Requirements

- Prometheus with node_exporter installed and running.
- `ipa-getcert` CLI available and accessible by root.
- System running `systemd` (most modern Linux distros).

## Role Variables

| Variable              | Default                    | Description                             |
|-----------------------|----------------------------|-----------------------------------------|
| `script_path`         | `/usr/local/bin/ipa_cert_expiry.sh` | Path to the exporter script              |
| `timer_interval_minutes` | `15`                      | Interval in minutes between metric updates |

## Usage

1. Install and start `node_exporter` on your hosts.

2. Apply this role to install the exporter script and systemd timer.

3. Check that the timer is active:

```bash
systemctl status ipa_cert_expiry_exporter.timer
```
4. View logs and output with:
```bash
journalctl -u ipa_cert_expiry_exporter.service
```
5. Prometheus will scrape the metrics exported to:
```bash
/var/lib/node_exporter/ipa_cert_expiration.prom
Make sure node_exporter is configured to read from this path (e.g., with --collector.textfile.directory=/var/lib/node_exporter).
```
---

## 🚀 Example Playbook

```yaml
- hosts: ipa_clients
  become: true
  roles:
    - ipa_cert_expiry_exporter
```

## 🔔 Example Prometheus Alert Rule
```yaml
- alert: IpaCertificateExpiringSoon
  expr: (ipa_cert_expiration_ts - time()) < 86400 * 15
  for: 1h
  labels:
    severity: warning
  annotations:
    summary: "IPA Certificate Expiring Soon on {{ $labels.instance }}"
    description: "Cert '{{ $labels.request_id }}' will expire in less than 15 days."
```

## Troubleshooting

If no metrics appear, verify the exporter script runs successfully:
1. `sudo /usr/local/bin/ipa_cert_expiry.sh`
2. Ensure permissions of `/var/lib/node_exporter/ipa_cert_expiration.prom` are owned by node-exp.
3. Confirm node_exporter service is running and configured properly.

## 👤 Author

Maintained by Ibrahim Musayev.