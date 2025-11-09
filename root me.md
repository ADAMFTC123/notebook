sudo find / -perm -4000 -type f 2>/dev/null -exec ls -la {} \;

Rule of thumb for weird SUID files

Anything in non-standard directories (/home, /tmp) → suspicious.

Anything not owned by root → suspicious (except some legit system users like daemon, messagebus, dip).

Interpreted scripts (like .sh or .py) with SUID → almost always weird.