# Ports 2049 | NFS

### Enumeration

```bash
nmap --script nfs-ls,nfs-showmount,nfs-statfs <IP>
showmount -e <IP>

# Metasploit
use auxiliary/scanner/nfs/nfsmount
```

In the example below `nmap` has identified `/home/simon *` as being mountable. The asterisk dictates any address can mount this file path.

![](<../../.gitbook/assets/image (1857).png>)

### Mounting

To mount an export first create a directory on the attacking machine.

```markup
sudo mkdir /mount/
```

Then use the command below to mount to the directory just created.

```markup
sudo mount -t nfs <IP>:<PATH> /mount/ -o nolock
```

![](<../../.gitbook/assets/image (1858).png>)

### Mount Confirmation

Running the command mount will list available mounted paths. Using grep we can filter for relevant paths.

```markup
mount <PATH>
```

![](<../../.gitbook/assets/image (1859).png>)

