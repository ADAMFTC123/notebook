

#### level06
Here you can see the source code, from this check i guess we need to access the `localhost` and retrieve `flag.php`, but not using `localhost`, rather using `127.0.0.1`.

```
 /* People tends to do funny things with curl. */
if (preg_match ('/[https?|[st]?ftp|dict|gopher|scp|telnet|ldaps?]\:\/\/.*(\d+|[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3})/i', $url)) {
    die('Please do not access by IP.');
} elseif (preg_match ('/localhost/i', $url)) {
    die ('Please do not access localhost.'); 
}
```

Also, we can see here that it appends `index.php` to the url.

```
if (stripos ($url, '/', -1) !== '/') { $url .= '/'; }
    $url .= 'index.php';
```

However, we still get _Nope. Not allowed here._, WHAT IS THAT?

Okay, let’s try use `file` schema: `file:///flag.php?`


![Final FLAG](https://avishaigonen123.github.io/CTF_writeups/websec.fr/images/level06.png)

We can see what denied us from accessing using `http` to `127.0.0.1`:

```
if(!defined('ACCESS_ONLY')) {
    die('Nope. Not allowed here.');
}
```

Here you can see how it works [using defined to prevent access - stackoverflow](https://stackoverflow.com/a/409515).

**Flag:** **_`WEBSEC{SSRFToReadArbitraryFiles}`_**

#### What’s happening (step‑by‑step)

1. You submit the form in the browser with a URL (e.g. `file:///var/www/flag.php`).
    
2. **The server** (the PHP page with the cURL code) receives that form and runs `curl_init($url)` — so _the server_ performs the request, not your browser.
    
3. `file:///...` is a **file URI**/stream wrapper: when cURL (running on the server) sees `file://`, it uses the operating system / PHP file wrapper to **read the server’s local filesystem**, returning the raw contents of the file. This is _local_ to the server.
    
4. For an HTTP request like `http://127.0.0.1/flag.php`, the webserver would execute `flag.php` as PHP and send the _output_ — and because `flag.php` contains:
    
    `if(!defined('ACCESS_ONLY')) {     die('Nope. Not allowed here.'); }`
    
    a direct HTTP request will cause PHP to run that check (constant not defined) and die — so you don’t get the flag over HTTP.
    
5. But `file:///var/www/flag.php` does **not** invoke the webserver/PHP processor — it just reads the file contents. You get the source (including the `die()` guard and any secret present in the file), because reading the file bypasses the webserver’s execution step entirely.
    

So, when you “fetch” `file://` through the server’s cURL, that cURL process is acting locally on the server’s filesystem — that’s why it works even though you’re sitting on a remote client. You aren’t actually “on the server” — the server is reading itself on your behalf.

#### Why the `defined('ACCESS_ONLY')` guard doesn’t help here

- The guard prevents **execution** of the PHP file via the webserver/PHP interpreter (i.e. `http://…/flag.php`).
    
- It does **not** prevent the raw file from being read off disk. `file://` reads the file bytes; it never runs the `if` check because the PHP interpreter is not involved.
    

#### Why the regex / IP checks don’t block it

- Your code is checking for IPs / `localhost` in the submitted URL using a couple of `preg_match` rules crafted for typical network protocols. Those checks look for protocols like `http`/`ftp`/`scp`/etc. They don’t detect or explicitly forbid the `file` scheme. So `file:///…` passes those checks.
    
- Also note: even blocking `127.0.0.1` in the hostname won’t stop a `file://` fetch because there is _no_ network resolution stage — it’s direct file access.
#### What I mean by “the server”

When I say **“the server”** I mean **the machine (and webserver process) that’s running your PHP code** — e.g. the computer where `grabber.php` lives, typically running Apache/Nginx + PHP-FPM.  
Flow of events:

1. Your **browser (client)** submits the form to `https://example.com/grabber.php` (that domain is the _server_).
    
2. The webserver on **that machine** receives the POST and runs `grabber.php` (the PHP interpreter executes the script).
    
3. Inside `grabber.php` you call `curl_init($url)` — _that cURL runs on the same machine as step 2_, so cURL’s network/file access happens **from the server**, not from your browser.
#### Key point: who executes the request matters

- If you (the client) open `http://example.com/flag.php` in your browser, the webserver runs `flag.php` under PHP and the `if(!defined('ACCESS_ONLY'))` guard executes — so you get blocked.
    
- If the server itself reads `file:///var/www/flag.php`, it **does not** execute it — it just returns the source file content. That’s why the guard can’t stop it.


#### Quick examples to illustrate

Client → Server request:

`Browser POST -> https://example.com/grabber.php (runs on server) grabber.php: curl_init('file:///var/www/flag.php')  <-- executed by PHP on server Result: server reads /var/www/flag.php and returns contents to browser`

Client direct access:

`Browser GET -> https://example.com/flag.php  (webserver runs flag.php under PHP) flag.php: if(!defined('ACCESS_ONLY')) die();  <-- guard executes, blocks access`