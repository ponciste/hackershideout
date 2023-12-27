# Hacker's Hideout Write-Up

## Investigation

### First Attack

Let's investigate the oldest logs by browsing page number 50 of the logs.

Perform a standard search by typing `*` in the search bar and selecting `All time` in the date filter.

![Standard Search](images/all_logs_standard_search.png?raw=true "Standard Search")

From the logs below, we can deduce several key pieces of information about what transpired:

1. The date and time of the logs are consistent across multiple entries, indicating a **brute-force attack**.
2. All HTTP requests are of type GET, and the URLs vary for each log entry.
3. The URLs suggest that someone attempted a brute-force attack to **load a local path through a GET parameter**.
4. The name of a well-known brute-forcing tool; **Fuzz Faster U Fool v2.0.0-dev**.
5. The attacker's IP address.

![Logs page 50](images/oldest_logs.png?raw=true "Logs Page 50")

Based on this information, it becomes evident that, for this part of the analysis, we are dealing with a **brute-forced LFI (Local File Inclusion) attack**.

---

Next, we need to dig deeper into the LFI **possible damage**. Let's apply some filters to assess the impact of the LFI attack.

Let's apply the following search filter:

```sql
"GET" AND "200" AND "Fuzz Faster U Fool"
```

#### Result

We get **97** events (or log entries), which means, the hacker was successfully able to access the content of **97 files on the server**.

![GET 200 response filter](images/200_response_filter.png?raw=true "GET 200 response filter")

If we want to double-check it, we can easily try ourselves by copying and pasting the URL in the browser. 

#### Example

![Logs for GET 200 response](images/200_response.png?raw=true "Logs for GET 200 response")

**OS Information**

![/etc/os-release content](images/os-release.png?raw=true "/etc/os-release content")


**Users (/etc/passwd) Information**

![Users Information](images/passwd.png?raw=true "Users Information")

---

### Second Attack

In the challenge, a second attack is mentioned. Therefore, let's use the information we have retrieved so far to exclude the first attack.

To do this, we will include the attacker's IP address and exclude the LFI attack:

```sql
"192.168.178.83" AND NOT "Fuzz"
```

#### Result

It seems that, after the LFI attack, the attacker tried to **brute-force** again. This time however, the target was SSH.

Some of the information that stand out are:
1. Many requests each second.
2. Multiple "Failed login for ironhack from 192.168.178.83", which is the attacker's IP address.
3. The ssh2 service.

![Second attack logs](images/second_attack_logs.png?raw=true "Second attack logs")


Now, we need to focus on these SSH logs. Let's check how many times the attacker tried and failed to log in.

```sql
"Failed password for ironhack from 192.168.178.83" AND NOT "repeated"
```

#### Result

![Failed logins count](images/failed_logins_count.png?raw=true "Failed logins count")


Similarly, we can also verify whether or not the are different SSH login requests different than failed:

```sql
"from 192.168.178.83" AND NOT "Failed"
```

#### Result

![Attacker Login](images/failed_logins_count.png?raw=true "Attacker Login")

The screenshot above indicated that the attacker **was able to find the correct password by brute-forcing SSH**.


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

**Q:** Where did the attacked find the username?\
**A:** /etc/passwd. 

**Q:** How many failed attempts has the attacker performed?\
**A:** ssh. 

**Q:** Was the attacker able to successfully log in into the server? (y/n)\
**A:** y. 

**Q:** What is the sshd process ID of the log related to the successful attacker's log in?\
**A:** 51362. 

**Q:** What mechanism would you implement to block such attack?\
**A:** Intrusion Prevention System. 
