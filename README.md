# Penetration Testing and Vulnerability Exploitation on Metasploitable 3 with Kali Linux
This project is a hands-on **penetration testing** exercise, simulating real-world cyberattacks using **Kali Linux** as the attacker and **Metasploitable 3** as the target. The primary objective is to breach the system and gather sensitive credentials while following a structured approach.

Key actions include:
1. **Network Discovery**: Using **Nmap**, we perform a detailed scan to uncover active hosts, open ports, and running services on the target network.
2. **Exploiting SMB Vulnerabilities**: After identifying an open **SMB (Server Message Block)** port, a brute force attack is launched to crack usernames and passwords.
3. **Local User Enumeration**: With the retrieved SMB credentials, we further explore the system by enumerating local users on the target machine.
4. **Remote Command Execution**: Leveraging valid credentials, we execute remote commands using **Metasploit**, establishing a **Meterpreter** session for deeper system access.
5. **Credential Theft**: We employ tools like **hashdump** and **Mimikatz** to extract password hashes and plaintext credentials directly from the system memory.
6. **MySQL Database Breach**: Targeting the **MySQL** service, we run a brute force attack to gain access to sensitive database information.
7. **Covering Tracks**: To avoid detection, we clear system logs and remove traces of the attack.
8. **DoS Attack as a Diversion**: To distract defenders, we execute a **Denial of Service (DoS)** attack, exploiting a vulnerability in the **Remote Desktop Protocol (RDP)**.

The exercise highlights critical vulnerabilities and emphasizes the importance of strong security practices like robust password policies, timely system updates, and proactive monitoring to defend against potential threats.
