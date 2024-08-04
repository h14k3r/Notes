[[resources]]
# General 

CSRF is when an attacker tricks a user into performing un-intended actions. It allows an attacker to partially bypass Cross-Origin interactions between different websites

> [!NOTE] port swigger
> Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.


> [!NOTE] 3 Conditions for it to work ( big 3 )
> 1. Relevant Action ( change username email -> attacker email)
> 2. Cookie-based session handling
> 3. No Unpredictable request parameters 


For example, suppose an application contains a function that lets the user change the email address on their account. When a user performs this action, they make an HTTP request like the following:

```
POST /email/change HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 30 
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE 

email=wiener@normal-user.com
```
-  meets the big 3


#### Example of payload


> [!NOTE] Malicious Script
> With these conditions in place, the attacker can construct a web page containing the following HTML:
> 
> The attacker would send a link for the user to visit the webpage that the attacker as created ( the script below )
> 
> Few things happen here: 
> - The attackers page will trigger an HTTP "request" to the vulnerable website.
> - If the User is logged into the vulnerable website, their browser will automatically include their session cookie in the request 
>   
>   ( assuming SameSite cookies are NOT being used )

```
<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

> [!NOTE] Note
> Although CSRF is normally described in relation to cookie-based session handling, it also arises in other contexts where the application automatically adds some user credentials to requests, such as HTTP Basic authentication and certificate-based authentication.
> 
## Impact

- Change email on users account from user@domain.com --> attacker@us.com
- Change Passwords
- Make funds transfer

*CAN EVEN GAIN FULL ACCESS OF USER ACCOUNT ( depending on the "action")*


## How to construct a CSRF attack

Manually creating the HTML needed for a CSRF exploit can be cumbersome, particularly where the desired request contains a large number of parameters, or there are other quirks in the request. The easiest way to construct a CSRF exploit is using the [CSRF PoC generator](https://portswigger.net/burp/documentation/desktop/tools/engagement-tools/generate-csrf-poc) that is built in to [Burp Suite Professional](https://portswigger.net/burp/pro):

- Select a request anywhere in Burp Suite Professional that you want to test or exploit.
- From the right-click context menu, select Engagement tools / Generate CSRF PoC.
- Burp Suite will generate some HTML that will trigger the selected request (minus cookies, which will be added automatically by the victim's browser).
- You can tweak various options in the CSRF PoC generator to fine-tune aspects of the attack. You might need to do this in some unusual situations to deal with quirky features of requests.
- Copy the generated HTML into a web page, view it in a browser that is logged in to the vulnerable website, and test whether the intended request is issued successfully and the desired action occurs.


---

# Labs


# Lab: CSRF vulnerability with no defenses

This lab's email change functionality is vulnerable to CSRF.

To solve the lab, craft some HTML that uses a [CSRF attack](https://portswigger.net/web-security/csrf) to change the viewer's email address and upload it to your exploit server.

You can log in to your own account using the following credentials: `wiener:peter`

#### Hint

You cannot register an email address that is already taken by another user. If you change your own email address while testing your exploit, make sure you use a different email address for the final exploit you deliver to the victim.



#### Solution

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. If you're using [Burp Suite Professional](https://portswigger.net/burp/pro), right-click on the request and select Engagement tools / Generate CSRF PoC. Enable the option to include an auto-submit script and click "Regenerate".
    
    Alternatively, if you're using [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload), use the following HTML template. You can get the request URL by right-clicking and selecting "Copy URL".
    
    `<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email"> <input type="hidden" name="email" value="anything%40web-security-academy.net"> </form> <script> document.forms[0].submit(); </script>`
3. Go to the exploit server, paste your exploit HTML into the "Body" section, and click "Store".
4. To verify that the exploit works, try it on yourself by clicking "View exploit" and then check the resulting HTTP request and response.
5. Change the email address in your exploit so that it doesn't match your own.
6. Click "Deliver to victim" to solve the lab.

## Start Lab



