# Log Analysis Using Linux Commands

Firstly, we will use **PuTTY** for the CLI and **WinSCP** for transferring files from Windows to Linux.

Now we need to download the `access.log` file on Windows and transfer it to Linux using WinSCP.

We found a log file on GitHub and downloaded it.

Before transferring the file, we should create a directory in Linux.

## Create a Working Directory
```bash
mkdir logs
```

Then transfer the `access.log` file into this directory using WinSCP.

After the transfer, we can verify the file inside the directory.

Use the following command to list files:

```bash
ls
```

We can also verify the file details and preview its contents:

```bash
ls -lh access.log
head -5 access.log
```

---

# 1. Count Total Requests in the Log File

## Command
```bash
wc -l access.log
```

## Explanation
- `wc` = word count command
- `-l` = count lines
- Each line represents one request

---

# 2. Count Unique IP Addresses

## Step-by-Step Approach

### Step 1: Extract Only IP Addresses
```bash
awk '{print $1}' access.log | head -10
```

## Explanation
- `awk '{print $1}'` = print the first field (IP address)
- `head -10` = show the first 10 results

---

### Step 2: Sort the IP Addresses
```bash
awk '{print $1}' access.log | sort | head -10
```

## Explanation
- `sort` = arrange IP addresses in alphabetical/numerical order
- Sorting groups duplicate IPs together

---

### Step 3: Remove Duplicate IP Addresses
```bash
awk '{print $1}' access.log | sort | uniq | head -10
```

## Explanation
- `uniq` = remove adjacent duplicate lines
- `uniq` should be used after `sort`

---

### Step 4: Count Unique IP Addresses
```bash
awk '{print $1}' access.log | sort | uniq | wc -l
```

## Alternative (More Efficient)
```bash
awk '{print $1}' access.log | sort -u | wc -l
```

## Explanation
- `sort -u` = sort and remove duplicates at the same time

---

# 3. Find the Top 10 Most Frequent IP Addresses

## Command
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

## Explanation
1. `awk '{print $1}'` → extract IP addresses  
2. `sort` → sort the IPs  
3. `uniq -c` → count occurrences of each IP  
4. `sort -rn` → sort numerically in reverse order  
5. `head -10` → show the top 10 results  

---

# 4. Count Total 404 Errors

## Command
```bash
grep " 404 " access.log | wc -l
```

## Explanation
- `grep " 404 "` = search for lines containing status code `404`
- Spaces avoid matching values like `4040`
- `wc -l` = count matching lines

---

## Better Approach Using `awk`
```bash
awk '$9 == 404' access.log | wc -l
```

## Explanation
- `$9 == 404` = match lines where the 9th field (HTTP status code) equals `404`

---

## Analyze All Status Codes
```bash
awk '{print $9}' access.log | sort | uniq -c | sort -rn
```

---

# 5. Find the Top 5 Requested URLs That Returned 404 Errors

## Command
```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -rn | head -5
```

## Explanation
1. `$9 == 404` → filter only 404 responses  
2. `{print $7}` → extract the requested URL  
3. `sort | uniq -c` → count occurrences  
4. `sort -rn` → sort by highest frequency  
5. `head -5` → show the top 5 results  

---

# 6. Count Requests Made by the User-Agent "Nikto"

## Command
```bash
grep -i "nikto" access.log | wc -l
```

## Explanation
- `grep -i` = case-insensitive search
- Searches the entire log line, including the User-Agent field

---

## View Sample Requests
```bash
grep -i "nikto" access.log | head -5
```

---

## Get Unique IPs Using Nikto
```bash
grep -i "nikto" access.log | awk '{print $1}' | sort -u
```

---

# 7. Find Potential SQL Injection Attempts

## Common SQL Injection Patterns
- `' OR '1'='1`
- `UNION SELECT`
- `DROP TABLE`
- `; --`
- `%27` (URL-encoded single quote)

---

## Search for SQL Keywords
```bash
grep -iE "(union|select|insert|update|delete|drop|exec|script)" access.log | wc -l
```

