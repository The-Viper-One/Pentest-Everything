# Ports 8080 | 8180 | Apache Tomcat

Metasploit default credentials module

```bash
use auxiliary/admin/http/tomcat_administration 
use auxiliary/scanner/http/tomcat_mgr_login
use auxiliary/scanner/http/tomcat_enum
```

Post Exploitation

```bash
use post/multi/gather/tomcat_gather
use post/windows/gather/enum_tomcat
```

