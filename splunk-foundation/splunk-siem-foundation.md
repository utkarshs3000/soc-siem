# [ SIEM and Log Monitoring Lab ]

## Splunk Enterprise Setup

I installed Splunk Enterprise on RHEL and used it as the SIEM for this lab.

I started Splunk using:

```bash
sudo /opt/splunk/bin/splunk start --accept-license
```

Splunk Web was available at:

```text
http://127.0.0.1:8000
```

I also created separate lab indexes so the data was easier to manage, including `lab_web` and `lab_windows`.

## Web Server and Web Logs

I created a simple page on RHEL and started a Python HTTP server on port `8080`.

```bash
echo "Phase 5 SIEM Lab" > index.html
python3 -m http.server 8080 > logs/webserver.log 2>&1
```

I opened the page from RHEL and from my Windows VM `10.10.1.11`.

Each request was written into:

```text
/home/sate/soc-siem/logs/webserver.log
```

At first, one wrong Python command was also written into the log. After correcting the command, the web server worked normally.

## Web Log Ingestion

I added `webserver.log` to Splunk as a monitored file and stored it in the `lab_web` index with the `soc-web` sourcetype.

![alt text](Screenshot%202026-09-12%20143335.png)

I searched for HTTP GET requests using:

```spl
index=lab_web "GET"
```

The results showed requests from localhost and the Windows VM.

![alt text](Screenshot%202026-09-12%20151301.png)

## Basic SPL Practice

I practiced counting and summarizing the web events.

```spl
index=lab_web
| stats count
```

![alt text](Screenshot%202026-09-12%20152447.png)

```spl
index=lab_web
| timechart count
```

![alt text](Screenshot%202026-09-12%20152510.png)

```spl
index=lab_web
| stats count by host
```

![alt text](Screenshot%202026-09-12%20153735.png)

I also changed the time range while searching so I could focus on recent activity instead of all stored events.

## Field Extraction

I used `rex` to extract useful values from the raw web logs and displayed them with `table`.

The fields I extracted were:

```text
client_ip
method
uri
status
```

![alt text](Screenshot%202026-09-12%20230114.png)

This helped me understand how raw log text can be turned into useful fields for investigation.

## Windows Log Forwarding

I installed Splunk Universal Forwarder on the Windows VM.

On RHEL Splunk, I enabled receiving on TCP port `9997` and allowed the Windows VM to reach that port through the RHEL firewall.

The forwarder was configured to send logs to:

```text
10.10.1.10:9997
```

I collected:

```text
Windows Security
Windows System
Sysmon Operational
PowerShell Operational
```

I searched:

```spl
index=lab_windows
```

and confirmed that Windows events were reaching Splunk.

![alt text](Screenshot%202026-09-13%20221000.png)

## Windows Log Sources

I checked the event count by source:

```spl
index=lab_windows
| stats count by source
```

Splunk showed:

```text
WinEventLog:Security
WinEventLog:System
WinEventLog:Microsoft-Windows-Sysmon/Operational
WinEventLog:Microsoft-Windows-PowerShell/Operational
```

![alt text](Screenshot%202026-09-13%20222341.png)

This confirmed that multiple Windows security log sources were reaching the RHEL Splunk server.

## Authentication Investigation

I used my `soclab` test account to generate failed and successful login activity.

I searched for Event IDs `4625` and `4624` related to the account:

```spl
index=lab_windows "soclab" (4625 OR 4624)
```

`4625` showed failed login activity and `4624` showed successful login activity.

![alt text](Screenshot%202026-09-13%20224410.png)

## Result

I built a basic SIEM lab where RHEL runs Splunk and Windows sends security logs to it.

I was able to generate web activity, collect the logs, search them with SPL, extract fields and investigate Windows authentication events.

## What I learned

I learned how activity becomes a log, how the log is sent to Splunk, and how I can search the same data from one place.

I also learned that useful fields and time ranges make an investigation easier than reading raw events one by one.

## SOC Relevance

This lab gave me practice with log collection, centralized monitoring and basic authentication investigation using Splunk.
