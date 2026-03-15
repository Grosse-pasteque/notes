# Port scanning
```bash
sudo nmap -sS -O -p- -T4 -Pn --open --min-rate 10000 example.com
```
# File transfer
### netcat
```bash
netcat -l 8080 > file.txt
cat file.txt | netcat 192.168.0.1 8080 -q 0
```
### ssh (scp)
```bash
# use -r for folders
scp file.txt root@1.2.3.4:/path/to
scp root@1.2.3.4:/path/to/file.txt .
```
# Start php server in working directory
```bash
php -S localhost:8000
```
# Get network stats
### Windows
```powershell
Get-NetAdapterStatistics
```
### Linux
```bash
cat /proc/net/dev
```
# Get process stats
```bash
cat /proc/(self|[pid])/(stat|status)
```
# Simple curl
### fetch headers only
```bash
curl -I https://example.com
```
### fetch response and headers
```bash
curl -i https://example.com
```
### fetch with headers
```bash
curl -H "Accept-Encoding: gzip, deflate" -i https://example.com
```
