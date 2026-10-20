# RemoteAccessRegret — Digital Forensics & Incident Response Investigation

**Platform:** Hack The Box (Sherlock)  
**Challenge:** RemoteAccessRegret  
**Category:** Digital Forensics & Incident Response (DFIR)  
**Focus:** Browser Forensics | AnyDesk Log Analysis | Social Engineering | Data Exfiltration  
**Status:** Completed

**User:** Andree Salvo (@Wompyxyz)

> This repository documents my analysis of the supplied Hack The Box training artifacts. The incident and identifiers belong to the fictional lab scenario; this is not a live incident report.

## Overview

I investigated a technical-support scam involving a fake Microsoft security warning, the misuse of AnyDesk for remote access, and the theft of sensitive files. By examining Google Chrome browsing artifacts and AnyDesk application data, I reconstructed how the victim encountered the scam, identified the remote connection, and documented file transfers from the victim's computer.

## Scenario

On **February 18, 2025**, Margaret was browsing for Thanksgiving recipes when a browser page displayed an alarming message claiming her computer was infected. The page instructed her to contact a supposed Microsoft support representative. The caller convinced her to install remote-access software, remotely accessed her computer, and ultimately demanded **$500 in gift cards** for a supposed protection plan.

My objective was to validate the incident through forensic evidence and answer the ten Sherlock investigative questions.

## Tools and Artifacts

- **Linux terminal:** `file`, `mount`, `tree`, `cat` for inspecting artifacts and logs.
- **DB Browser for SQLite:** inspected Google Chrome's `History` SQLite database.
- **Firefox:** rendered locally recovered cached HTML to examine the fake warning page.
- **Chrome artifacts:** `History` database and cached webpage HTML.
- **AnyDesk artifacts:** `system.conf` and `ad.trace`.

The supplied lab write-up describes identifying a FAT filesystem image, mounting it, and exploring its directory structure before examining the relevant artifacts. My included screenshots begin with the AnyDesk configuration examination; earlier filesystem inspection screenshots are not included in this repository.

## Investigation and Evidence

### 1. Identify the victim's AnyDesk ID

I opened `Users/Margaret/AppData/Roaming/AnyDesk/system.conf`. Under the `[client]` section, the configuration recorded `id=847291503`.

**Finding:** Victim AnyDesk ID — `847291503`.

<img width="959" height="489" alt="01-victim-anydesk-id" src="https://github.com/user-attachments/assets/1fb59c49-af71-475f-9eb6-acc9c7d86c0a" />

### 2. Investigate the first website visited

I opened `Users/Margaret/AppData/Local/Google/Chrome/User_Data/Default/History` in DB Browser for SQLite and inspected the `urls` table. The first entry was a Google search for Thanksgiving recipes.

**Finding:** `www.google.com`.

<img width="1023" height="661" alt="02-chrome-history" src="https://github.com/user-attachments/assets/2f315dd6-88c9-4443-966d-bac9aceece6b" />

### 3. Identify the malicious redirect

Continuing through the Chrome history entries, I found a suspicious Windows-themed security alert domain following the initial recipe-related browsing activity.

**Finding:** `ww1.windows-security-alert.com`.

<img width="1024" height="661" alt="03-malicious-redirect" src="https://github.com/user-attachments/assets/fe9a116f-531d-4316-8c81-33791992e928" />

### 4. Recover the scammer's phone number

I examined HTML recovered from Chrome's cache. The page impersonated a Microsoft Windows Defender alert, displayed an urgent infection warning, and directed the victim to call a supposed support number. I also rendered the recovered HTML locally in Firefox to examine the warning as the victim would have seen it.

**Finding:** `+1 (888) 351-4019`.

<img width="965" height="488" alt="04-scam-html" src="https://github.com/user-attachments/assets/4db8ef94-79b0-44f8-9a26-3d4e6d946e9a" />

<img width="1021" height="742" alt="04-scam-html-webpage" src="https://github.com/user-attachments/assets/02446de5-c4df-4c92-a8f5-8dc993757380" />

**Analysis:** This is evidence of a browser-based tech-support impersonation lure. The warning's claim of a Trojan infection is part of the scam page, not independent proof that the computer contained that malware.

### 5. Identify the attacker's remote AnyDesk ID

I reviewed `Users/Margaret/AppData/Roaming/AnyDesk/ad.trace`. The log recorded an incoming session request and the remote ID of the connecting system.

**Finding:** Attacker remote AnyDesk ID — `529481627`.

<img width="969" height="488" alt="05-fake-security-alert" src="https://github.com/user-attachments/assets/a83e860a-52b3-45f8-8a81-0b61480eb21a" />

### 6. Identify the remote connection alias

Immediately after the remote ID, the same trace log listed an alias chosen by the remote user.

**Finding:** `MicrosoftSupport`.

<img width="971" height="484" alt="06-attacker-anydesk-id" src="https://github.com/user-attachments/assets/7478a6eb-aeb2-48f2-a462-156404091b66" />

**Analysis:** The alias is consistent with the Microsoft support impersonation described in the scenario; it does not establish a genuine Microsoft affiliation.

### 7. Determine the remote session duration

I examined the session summary near the end of `ad.trace` and found the total duration recorded by AnyDesk.

**Finding:** `4052 seconds` (1 hour, 7 minutes, 32 seconds).

