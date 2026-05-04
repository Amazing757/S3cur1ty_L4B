\# PortSwigger Lab: Insecure Direct Object References (IDOR)



\## Summary



This write-up is based on the PortSwigger Web Security Academy lab:



\*\*Access control vulnerabilities → Insecure direct object references\*\*



PortSwigger Web Security Academy is a free web security training platform provided by Burp Suite.



To find the lab, open PortSwigger Web Security Academy, go to \*\*All labs\*\*, and select \*\*Access control vulnerabilities\*\*.



!\[Access control lab list](<images/01-Start01bp\_Access control vulnerabilities\_IDOR.png>)



Find the specific lab: \*\*Insecure direct object references (IDOR)\*\*.



!\[IDOR lab list](<images/02-Start02bp\_Access control vulnerabilities\_IDOR.png>)



\## Lab Description



The lab stores user chat logs directly on the server's file system and retrieves them using static URLs.



The goal is to find the password for the user `carlos` and log in to his account.



!\[Lab description](<images/03-Access\_lab\_bp\_Access control vulnerabilities\_IDOR.png>)



\## Step 1: Access the Lab



Click \*\*Access the lab\*\* to enter the target website.



After entering the lab, open the \*\*Live chat\*\* page and send a message to the support bot.



!\[Live chat page](<images/04-Live\_chat\_bp\_Access control vulnerabilities\_IDOR.png>)



\## Step 2: Check HTTP History in Burp Suite



Go back to Burp Suite and open \*\*Proxy → HTTP history\*\*.



In the HTTP history, the requests related to `/viewTranscript.js` and `/chat` can be observed.



!\[HTTP history](<images/06-history\_chat1\_bp\_Access control vulnerabilities\_IDOR.png>)



The `viewTranscript` function shows that the current chat content is sent to a transcript download path.



The frontend JavaScript sends a POST request to `downloadTranscriptPath`:



```javascript

xhr.open("POST", downloadTranscriptPath);

data.append("transcript", transcript.join("<br/>"));

xhr.send(data);

```



!\[Transcript JavaScript](<images/07-history\_chat2\_bp\_Access control vulnerabilities\_IDOR.png>)



\## Step 3: Find the Download Path



On the `/chat` page, the HTML and JavaScript reveal the transcript download function.



The interesting keywords are:



```text

viewTranscript

download-transcript

.txt

```



The transcript download path is:



```text

/download-transcript/

```



Although the page does not directly show a `.txt` file, we can guess that the transcript file may be stored using a predictable static filename, such as:



```text

1.txt

```



\## Step 4: Access the Transcript File



Manually construct the following URL:



```text

/download-transcript/1.txt

```



The server returns a transcript file.



This confirms that the chat transcript files are stored with predictable filenames and can be directly accessed.



!\[Download transcript](<images/05-Download\_transcript\_bp\_Access control vulnerabilities\_IDOR.png>)



\## Step 5: Find Carlos' Password



The downloaded transcript contains a chat history.



Inside the transcript, there is a password belonging to the user `carlos`.



!\[Password found](<images/08-passwd\_found\_bp\_Access control vulnerabilities\_IDOR.png>)



This is a dynamic lab password, so do not directly copy the password shown in another person's screenshot.



\## Step 6: Log in as Carlos



Return to the login page.



Use the username:



```text

carlos

```



and the password found in the transcript file to log in.



!\[Return to login](<images/09-return\_login\_bp\_Access control vulnerabilities\_IDOR.png>)



After logging in successfully, the lab is solved.



!\[Login success](<images/10-Login-success\_bp\_Access control vulnerabilities\_IDOR.png>)



\## Root Cause



The application stores transcript files in a predictable location:



```text

/download-transcript/

```



The filenames are also predictable:



```text

1.txt

2.txt

3.txt

...

```



However, the backend does not properly verify whether the current user is authorized to access the requested transcript file.



Because of this missing server-side access control check, an IDOR vulnerability appears.



\## Impact



An attacker may access sensitive chat transcript files belonging to other users.



This may lead to:



\- Information disclosure

\- Credential leakage

\- Account compromise



\## Remediation



To fix this vulnerability:



1\. Do not expose sensitive files through predictable filenames.

2\. Enforce server-side authorization checks on every transcript request.

3\. Verify that the authenticated user owns the requested transcript before returning it.

4\. Do not rely only on frontend controls or hidden URLs.

5\. Even if random filenames are used, backend authorization checks are still required.

