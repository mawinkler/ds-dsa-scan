# DSA Scan

Documentation: [Deep Security Help](https://cloudone.trendmicro.com/docs/workload-security/command-line-interface/#dsa_scan)

Uses: [inotifywait](https://linux.die.net/man/1/inotifywait)

## inotifywait

Install

```sh
sudo apt-get install inotify-tools -y
```

## Malicious Files

```sh
curl --silent --location https://secure.eicar.org/eicarcom2.zip -o /home/ubuntu/download/eicarcom2.zip
```

## Code Example

```sh
#!/bin/bash

# sudo apt install inotify-tools -y

# https://cloudone.trendmicro.com/docs/workload-security/command-line-interface/#dsa_scan

monitor=/home/ubuntu/download
logs=/home/ubuntu/watchfolder-logs
action=quarantine

mkdir -p ${logs}

sudo inotifywait -r -m -e close_write ${monitor} |
  while read file_path file_event file_name; do
    scan_file=${file_path}${file_name}
    scan_file_log=$(date +%s)_${scan_file//\//_}.log
    printf "%s" "$(date): ${scan_file}" >> scan.log

    sudo /opt/ds_agent/dsa_scan --action ${action} --log ${logs}/${scan_file_log} --target ${file_path}${file_name}
    exit_code=$?
    case ${exit_code} in
    0)
      echo Scan completed and no malware found.
      printf "%s\n" " - clean" >> scan.log
      ;;&
    1)
      echo Scan completed with at least one malware found.
      printf "%s\n" " - infected, ${action}" >> scan.log
      ;;&
    2)
      echo Scan completed, no malware found but some files skipped.
      printf "%s\n" " - clean, files skipped" >> scan.log
      ;;&
    3)
      echo Scan completed, but at least malware found and some files skipped.
      printf "%s\n" " - infected, files skipped, ${action}" >> scan.log
      ;;&
    *)
      echo ${file_path}${file_name}: ${exit_code}
    esac
  done
```

Example `scan.log`:

```
Wed Mar 20 13:50:43 UTC 2024: /home/ubuntu/download/scan.log - clean
Wed Mar 20 13:59:02 UTC 2024: /home/ubuntu/download/ICRC_ROLL.lck - clean
Wed Mar 20 13:59:03 UTC 2024: /home/ubuntu/download/PlatformDetection - clean
Wed Mar 20 13:59:03 UTC 2024: /home/ubuntu/download/TRXHANDLER_ROLL.lck - clean
Wed Mar 20 13:59:04 UTC 2024: /home/ubuntu/download/agent.deb - clean
Wed Mar 20 14:01:50 UTC 2024: /home/ubuntu/download/eicarcom2.zip - infected, quarantine
```

## Demo Setup

- Add directory exclusions on monitored directory<br>`/home/ubuntu/download/`
- Policy -> AM -> Manual Scan:
  enable <br>`Allow agent to trigger or cancel a manual scan from Trend Micro's notifier application` (even on linux)
- Policy -> Web Reputation: disable

## Exit Codes

Exit code | Description | Resolution
--------- | ----------- | ----------
0 | Scan completed and no malware found. | Scan task completed without malware found.
1 | Scan completed with at least one malware found. | Check lines labelled as Infected and Warning in the output.
2 | Scan completed, no malware found but some files skipped | Check lines labelled as Skipped in the output.
3 | Scan completed, but at least malware found and some files skipped. | Check lines labelled as Infected, Warning, and Skipped in the output.
246 | The argument string is too long. | The string size limit is 2048 characters. Shorten the target parameter and try again.
247 | The Security Platform is shutting down. | The agent is stopping, try again later.
248 | Too many instances. | The concurrent dsa_scan running instances are 10.
249 | No permission. | The command requires root on Linux and Administrator on Windows. Checkup the "Allow the agent to trigger or cancel a manual scan" on the scan policy.
250 | Manual Scan Configuration is not set. | Configure the Manual Scan setting on the scan policy.
251 | AM feature is not enabled. | Enable the AM feature on the scan policy.
252 | The platform is not supported. | The dsa_scan is not supported on the current OS platform.
253 | The agent is not running. | Deep Security Agent is not running. Enable the agent or contact the administrator.
254 | Invalid parameters. | The input parameters are incorrect.
255 | Unexpected error. | Try it again if the issue still persists, contact the administrator.