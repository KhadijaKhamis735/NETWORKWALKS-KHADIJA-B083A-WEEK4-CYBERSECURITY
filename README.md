Student Name:Khadija Khamis  
Batch: B083A  
Program: NetworkWalks Cybersecurity Internship  
Week: 4  
Target: https://medirozahospital.com  
Operating System: Kali Linux


Introduction
This project is a black-box penetration test on Mediroza General Hospital website (https://medirozahospital.com). 
The goal was to identify vulnerabilities, gain unauthorized access, retrieve confidential patient lab reports, crack their encryption, and find sensitive staff and shareholder information.
I performed the test on Kali Linux using tools such as curl, browser, pdfcrack, qpdf, and grep.


Using Tools for Reconnaissance
I started the penetration test by performing reconnaissance on the target website https://medirozahospital.com.
First, I used the curl command to check the robots.txt file. This file helps identify directories that the website owner does not want search engines to crawl.
Command used: curl https://medirozahospital.com/robots.txt
The output showed that the following directories were disallowed:
1) patient
2) staff
3) old
<img width="483" height="282" alt="image" src="https://github.com/user-attachments/assets/620188ce-a4a1-45cc-a250-ced29d86377c" />
Next, I visited the /old/ directory in the browser and also used curl to confirm the content. Directory listing was enabled, and I found a sensitive database backup file named mediroza_db_backup_2019.sql.
Command used to download the file: wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
<img width="1080" height="351" alt="image" src="https://github.com/user-attachments/assets/758b506d-a945-4673-b7bb-21c9253ff6b7" />
This was a critical finding because the SQL backup contained confidential staff salaries and shareholder details.



Gaining Access using SQL Injection
After reconnaissance, I focused on the Patient Portal login page located at:
https://medirozahospital.com/patient/login.php
I tested the login form for SQL Injection vulnerabilities. I discovered that the application was vulnerable to authentication bypass.
I used the following payload in the username field: admin'
I left the password field with any value.
Command used: curl -X POST https://medirozahospital.com/patient/login.php -d "username=admin'--&password=test" -c cookies.txt -L -o portal.html


After sending the request, I was successfully logged into the Patient Portal without knowing the real password.
<img width="1080" height="139" alt="image" src="https://github.com/user-attachments/assets/a8ca63f1-99c3-40eb-ac97-ee24c2bce558" />

Inside the Patient Portal, I found three confidential patient laboratory reports available for download:

- Pathology Report - S. Dlamini
- Pathology Report - P. Reddy
- Pathology Report - E. Thompson
<img width="1080" height="294" alt="image" src="https://github.com/user-attachments/assets/22c7e0f9-59b0-4c83-a8ae-47f7acaab9f9" />
I downloaded all three encrypted PDF files for further analysis.




Cracking the PDF Passwords
After downloading the three PDF lab reports, I noticed that all of them were password protected.
I used pdfcrack and qpdf tools on Kali Linux to recover the passwords.
For the first report (S. Dlamini), the password was successfully cracked as:

123456

For the second report (P. Reddy), the password was successfully cracked as:

password

Commands used:

qpdf --password=123456 --decrypt report_1.pdf report_1_opened.pdf
qpdf --password=password --decrypt report_2.pdf report_2_opened.pdf

<img width="1080" height="164" alt="image" src="https://github.com/user-attachments/assets/7af42a1b-7b53-4ba0-8355-650a4e4d9750" />

The third report (E. Thompson) remained encrypted after multiple dictionary and targeted attacks.

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/5bd30bee-65b2-46d5-bf1f-9b628933fd0f" />



 
Extracting Staff Salaries and Shareholder Details
I analyzed the database backup file (mediroza_db_backup_2019.sql) that was found during reconnaissance.
The file contained two important tables:

1. Staff table – with full names, job titles, departments, and monthly salaries
2. Shareholders table – with shareholder names, share percentages, and share classes

Commands used:

grep -A 50 "INSERT INTO \`staff\`" mediroza_db_backup_2019.sql
grep -A 20 "INSERT INTO \`shareholders\`" mediroza_db_backup_2019.sql


<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/43ddb8f0-f330-4edc-891a-3c87678c1296" />


<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/927701e5-0989-4d66-9c69-028ca55d56ec" />

This confirmed that sensitive internal data was publicly accessible due to the exposed backup file.



Key Findings
During the penetration test, I discovered the following important findings:

1) Directory Listing Vulnerability
   The `/old/` directory had directory listing enabled, which exposed a sensitive database backup file containing staff salaries and shareholder information.

2) SQL Injection Vulnerability
   The Patient Portal login page was vulnerable to SQL Injection. Using the payload `admin'--`, I was able to bypass authentication and gain unauthorized access.

3) Weak Password Protection on Patient Reports
   Two out of three confidential patient lab reports were protected with very weak passwords (`123456` and `password`). This shows poor password policy on sensitive medical documents.

4) Exposure of Confidential Internal Data
   The exposed SQL backup file revealed full staff salary details and the complete list of hospital shareholders.


 
 Challenges Faced
While performing this penetration test, I faced the following challenges:

1) The third PDF report (E. Thompson) was protected with a stronger password. I tried many dictionary attacks and targeted wordlists but was not able to recover the password.

2) At the beginning, the rockyou wordlist was not available on my Kali Linux system, so I had to create custom wordlists.

3. Some commands took a long time to run, especially when trying to brute-force the third PDF password.

Despite these challenges, I was able to complete the main objectives of the project successfully.



Conclusion
The Week 4 black-box penetration test on Mediroza General Hospital was completed successfully.
I was able to:
1) Discover a sensitive database backup through directory listing
2) Bypass authentication using SQL Injection
3) Download three confidential patient lab reports
4) Crack the passwords of two out of three encrypted PDF files
5) Extract staff salary information and shareholder details
These findings show the importance of securing web directories, fixing SQL Injection vulnerabilities, using strong passwords for sensitive files, and removing old backup files from production servers.

