# Target Acquisition

Target acquisition through PsMapExec is utilized through ADSI Searcher. As long as you are operating from a domain joined system as a domain user account, no issues should be encountered when acquiring targets.

By default only enabled Active Directory computer accounts are populated into the target list. \
\
IP Address targeting and CIDR ranges are supported but are less preferred than using ADSI methods.

### Syntax

```
PsMapExec -Targets [Targets]
```

The following parameters are supported for `-Targets`&#x20;

```powershell
# Grabs all workstations and Servers and Domain Controllers from the domain
PsMapExec -Targets All

# Grabs only servers from the domain (Excludes DCs)
PsMapExec -Targets Servers

# Grabs only Domain Controllers from the domain
PsMapExec -Targets DCs

# Grabs only workstations from the domain
PsMapExec -Targets Workstations

# Set the target values to a defined computer name
PsMapExec -Targets DC01.Security.local

# Wildcard selection
PsMapExec -Targets SRV0*

# Gather targets from file 
PsMapExec -Targets C:\Targets.txt

# Single IP Address
PsMapExec -Targets 192.168.56.11

# CIDR Range
PsMapExec -Targets 192.168.56.0/24
```

