# blogs-oracle
my old blogs from blogs.oracle.com/vlad/

## Getting the blogs

It seesm that `wget -m` cannot be used as the content provider does not seem to like HTTP/1.1 requests and only like HTTP/2 so I got the list of blog entries first:

```
curl -o vlad.html -L -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: cs,sk;q=0.8,en-US;q=0.5,en;q=0.3' --user-agent 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:76.0) Gecko/20100101 Firefox/76.0' --http2 http://blogs.oracle.com/vlad/
grep 'data-url="https://blogs.oracle.com/vlad/' vlad.html | awk -F 'data-url=' '{ print $2; }' | grep data-text | cut -d" " -f1 | sed 's/"//g' > vlad-urls.txt
```

and then extracted the HTML pages of each blog:

```
cat vlad-urls.txt | while read url; do curl -O -L -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: cs,sk;q=0.8,en-US;q=0.5,en;q=0.3' --user-agent 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:76.0) Gecko/20100101 Firefox/76.0' --http2 $url; done
ls -1 *.html | while read f; do cat $f | sed -n '/<div class="cb11v2-posturltracking">/,/<script type="text\/javascript">/p' | sed '1d;$d' > $f.extract; done
ls *.extract | while read f; do cat $f | awk 'BEGIN { print "<html><body>"; } END { print "</body></html>" } // { print; }' > $f.new; done
ls *.new | while read f; do mv $f `echo $f | sed 's/\.extract.*//'`; done
rm *.extract
```
