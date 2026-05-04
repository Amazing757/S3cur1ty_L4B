\# IDOR Vulnerability Write-up



\## Summary



This lab demonstrates an IDOR vulnerability.



The application stores transcript files with predictable filenames. The server returns the requested file without properly verifying whether the current user owns it.



As a result, another user's transcript may be accessed by modifying the file identifier in the request.



\## Root Cause



The server uses predictable object references and does not enforce proper server-side access control checks.



\## Impact



An attacker may access sensitive information belonging to other users.



\## Fix



The server should verify the authenticated user's permission before returning any transcript file. Access control checks should be enforced on every request.

