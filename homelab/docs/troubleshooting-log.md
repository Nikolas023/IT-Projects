# Issue Log
## Here are some issues that I ran into during this project

### Active Directory and DNS
- Symptom: nslookup on DC01 showed "Server: UnKnown" and reverse lookups of 192.168.50.10 returned "Non-existent domain."
- Investigation: Forward lookup (nslookup dc01.lab.local) returned 192.168.50.10, so forward DNS was working and the problem was limited to reverse lookups. 
- Root cause: No PTR record mapped 192.168.50.10 back to dc01.lab.local.
- Fix: Created the PTR record by enabling "Update associated pointer (PTR) record" on DC01's A record.
- Verification: nslookup 192.168.50.10 returned dc01.lab.local, and nslookup showed "Server: dc01.lab.local."
- Takeaway: AD doesn't require reverse DNS, but troubleshooting tools and logs rely on it.

### Group Policy
- Symptom: Running gpresult /h C:\report.html on CLIENT01 as a domain user returned "Access is denied."
- Investigation: The error appeared when the report was being saved, not during the policy query, which pointed to a file permissions problem rather than a Group Policy problem.
- Root cause: The signed-in user account didn't have write permission to C:\.
- Fix: Saved the report to a folder the user owns: gpresult /h %USERPROFILE%\Downloads\report.html. 
- Verification: The report was generated and showed the Nursing GPOs (drive mapping, desktop restrictions) applied to the user.
- Takeaway: Read where an error comes from before fixing it. Here, Group Policy was working fine; the problem was just where I tried to save the file.

### Remote Desktop
- Symptom: Connecting from DC01 to CLIENT01 over RDP failed with "Remote Desktop can't find the computer 'CLIENT01'."
- Investigation: Researched the error and found it means name resolution failed, not an RDP configuration problem. Checked each layer: ipconfig /all on CLIENT01 showed a valid DHCP address and DNS server 192.168.50.10. nslookup client01.lab.local on DC01 returned "Non-existent domain." Connecting by IP address succeeded, confirming the problem was DNS.
- Root cause: CLIENT01 hadn't registered its A record in DNS.
- Fix: Ran ipconfig /registerdns on CLIENT01 from an elevated Command Prompt.
- Verification: A CLIENT01 A record appeared in DNS Manager, nslookup client01.lab.local returned the correct IP, and RDP connected as LAB\itsupport.
- Takeaway: Read the error to identify the failing layer, and confirm DNS before changing RDP or firewall settings.
