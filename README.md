# 🪖 Fuzz on WordPress Sites

---

These are common backup, configuration, and log files that are often left exposed on WordPress sites, potentially leaking sensitive information like database credentials, API keys, or other secrets.

### **Key Files to Fuzz on WordPress Sites:**
1. **`.env` Related Files** (May contain environment variables):
   - `.env.bak`  
   - `.env.php`  
   - `.env.backup`  
   - `.env.save`  

2. **`wp-config.php` Backups** (Contains database credentials & salts):
   - `wp-config-backup.php`  
   - `wp-config.php.save`  
   - `wp-config.php~` (common temp/backup extension)  
   - `wp-config.php.old`  
   - `wp-config.bak`  

3. **Error Logs** (May contain sensitive debug info):
   - `error_log.log`  
   - `php_error.log`  
   - `debug.log`  

4. **Database Dumps** (Full DB backups with user data):
   - `wp.sql`  
   - `db.sql`  
   - `wpbackup.sql`  
   - `mysql_backup.sql`  

5. **Website Backups** (May contain full site copies):
   - `{TARGET}.zip`  
   - `{TARGET}-backup.zip`  
   - `backup-{TARGET}.tar.gz`  

### **Automation Tools:**
- **Fback** ([GitHub](https://github.com/Spix0r/Fback)) – Generates wordlists for fuzzing.  
- **ffuf / wfuzz / dirsearch** – Use these tools with the generated wordlist.  
- **WPScan** (`wpscan --url TARGET --enumerate vp,vt,u`) – Checks for common vulnerabilities.  

### **Example Fuzzing Command (ffuf):**
```bash
ffuf -w backups_wordlist.txt -u https://TARGET.com/FUZZ -mc 200,403
```
(Checks for exposed backup files and returns HTTP 200/403 responses.)

### **Bug Bounty Tip:**  
If you find any of these files, check for:
- **Database credentials** (`DB_NAME`, `DB_USER`, `DB_PASSWORD`)  
- **Authentication salts** (`AUTH_KEY`, `SECURE_AUTH_KEY`)  
- **API keys** (AWS, SMTP, payment gateways)  

---


### **🔍 Extended WordPress Fuzzing Wordlist**
#### **1. Configuration & Environment Files**
```
.env
.env.prod
.env.local
.env.dev
.env.staging
.env.test
.env.backup
.env.bak
.env.old
.env.swp
.env.save
.env.example
.env.dist
.env.php
.env.txt
env.json
config.env
wp-config.php
wp-config.php.bak
wp-config.php.old
wp-config.php.save
wp-config.php~
wp-config.php.swp
wp-config.php.backup
wp-config.php.tmp
wp-config-backup.php
wp-config.inc.php
wp-config-sample.php
wp-config.php.orig
wp-config.php.before-update
```

#### **2. Database & Backup Files**
```
wp.sql
db.sql
backup.sql
database.sql
mysql.sql
wp-db.sql
wp_backup.sql
wpbackup.sql
mysql_backup.sql
backup_db.sql
dump.sql
wordpress.sql
sqlbackup.sql
backup-mysql.sql
backup-db.sql
{TARGET}.sql
{TARGET}-backup.sql
{TARGET}_backup.sql
```

#### **3. Error & Debug Logs**
```
error_log
error.log
php_error.log
debug.log
wp-debug.log
wp-error.log
logs/error.log
logs/access.log
logs/debug.log
```

#### **4. Full Site Backups (ZIP/TAR)**
```
{TARGET}.zip
{TARGET}.tar.gz
{TARGET}.rar
{TARGET}-backup.zip
{TARGET}_backup.zip
backup-{TARGET}.zip
backup.zip
site-backup.zip
wp-backup.zip
full-backup.zip
archive.zip
latest.zip
```

#### **5. Version Control & Temp Files**
```
.git/config
.git/HEAD
.gitignore
.DS_Store
.htaccess
.htaccess.bak
.htaccess.old
wp-content/debug.log
wp-content/backup-db/
wp-content/uploads/wp-backup/
wp-content/backups/
```

---

### **🛠️ Fuzzing Techniques**
#### **1. Using `ffuf` (Fast Web Fuzzer)**
```bash
ffuf -w wordlist.txt -u https://example.com/FUZZ -mc 200,403,301 -t 50
```
- `-mc 200,403,301` → Looks for successful hits, forbidden access, or redirects.  
- `-t 50` → Increases threads for faster scanning.  

#### **2. Using `dirsearch` (Directory Bruteforcer)**
```bash
python3 dirsearch.py -u https://example.com -w wordlist.txt -e php,log,sql,zip,txt
```
- `-e php,log,sql,zip,txt` → Checks for specific extensions.  

#### **3. Using `WPScan` (WordPress-Specific Scanner)**
```bash
wpscan --url https://example.com --enumerate vp,vt,u --plugins-detection aggressive
```
- Checks for vulnerable plugins (`vp`), themes (`vt`), and users (`u`).  

---

### **🔎 What to Do If You Find Exposed Files?**
✔ **`.env` / `wp-config.php`** → Check for **database credentials, API keys, salts**.  
✔ **SQL backups** → Look for **admin passwords, user emails, sensitive data**.  
✔ **Error logs** → May contain **debug info, SQL errors, or paths**.  
✔ **ZIP backups** → Extract and check for **source code, config files, or credentials**.  

---

### **🚀 Pro Tip: Automate with `Fback`**
If you want to generate **dynamic wordlists** based on the target domain, use:  
```bash
python3 fback.py -d example.com -o custom_wordlist.txt
```
This will create a wordlist with patterns like `example.com.zip`, `example.com_backup.sql`, etc.  

---

### **💡 Bonus: Common Credential Locations**
- **`wp-config.php`** → `DB_USER`, `DB_PASSWORD`  
- **`.env`** → `DB_HOST`, `REDIS_PASSWORD`, `AWS_ACCESS_KEY`  
- **SQL Dumps** → `wp_users` table (hashed passwords)  

---

Let’s level up with **custom wordlist generation**, **advanced WordPress fuzzing techniques**, and **exploitation tips** for bug bounty hunters.  

---

## **🚀 Part 1: Custom Wordlist Generator (Python Script)**
This script generates **dynamic fuzzing wordlists** based on the target domain.  

### **📜 Script: `wp_fuzzlist_gen.py`**
```python
import sys

def generate_wordlist(domain):
    # Base patterns
    patterns = [
        # Config & Env files
        ".env", ".env.bak", ".env.prod", ".env.local", ".env.backup",
        "wp-config.php", "wp-config.php.bak", "wp-config.php.old", "wp-config.php.save",
        
        # Database backups
        "wp.sql", "db.sql", "backup.sql", f"{domain}.sql", f"{domain}-backup.sql",
        
        # Error logs
        "error_log", "error.log", "debug.log", "php_errors.log",
        
        # ZIP/TAR backups
        f"{domain}.zip", f"{domain}.tar.gz", f"{domain}-backup.zip", "backup.zip",
        
        # Version control
        ".git/config", ".git/HEAD", ".htaccess", ".htaccess.bak"
    ]
    
    # Save to file
    output_file = f"{domain}_wordlist.txt"
    with open(output_file, 'w') as f:
        for item in patterns:
            f.write("%s\n" % item)
    
    print(f"[+] Wordlist generated: {output_file}")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python wp_fuzzlist_gen.py <domain>")
        sys.exit(1)
    
    domain = sys.argv[1]
    generate_wordlist(domain)
```

### **▶ How to Use:**
```bash
python3 wp_fuzzlist_gen.py example.com
```
This creates **`example.com_wordlist.txt`** with all possible backup/config/log paths.  

---

## **🔥 Part 2: Advanced WordPress Fuzzing Techniques**
### **1. Wayback Machine + Fuzzing**
Find **historical paths** from Wayback Machine, then fuzz them:  
```bash
waybackurls example.com | tee urls.txt  
ffuf -w urls.txt -u https://example.com/FUZZ -mc 200,403,301 -t 100
```

### **2. Backup File Extensions Bruteforce**
Check for **multiple backup extensions**:  
```bash
ffuf -w extensions.txt -u https://example.com/wp-config.phpFUZZ -mc 200  
```
Where **`extensions.txt`** contains:  
```
.bak
.old
.save
~
.swp
.backup
.zip
.tar.gz
```

### **3. Hidden Admin & Debug Paths**
Check for **exposed admin/debug endpoints**:  
```
/wp-admin/admin-ajax.php  
/wp-content/debug.log  
/wp-content/uploads/wp-backup/  
/wp-json/wp/v2/users/  
```

---

## **💣 Part 3: Exploiting Found Files**
### **1. If You Find `.env` or `wp-config.php`**
- **Database takeover**: Look for `DB_USER`, `DB_PASSWORD`.  
- **Salts exploit**: If `AUTH_KEY` is exposed, attackers can **hijack sessions**.  

### **2. If You Find SQL Backups (`wp.sql`, `db.sql`)**
- Extract **admin credentials** (`wp_users` table).  
- Check for **plaintext passwords** or crack hashes with `hashcat`.  

### **3. If You Find ZIP Backups**
- Extract and check for:  
  - **`wp-config.php`**  
  - **`.env` files**  
  - **Database scripts**  

---

## **🔐 Part 4: Protecting Your WordPress Site**
### **✅ Prevention Tips for Developers**
1. **Block access to sensitive files** in `.htaccess`:  
   ```apache
   <Files ~ "\.(env|sql|bak|old|log)$">
      Order allow,deny
      Deny from all
   </Files>
   ```
2. **Disable directory listing**:  
   ```apache
   Options -Indexes
   ```
3. **Use `robots.txt` to block crawlers**:  
   ```
   User-agent: *
   Disallow: /wp-admin/
   Disallow: /wp-content/uploads/
   Disallow: /wp-config.php
   ```

---

## **🎯 Final Tip: Automate with Nuclei**
Use **Nuclei templates** to scan for exposed WordPress files:  
```bash
nuclei -u https://example.com -t ~/nuclei-templates/exposures/
```
(Check for **`wp-config.php`, `.env`, and SQL backups** automatically.)  

---

### **📌 Summary**
✔ **Generated a custom wordlist script** for fast fuzzing.  
✔ **Advanced fuzzing tricks** (Wayback, extensions, hidden paths).  
✔ **Exploitation tips** for `.env`, SQL backups, and ZIP files.  
✔ **Defensive measures** to protect WordPress sites.  

