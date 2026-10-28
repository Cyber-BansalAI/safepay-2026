# Hunt guide: did SafePay get in, and did it touch our backups?

As of **2026-10-27**. Read [DISCLAIMER.md](../DISCLAIMER.md) first. We have not tested these commands on your systems.
Run them yourself, on a copy of the logs if you can, and adjust paths to your setup. Nothing in this package runs
anything for you, and every command shown only reads.

**Two questions, kept apart.**

1. *Is there sign the group was in our network?* Answered from firewall, VPN and sign-in records, and from files and
   tools on computers and servers.
2. *Did it go after our backups and our data?* Answered from backup storage, backup software, and records of large
   uploads.

The advisory describes the group stealing data first, then locking files, then posting stolen data on a leak site if
the victim does not pay. So a locked file is the **last** step. The signs below can appear before that.

## 0. Set your window

- The agencies saw these indicators between **November 2024 and August 2026**. Search at least that far back if your
  records go that far.
- If your logs begin later, say so in your notes: you cannot rule anything out for the earlier period.

## 1. Read the cautions first

- The advisory says some listed files are legitimate tools and that it recommends verifying them before acting. A match
  on a tool such as AnyDesk, WinRAR or PsExec is a question for whoever installed it, not a finding.
- The agencies also say an address used for harm at one time can later belong to a harmless service. Several addresses
  in `iocs/iocs.csv` look like shared services. Look each one up before you block it.
- A clean result is not proof of safety. File names and hashes are easy to change.

## 2. Network records: firewall, VPN and remote access

Look at the VPN or remote access log first. The advisory says the group got in with stolen or guessed passwords, with
phished account profiles on VPNs, and by exploiting flaws in internet-facing equipment. It also says the group
connected from addresses of the VPN provider Mullvad.

Read the addresses from the dataset and search an exported firewall or proxy log for them (adjust the file names):

```
$ips = (Import-Csv .\iocs\iocs.csv | Where-Object { $_.type -eq 'ip' }).value
Select-String -Path .\firewall-export.log -SimpleMatch -Pattern $ips
```

How to read a match:

- Any match to an address in the dataset deserves a look at **when** and **what it was**. Several of these addresses
  also carry ordinary traffic.
- A VPN or remote access sign-in from a country or provider your staff never use is worth asking about even with no
  match here.
- Large uploads from a server to an outside address are covered in section 7.

## 3. Files and tools on computers and servers

Search for the file names in the dataset. This lists matches without changing anything:

```
Get-ChildItem -Path C:\ -Recurse -Force -ErrorAction SilentlyContinue -Include dcsync.exe,ShareFinder.ps1,SharpShares.exe,Socks.exe,Socks_werfault.exe,locker.dll,libsmb2.dll,portable.dat,netscanNEW.zip,DYRasWtb.exe,'Libwa*.dll','wa_3rd_party_host_*.exe','*.SafePay' | Select-Object FullName, Length, LastWriteTime
```

If a file turns up, work out its hash and compare it with the `sha1` and `sha256` rows:

```
Get-FileHash -Algorithm SHA256 -Path "C:\path\to\the\file"
```

How to read it:

- Files ending **.SafePay** mean data was already locked. Treat that as an incident (section 9).
- `libsmb2.dll` is also the name of a real open-source library, and `1 - Copy.bat` is the name Windows gives a copy of
  a file called `1.bat`. Check the location, the date and what installed the file.
- The advisory also lists tools the group used that are common and legitimate: PsExec, WinRM, WinRAR, 7-Zip, FileZilla,
  WinSCP, rclone, Mimikatz, NetExec, a disk-management tool, and a system cleaner. We did not list them as indicators.
  Ask whether anyone at your company installed them.

## 4. Accounts and sign-ins

The advisory says the group makes new accounts, or makes existing accounts administrators, to stay in. On a Windows
server or domain controller these read-only commands show the last 30 days of new accounts and of people added to
administrator groups:

```
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4720; StartTime=(Get-Date).AddDays(-30)} | Select-Object TimeCreated, Message
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4728,4732,4756; StartTime=(Get-Date).AddDays(-30)} | Select-Object TimeCreated, Message
```

Remote Desktop sign-ins (logon type 10) from addresses you do not recognise are also worth a look:

```
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddDays(-30)} | Where-Object { $_.Message -match 'Logon Type:\s+10' } | Select-Object TimeCreated, Message
```

These records need Windows auditing turned on. If they are empty, that is itself a finding: you may not be recording
what you need.

The dataset also lists four computer names the advisory calls IOC hostnames. They may appear as the source computer
name in sign-in or network records.

