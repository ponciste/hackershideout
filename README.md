# Hacker's Hideout Write-Up

## Investigation

### First Attack

Let's investigate the oldest logs by browsing page number 50 of the logs.

Perform a standard search by typing `*` in the search bar and selecting `All time` in the date filter.

![Standard Search](images/all_logs_standard_search.png?raw=true "Standard Search")

![Logs page 50](images/oldest_logs.png?raw=true "Logs Page 50")

From the logs below, we can deduce several key pieces of information about what transpired:

1. The date and time of the logs are consistent across multiple entries, indicating a **brute-force attack**.
2. All HTTP requests are of type GET, and the URLs vary for each log entry.
3. The URLs suggest that someone attempted a brute-force attack to **load a local path through a GET parameter**.
4. The name of a well-known brute-forcing tool; **Fuzz Faster U Fool v2.0.0-dev**.
5. The attacker's IP address.

Based on this information, it becomes evident that, for this part of the analysis, we are dealing with a **brute-forced LFI (Local File Inclusion) attack**.

---

To further explore the **potential damage caused by the LFI** (Local File Inclusion) vulnerability, we will apply specific filters to assess the impact of the LFI attack.

Let's proceed by implementing the following search filter:

```sql
"GET" AND "200" AND "Fuzz Faster U Fool"
```

#### Result


We have identified **97** events (or log entries), indicating that the hacker successfully accessed the content of **97 files on the server**.

![GET 200 response filter](images/200_response_filter.png?raw=true "GET 200 response filter")

To verify this, we can independently confirm by copying and pasting the URL into a web browser.

#### Example

![Logs for GET 200 response](images/200_response.png?raw=true "Logs for GET 200 response")

**OS Information**

![/etc/os-release content](images/os-release.png?raw=true "/etc/os-release content")


**Users (/etc/passwd) Information**

![Users Information](images/passwd.png?raw=true "Users Information")

---

### Second Attack

In the challenge, a **second attack** is mentioned. Therefore, let's utilize the information we have gathered to exclude the first attack.

To accomplish this, we will include the attacker's IP address and exclude the LFI attack:

```sql
"192.168.178.83" AND NOT "Fuzz"
```

#### Result

It appears that following the LFI attack, the attacker attempted another assault, this time focusing on **brute-forcing** the SSH credentials.

Key observations include:

1. A high volume of requests occurring each second.
2. Numerous instances of "Failed login for ironhack from 192.168.178.83," where the IP address 192.168.178.83 corresponds to the attacker.
3. The utilization of the ssh2 service in the attack.

![Second attack logs](images/second_attack_logs.png?raw=true "Second attack logs")


Now, our focus shifts to the SSH logs. Let's examine the frequency of the attacker's unsuccessful login attempts.

```sql
"Failed password for ironhack from 192.168.178.83" AND NOT "repeated"
```

#### Result

![Failed logins count](images/failed_logins_count.png?raw=true "Failed logins count")

Similarly, we can verify whether there are any SSH login requests other than those that resulted in a failed attempt.

```sql
"from 192.168.178.83" AND NOT "Failed"
```

#### Result

![Attacker Login](images/failed_logins_count.png?raw=true "Attacker Login")

The screenshot above indicates that the attacker successfully found the correct password through brute-forcing SSH.

---

In order ot answer the last question (#13), we just need to google `fail2ban`.

#### Result 

From a very quick search, we see that `fail2ban` is classified as an **Intrusion Prevention** software. Therefore, the correct answer is **Intrusion Prevention System**.

![fail2ban description](images/fail2ban.png?raw=true "fail2ban description")

### Questions and Answers

**Q:** What is the technique used for the first attack?\
**A:** brute-force.

**Q:** What is the name of the attack used by the technique intended above?\
**A:** Local File Inclusion.

**Q:** What is the name of the tool the attacker used to perform the attack?\
**A:** Fuzz Faster U Fool. 

**Q:** What is the IP address of the attacker?\
**A:** 192.168.178.83. 

**Q:** How many files did the attacker successfully expose?\
**A:** 97. 

**Q:** What's the technique used for the second attack?\
**A:** brute-force. 

**Q:** What is the service the attacker targeted for the second attack?\
**A:** ssh. 

**Q:** What is the username used by the attacker for the second attack?\
**A:** ironhack. 

**Q:** In which file, previously exposed, did the attacked find the username?\
**A:** /etc/passwd. 

**Q:** How many failed attempts has the attacker performed?\
**A:** ssh. 

**Q:** Was the attacker able to successfully log in into the server? (y/n)\
**A:** y. 

**Q:** What is the process ID (PID) associated with the SSH session in relation to the question above?\
**A:** 51362. 

**Q:** What mechanism would you implement to block such attack?\
**A:** Intrusion Prevention System. 
