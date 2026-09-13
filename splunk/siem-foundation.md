# [ SIEM and Log Monitoring Lab ]

## Splunk Enterprise Setup

I installed Splunk Enterprise on my RHEL system and used it as the SIEM for this lab.

I started Splunk using:

```bash
sudo /opt/splunk/bin/splunk start --accept-license
```
Install Splunk Enterprise on RHEL
Splunk Web was available at:

```text
http://127.0.0.1:8000
```

RHEL was used as the Splunk server and Windows was used as the endpoint sending security logs.

## Web Log Ingestion

I created a web log on RHEL using a Python HTTP server and added the log file to Splunk.

At first, one incorrect Python command was also written into the log. After starting the server correctly, I generated HTTP requests and checked them in Splunk.

![Web log added to Splunk](Screenshot%202026-09-12%20143335.png)

I searched for HTTP GET requests using:

```spl
index=lab_web "GET"
```

The results showed requests from localhost and my Windows VM `10.10.1.11`.

![HTTP GET search](Screenshot%202026-09-12%20151301.png)

## Basic SPL Practice

I used `stats` to count the events:

```spl
index=lab_web
| stats count
```

![Stats count](Screenshot%202026-09-12%20152447.png)

I also used `timechart` to check how the events were spread over time:

```spl
index=lab_web
| timechart count
```

![Timechart count](Screenshot%202026-09-12%20152510.png)

Then I counted events by host:

```spl
index=lab_web
| stats count by host
```

![Events by host](Screenshot%202026-09-12%20153735.png)

This helped me understand how Splunk can summarize logs instead of checking every event one by one.

## Field Extraction

I practiced extracting useful fields from the raw web logs.

I extracted:

```text
client_ip
method
uri
status
```

and displayed them in a table.

![Web log field extraction](Screenshot%202026-09-12%20230114.png)

This helped me understand how raw log text can be converted into useful fields for investigation.

## Windows Log Collection

I installed Splunk Universal Forwarder on the Windows VM.

On RHEL Splunk, I enabled receiving on TCP port `9997`.

I configured the Windows forwarder to send logs to:

```text
10.10.1.10:9997
```

I also allowed the required traffic through the RHEL firewall for the lab.

I collected:

```text
Windows Security
Windows System
Sysmon Operational
PowerShell Operational
```

I then searched:

```spl
index=lab_windows
```

and confirmed that Windows events were reaching Splunk.

![Windows logs in Splunk](Screenshot%202026-09-13%20221000.png)

## Windows Log Sources

I checked the event count by source using:

```spl
index=lab_windows
| stats count by source
```

Splunk showed data from:

```text
WinEventLog:Security
WinEventLog:System
WinEventLog:Microsoft-Windows-Sysmon/Operational
WinEventLog:Microsoft-Windows-PowerShell/Operational
```

![Windows log sources](Screenshot%202026-09-13%20222341.png)

This confirmed that the Windows VM was sending multiple security log sources to the RHEL Splunk server.

## Authentication Investigation

I used my `soclab` test account to generate failed and successful login activity.

I searched for Windows authentication events related to the account and Event IDs `4625` and `4624`.

```spl
index=lab_windows "soclab" (4625 OR 4624)
```

`4625` showed failed login activity and `4624` showed successful login activity.

![Authentication investigation](Screenshot%202026-09-13%20224410.png)

## Result

I successfully built a basic SIEM setup using RHEL and Windows.

Windows logs were forwarded to Splunk on RHEL, and I was able to search both web and Windows security events from one place.

I also practiced basic SPL, field extraction and authentication log investigation.

## What I learned

I learned how logs from different systems can be collected into one SIEM.

I also learned how to search, filter, count and extract useful information from events instead of reading only raw logs.

The authentication test helped me understand how failed and successful login events can be checked together during an investigation.

## SOC Relevance

A SOC analyst uses SIEM data to find activity across different systems and log sources.

This lab helped me practice collecting endpoint logs, searching events and starting an investigation from authentication activity.