## Explanation
- `-i` = case-insensitive
- `-E` = extended regular expression
- `|` = OR operator

---

## Search for Encoded SQL Injection Attempts
```bash
grep -E "(%27|%20union|%20select)" access.log
```

---

## Extract Suspicious Requests
```bash
grep -iE "(union|select)" access.log | awk '{print $1, $7}' | head -10
```

## Output
Shows:
- IP address
- Requested URL

---

## Count SQL Injection Attempts by IP
```bash
grep -iE "(union|select)" access.log | awk '{print $1}' | sort | uniq -c | sort -rn
```

---

# 8. Find Potential Cross-Site Scripting (XSS) Attempts

## Common XSS Patterns
- `<script>`
- `javascript:`
- `onerror=`
- `%3Cscript%3E` (URL-encoded)

---

## Command
```bash
grep -iE "(<script|javascript:|onerror=|%3Cscript)" access.log
```

---

## Count XSS Attempts
```bash
grep -iE "(<script|javascript:|onerror=|%3Cscript)" access.log | wc -l
```

---

## Identify Possible Attackers
```bash
grep -iE "(<script|javascript:|onerror=)" access.log | awk '{print $1}' | sort | uniq -c | sort -rn
```

---

# 9. Find Directory Traversal Attempts

## Common Traversal Patterns
- `../`
- `..%2F` (URL-encoded)
- `....//`

---

## Command
```bash
grep -E "(\.\./|\.\.%2[Ff])" access.log
```

---

## Count Traversal Attempts
```bash
grep -E "(\.\./|\.\.%2[Ff])" access.log | wc -l
```

---

# 10. Find Which Hours Had the Most Traffic

## Command
```bash
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | sort -rn
```

## Explanation
1. `awk '{print $4}'` → extract the timestamp field  
2. `cut -d: -f2` → extract the hour  
3. `sort | uniq -c` → count requests per hour  
4. `sort -rn` → sort by highest traffic  

---

# 11. Identify HTTP Methods Used

## Command
```bash
awk '{print $6}' access.log | tr -d '"' | sort | uniq -c | sort -rn
```

## Explanation
- `awk '{print $6}'` → extract HTTP methods (GET, POST, etc.)
- `tr -d '"'` → remove quotation marks
- Count and sort the methods

---

# 12. Extract IP Addresses That Made POST Requests

## Command
```bash
awk '$6 == "\"POST"' access.log | awk '{print $1}' | sort -u
```

## Shorter Version
```bash
awk '$6 == "\"POST" {print $1}' access.log | sort -u
```

---

## View POST Request Details
```bash
awk '$6 == "\"POST" {print $1, $7, $9}' access.log | head -10
```

---

# 13. Calculate Total Bandwidth Usage

## Command
```bash
awk '{sum += $10} END {print sum/1024/1024 " MB"}' access.log
```

## Explanation
- `{sum += $10}` → add all transferred bytes
- `END {print ...}` → convert bytes into MB and display the result

---

## Bandwidth Usage by IP Address
```bash
awk '{bytes[$1] += $10} END {for (ip in bytes) print ip, bytes[ip]/1024/1024 " MB"}' access.log | sort -k2 -rn | head -10
```

---

# 14. Create a Clean List of Unique IP Addresses

## Command
```bash
awk '{print $1}' access.log | sort -u > unique_ips.txt
```

---

## Verify the File
```bash
wc -l unique_ips.txt
head unique_ips.txt
```

---

# 15. Mask the Last Octet of IP Addresses for Privacy

## Command
```bash
sed 's/\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}/\1XXX/' access.log | head -5
```

---

## Simpler Alternative
```bash
sed 's/\.[0-9]*\( \)/\.XXX\1/' access.log | head -5
```

---

# 16. Create a File Containing Only Requested URLs

## Command
```bash
awk '{print $7}' access.log | sort -u > urls.txt
```

---

## Filter Specific File Types
```bash
awk '{print $7}' access.log | grep -E "\.(php|asp|jsp)$" | sort -u
```
