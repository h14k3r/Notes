### **Title:**

**Insecure Direct Object Reference (IDOR) in Password Update Function**

### **Description:**

The application allows users to update their passwords via an HTTP request. However, it does not properly enforce access controls on the "email" parameter, leading to an Insecure Direct Object Reference (IDOR) vulnerability. An attacker can intercept the request, modify the "email" parameter to any other user's email address, and successfully change that user's password. This can lead to full account compromise as the attacker gains control over the target account by changing its password.

### **Steps to Reproduce:**

1. **Login as a regular user** and navigate to the account settings page where you can update your password.
    
2. **Intercept the HTTP request** made when attempting to update the password using a tool like Burp Suite.
    
3. **Modify the "email" parameter** in the intercepted request to another user's email address. This effectively instructs the server to update the password for the targeted user's account.
    
4. **Send the modified request** to the server.
    
5. **Observe the response** from the server, which indicates that the password has been updated successfully for the targeted user's account, as shown in the screenshot.
    

### **Proof of Concept (PoC):**

- **Request**: The intercepted HTTP request shows the email parameter being modified to another user's email address (e.g., `"email": "test@test.com"`).
    
- **Response**: The server responds with a success message confirming that the password has been updated (e.g., `"message": "Password updated successfully"`).
    

**Screenshot Reference:**

- Include the screenshot you provided that shows the intercepted request in Burp Suite with the modified email and the corresponding successful response from the server.

### **Impact:**

- **Severity:** High
- This vulnerability allows an attacker to take control of any user’s account by changing their password without their consent. The attacker can then log in as the victim, potentially gaining access to sensitive data, performing actions on behalf of the user, or locking the legitimate user out of their account.

### **Recommendations:**

1. **Implement Proper Access Controls:** Ensure that the server-side logic checks that the email parameter corresponds to the authenticated user's email address before processing the password update request.
    
2. **Session Binding:** Instead of passing the email in the request body, rely on session identifiers to determine which user's password should be updated. This ensures that the operation is bound to the authenticated user's session.
    
3. **Log Suspicious Activities:** Implement logging and monitoring to detect and respond to suspicious activities, such as multiple password changes from the same IP address for different accounts.
    
4. **User Notification:** Notify users via email or SMS whenever their account password is changed to alert them of unauthorized actions promptly.
    

### **References:**

- OWASP Insecure Direct Object Reference (IDOR)
- OWASP Broken Access Control




![[updatedanotherpassword.png]]