<img width="969" height="489" alt="07-attacker-alias" src="https://github.com/user-attachments/assets/471daf62-c356-42d4-b65c-50a46197bdc7" />

### 8. Identify the first file transferred from the Desktop

The AnyDesk trace recorded file transfers after the remote session began, including files from the Desktop and Documents folders. The first Desktop file transferred was a bank statement PDF.

**Finding:** `Bank Statements 2024.pdf`.

<img width="969" height="489" alt="08-Bank Statements 2024" src="https://github.com/user-attachments/assets/aec21b00-620e-4313-bfc3-bd3e05a67e99" />

**Additional file transfers recorded:**

- `C:\Users\Margaret\Desktop\Bank Statements 2024.pdf` — 247,293 bytes
- `C:\Users\Margaret\Documents\Tax Return 2023.pdf` — 1,293,847 bytes
- `C:\Users\Margaret\Desktop\Passwords.docx` — 18,293 bytes

  <img width="968" height="487" alt="Additional file Transfers recorded" src="https://github.com/user-attachments/assets/a07702b5-cad4-46ae-841b-6e75b7d67134" />


**Analysis:** The recorded upload activity demonstrates exfiltration of financial and potentially credential-related files through the remote-access application.

### 9. Quantify the uploaded data

The AnyDesk session summary recorded the number of bytes uploaded to the remote system.

**Finding:** `1,559,433 bytes` uploaded.

<img width="969" height="488" alt="09-Total Transferred" src="https://github.com/user-attachments/assets/fbcdbe01-7d5a-4052-90ec-aa560eb0b1bb" />

### 10. Count the browsed URLs

Finally, I returned to Chrome's `History` database and reviewed the `urls` table. The database view displayed ten entries, including bank- and Target-related pages later in the sequence.

**Finding:** `10` total URL entries for the lab browsing session.

<img width="1024" height="657" alt="11-browsing-session" src="https://github.com/user-attachments/assets/e66bf5ac-ba3e-479a-8328-a7a59fcc1236" />

## Key Findings

| Evidence | Result |
|---|---|
| Victim AnyDesk ID | `847291503` |
| Initial website | `www.google.com` |
| Malicious redirect | `ww1.windows-security-alert.com` |
| Fraudulent support number | `+1 (888) 351-4019` |
| Attacker AnyDesk ID | `529481627` |
| Attacker alias | `MicrosoftSupport` |
| Remote session duration | `4052 seconds` |
| First exfiltrated Desktop file | `Bank Statements 2024.pdf` |
| Total uploaded data | `1,559,433 bytes` |
| URLs in Chrome history | `10` |

## Incident Timeline

The following timestamps come from the supplied AnyDesk `ad.trace` evidence. The log's timezone is not established in the available screenshots, so times are reported exactly as recorded without assigning a timezone.

| Date/time (log time) | Event |
|---|---|
| Feb 18, 2025, 08:35:11 | AnyDesk started on the victim system. |
| 08:41:17 | Incoming request from remote ID `529481627`, alias `MicrosoftSupport`. |
| 08:41:23 | Local user accepted the connection; session established. |
| 08:41:23 | Remote session granted input, clipboard, and file-transfer permissions. |
| 08:48:22 | Remote user opened the file manager. |
| 08:52:17–08:52:22 | `Bank Statements 2024.pdf` uploaded. |
| 08:55:08–08:55:14 | `Tax Return 2023.pdf` uploaded. |
| 09:01:33–09:01:35 | `Passwords.docx` uploaded. |
| 09:22:07–09:40:08 | Multiple clipboard-to-remote events recorded. |
| 09:48:55 | Remote connection ended; summary recorded `4052` seconds and `1,559,433` bytes uploaded. |

The scenario and browsing history also support a broader sequence: recipe search → malicious redirect → fake Microsoft warning → telephone social engineering → AnyDesk session → file exfiltration → gift-card-related browsing. Exact browser-event times are not established by the provided screenshots.

## Recommendations

Based on the lab scenario, useful defensive measures include:

1. **Security awareness:** Train users to recognize fake browser infection alerts, urgent support numbers, and gift-card payment demands.
2. **Remote access controls:** Restrict or monitor unsanctioned remote-access tools and review unusual inbound sessions.
3. **Exfiltration detection:** Alert on unexpected bulk uploads, remote file-manager use, and clipboard transfers from sensitive endpoints.
4. **Incident response:** Terminate unauthorized sessions, preserve logs and browser artifacts, isolate affected devices where appropriate, and assess exposed credentials and financial data.
5. **Browser protection:** Apply web-filtering and threat-protection controls against known fraudulent support domains.

## Conclusion

This Sherlock lab strengthened my practical DFIR skills by requiring correlation of Chrome browser artifacts with AnyDesk configuration and event logs. I identified the fraudulent security warning, linked it to an unauthorized remote-access session, and documented three uploaded files totaling **1,559,433 bytes**. The investigation illustrates how social engineering can lead to remote-access abuse and sensitive-data theft—and how endpoint artifacts can reconstruct that activity.

## Skills Demonstrated

`Digital Forensics` · `Incident Response` · `Browser Forensics` · `SQLite` · `Linux` · `Log Analysis` · `AnyDesk Analysis` · `Social Engineering Investigation` · `Data Exfiltration Analysis` · `Evidence Documentation`

---

**Note:** This repository documents a controlled Hack The Box training exercise. Findings are based on the supplied lab artifacts and accompanying screenshots.
