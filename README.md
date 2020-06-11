# blogs-oracle

This repository stores my old blogs from blogs.oracle.com/vlad/ that are on the way to be deleted since I no longer blog there.
The blog was called "Czech techie's adventures".

Most of these blogs contain outdated information however these are sort of important for me personally because they contain
tidbits of what I was up to during these times.

## Getting the blogs

It seems that `wget -m` cannot be used as the content provider does not seem to like HTTP/1.1 requests and only like HTTP/2 so I got the list of blog entries first:

```
curl -o vlad.html -L -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: cs,sk;q=0.8,en-US;q=0.5,en;q=0.3' --user-agent 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:76.0) Gecko/20100101 Firefox/76.0' --http2 http://blogs.oracle.com/vlad/
grep 'data-url="https://blogs.oracle.com/vlad/' vlad.html | awk -F 'data-url=' '{ print $2; }' | grep data-text | cut -d" " -f1 | sed 's/"//g' > vlad-urls.txt
```

and then extracted the HTML pages of each blog:

```
mkdir out
cd out
cat ../vlad-urls.txt | while read url; do curl -O -L -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: cs,sk;q=0.8,en-US;q=0.5,en;q=0.3' --user-agent 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:76.0) Gecko/20100101 Firefox/76.0' --http2 $url; done
ls -1 | while read f; do mv $f $f.orig; done
ls -1 *.orig | while read f; do egrep '(<title>)|(meta name="publish_date")' $f > `echo $f | sed 's/\.orig//'`.title; done
ls -1 *.title | while read f; do ( echo "<head>"; cat $f; echo "</head>" ) > `echo $f | sed 's/\.title/\.header/'`; done
ls -1 *.orig | while read f; do cat $f | sed -n '/<div class="cb11v2-posturltracking">/,/<script type="text\/javascript">/p' | sed '1d;$d' > `echo $f | sed 's/\.orig/\.extract/'`; done
ls *.extract | while read f; do cat `echo $f | sed 's/\.extract/\.header/'` $f | awk 'BEGIN { print "<html><body>"; } END { print "</body></html>" } // { print; }' > `echo $f | sed 's/\.extract/\.new/'`; done
mkdir ../new
ls *.new | while read f; do cp $f ../new/`echo $f | sed 's/\.new/\.html/'`; done
```
