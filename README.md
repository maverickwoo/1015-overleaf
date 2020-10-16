# Problem

A. This works (all ascii):

https://www.overleaf.com/docs?snip_uri=https://raw.githubusercontent.com/maverickwoo/1015-overleaf/main/a.tex

----

B. This breaks (non-ascii in filename):

https://www.overleaf.com/docs?snip_uri=https://raw.githubusercontent.com/maverickwoo/1015-overleaf/main/è.tex

# Solution Attempts

Python3 setup:
```py
>>> import urllib.parse
>>> url = r"https://raw.githubusercontent.com/maverickwoo/1015-overleaf/main/è.tex"
```


Note that in Python3, `/` is not encoded by default:
```
>>> help(urllib.parse.quote)
Help on function quote in module urllib.parse:

quote(string, safe='/', encoding=None, errors=None)
```
----

C. This breaks because the `%`'s in `%C3%A8` are not encoded; this is just like the
original example above:
```py
>>> urllib.parse.quote(url)
'https%3A//raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%C3%A8.tex'
```
https://www.overleaf.com/docs?snip_uri=https%3A//raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%C3%A8.tex

----

D. This breaks because the `:` in `url` become `%3A` in the inner call and then the
`%` in this `%3A` gets encoded in the outer call:
```py
>>> urllib.parse.quote(urllib.parse.quote(url))
'https%253A//raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%25C3%25A8.tex'
```
https://www.overleaf.com/docs?snip_uri=https%253A//raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%25C3%25A8.tex

----

E. This works because the `:` is not quoted in the inner call:
```py
>>> urllib.parse.quote(urllib.parse.quote(url, safe="/:"), safe="/")
'https%3A//raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%25C3%25A8.tex'
```
https://www.overleaf.com/docs?snip_uri=https%3A//raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%25C3%25A8.tex

----

F. This also works and looks somewhat better for human-written HTML:
```py
>>> urllib.parse.quote(urllib.parse.quote(url, safe="/:"), safe="/:")
'https://raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%25C3%25A8.tex'
```
https://www.overleaf.com/docs?snip_uri=https://raw.githubusercontent.com/maverickwoo/1015-overleaf/main/%25C3%25A8.tex

----

G. But really this is what should be done (and this is consistent with advice
from an Overleaf developer):
```py
>>> urllib.parse.quote(urllib.parse.quote(url, safe="/:"), safe="")
'https%3A%2F%2Fraw.githubusercontent.com%2Fmaverickwoo%2F1015-overleaf%2Fmain%2F%25C3%25A8.tex'
```
https://www.overleaf.com/docs?snip_uri=https%3A%2F%2Fraw.githubusercontent.com%2Fmaverickwoo%2F1015-overleaf%2Fmain%2F%25C3%25A8.tex