## 5. Backups

This is where the advisory is most specific. It says the group stops the Volume Shadow Copy service and resets network
storage that holds backups to its factory settings, and in some cases deleted every backup that was connected to the
network.

- **Network storage.** Check its own admin log or ask your provider whether it was reset, had its settings changed or
  had backups deleted that nobody remembers deleting.
- **Backup software.** Look for failed, skipped or deleted jobs and for changed schedules or retention settings.
- **Volume Shadow Copy.** This command lists the service stopping or starting in the last 30 days. The service also
  stops by itself when idle, so a stop on its own proves little. Look for it alongside other signs:

```
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7036; StartTime=(Get-Date).AddDays(-30)} | Where-Object { $_.Message -like '*Volume Shadow Copy*' } | Select-Object TimeCreated, Message
```

- **The test that matters.** Do you have a backup copy that is **offline or cannot be changed**, and has someone
  restored from it recently? The agencies recommend backups that are offline or immutable, encrypted and tested.

## 6. Remote tools you did not install

The advisory says the group installed AnyDesk or TeamViewer on victims' computers to move ransomware around, and used
ScreenConnect to stay in. Many companies use these tools legitimately. List what is installed and ask who put it there:

```
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" | Get-ItemProperty | Where-Object { $_.DisplayName -match 'AnyDesk|TeamViewer|ScreenConnect' } | Select-Object DisplayName, InstallDate
```

An entry nobody can explain, or one installed on a server that never needed remote help, is a reason to call your IT
provider.

## 7. Data leaving

The advisory says the group uploaded stolen data with tools such as rclone and WinSCP to accounts it controlled, to
cloud storage such as Mega and Backblaze B2, and in some cases to OneDrive. Ask your IT provider or firewall vendor for
a report of the **largest outbound transfers** from servers and file shares over the last 90 days, and look for
uploads to storage services nobody at the company uses. Ordinary backups will also appear, so compare against what
you expect.

## 8. What the results mean

| You found | It suggests | Next step |
|---|---|---|
| A published address in the firewall or VPN log, nothing else | A connection to a shared or reassigned address, or early activity | Look up the address, check the date and the account. Keep the logs |
| A tool such as AnyDesk, WinRAR or ScreenConnect nobody can explain | A possible foothold, or a tool someone forgot to mention | Ask who installed it. If no one did, treat it as an incident |
| A listed file name, or a hash that matches | Likely attacker tooling | Treat as an incident |
| New administrator accounts, backups reset or deleted, large uploads nobody expected | Likely an intruder preparing to lock files | Treat as an incident now: the advisory puts data theft and backup damage before the lock-up |
| Files ending .SafePay, or a ransom note | Data is already locked | Section 9 |
| Nothing | Not proof of safety: these indicators change and your records may not capture what matters | Fix the basics in section 10 anyway |

## 9. If you think it is real

Preserve first, clean later:

1. Isolate the affected computers from the network, but do not turn them off if you can avoid it, and keep the logs
   unchanged.
2. Call your IT provider. Do not use the computers that may be affected to talk about it.
3. Report to the FBI's Internet Crime Complaint Center, a local FBI or Secret Service field office, or CISA. The agencies
   ask for logs showing connections to outside addresses, a sample ransom note, any messages from the group and a
   harmless sample of an encrypted file. The advisory lists the contact details.
4. The agencies do not encourage paying a ransom: payment is no promise that files will come back, and it can
   encourage further attacks.

## 10. Reduce the risk

These are drawn from the advisory's mitigations. Read the advisory for the full list.

- Require MFA on email, VPN and every account that can reach important systems.
- Patch internet-facing equipment first, especially anything the CISA Known Exploited Vulnerabilities list names.
- Keep backups that are offline or cannot be changed, encrypt them, and test a restore.
- Limit administrator accounts, review them regularly, and watch for new ones.
- Record network traffic and sign-ins, and look for odd connections between your own computers.

## Not sure what you found?

Bring the log lines to your IT person first. If you want a second pair of eyes,
[SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) offers a free 20-minute
[visibility check](https://cyber.bansalai.com/?ref=github#book). For the business-owner version of this page, with four
questions to ask your IT person, read
[SafePay Ransomware: What to Ask Your IT Person About Backups and Remote Access](https://blog.bansalai.com/news/safepay-ransomware-backups-remote-access/?ref=github).

## Source for this guide

The joint advisory AA26-293A by the FBI, CISA, the U.S. Secret Service and partner agencies (full credit and link in
[CREDITS.md](../CREDITS.md)). Windows event numbers are standard Windows records and are not from the advisory.
