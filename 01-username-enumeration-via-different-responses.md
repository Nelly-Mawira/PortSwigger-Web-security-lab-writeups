# PortSwigger Lab Write-up: Username Enumeration via Different Responses

**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Category:** Authentication — Password-based authentication  
**Lab:** [Username enumeration via different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)  
**Status:** Solved  
**Tools:** Burp Suite Community Edition (Proxy, Repeater, Intruder), Kali Linux

## Objective

Identify a valid account name from differences in login error messages, find its password using the candidate lists provided for the lab, and log into the account. Testing was limited to my assigned PortSwigger training lab.

## Vulnerability overview

The login endpoint disclosed whether a submitted username existed. When a username was unknown, the application responded with **`Invalid username`**. For an existing username paired with an incorrect password, it responded with **`Invalid password`**. These distinct responses let an unauthenticated tester distinguish valid accounts from invalid ones.

A successful HTTP request does not imply a successful login: **HTTP 200** just means the server returned a response. The contents of the login response mattered more than the status code during enumeration.

## Methodology

### 1. Capture the login request

I opened the lab in Burp's built-in browser, visited **My account**, and submitted deliberately invalid credentials. Under **Proxy → HTTP history**, I found the `POST /login` request and sent it to **Intruder**.

Example request body (illustrative invalid credentials):

```http
username=testuser999&password=wrongpassword123
```

### 2. Enumerate usernames

In Intruder, I used the **Sniper** attack type and selected only the `username` value as the payload position:

```http
username=§testuser999§&password=wrongpassword123
```

I loaded the lab's **Candidate usernames** list and compared the responses. Most returned `Invalid username`, with a recorded response length of **3352 bytes**. One candidate, **`arlington`**, produced a different response length (**3354 bytes**) and displayed **`Invalid password`**.

That difference identified a valid username in the lab.

### 3. Test the lab's candidate passwords

I kept the discovered username fixed and moved the Intruder payload position to the password:

```http
username=arlington&password=§wrongpassword123§
```

I loaded the lab's **Candidate passwords** list. Most attempts returned **HTTP 200** with the login failure page. One result stood out:

| Signal | Observed result |
|---|---|
| Password candidate | `qazwsx` |
| HTTP status | `302 Found` |
| Response length | 184 bytes |

The **302** redirect was a strong indication that the password had been accepted. I then verified the credentials by signing in through the lab browser and confirmed the lab was marked **Solved**.

## Root cause

The application returned distinguishable error messages for **unknown username** and **known username with incorrect password**. This exposed account-existence information and reduced the effort needed to test passwords against a real user.

## Remediation

- Return a consistent, generic message such as **`Invalid username or password`** for failed logins.
- Keep response behavior as uniform as practical, including status codes and timing.
- Rate-limit authentication attempts and monitor suspicious bursts.
- Consider multi-factor authentication and additional defenses against credential attacks.

## Key takeaways

- A `200 OK` response is not evidence that authentication succeeded.
- Response **body differences**, **length**, and **redirects** can reveal distinct authentication outcomes.
- Confirm unusual Intruder results by reading the actual response, not just its byte length.
- Repeating tests against an authorized training environment can demonstrate the security impact of overly specific login error messages.

## Evidence

You can optionally add your own redacted screenshots here:

- Intruder username results showing the outlier response.
- Intruder password results showing the `302` response.
- PortSwigger **Lab solved** banner.

Before publishing screenshots, remove session cookies, tokens, and unrelated personal information. Do not commit the full lab wordlists.

---

*This write-up documents testing of an intentionally vulnerable PortSwigger Web Security Academy lab for educational purposes.*
