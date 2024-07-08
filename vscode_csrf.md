## Guide 
Cross-Site Request Forgery (CSRF)

*how to*

Attack is on users(victim)
1. attacker sends link
2. User clicks on the link while logged into the app

    - victim must be logged into the app
    - perform actions using the victim as a proxy
    - send victim a malicous link > {change password link is sent to the attacker instead of the victim}
 
**poc** 
    Attacker sends link with our email to the victim > victim clicks on it, and the link attaches to the session cookie > that session cookie is sent to the backend, which reads "attacker@domain.com > then the backend sends back the info to the attacker, instead of the victim > attacker uses the "forgot password" function.

    Attacker sends a webpage while the invisble 'iframe' is running in the background, which is the CSRF script > which changes the email address for the attacker

*if*

    Relevant action: 
        Must be able to do something > {email function:}

    Cookie-Based session handling: 
        Must use cookies for sesesion management
    
    No unpredictable request parameters

*method*

    1. Run an Auto Scan with Burp-Pro
    2. Map the App > review findings > do any meet the requirements from **if > create PoC script to teste the exploit
        2.a: PoC script will be run with a GET request:
                GET:: <img> tag with src attribute set to vulnerable URL
                POST:: form with hidden fields for all the required parameters and the target set to vulnerable URL

*rem*

    adding CSRF tokens



*resources*
https://owasp.org/www-community/attacks/csrf
 

https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
