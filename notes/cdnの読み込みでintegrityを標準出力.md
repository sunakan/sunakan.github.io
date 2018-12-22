```
$ shasum -b -a 512 FILE.js | awk '{ print $1 }' | xxd -r -p| base64
XXXXX
```

`sha512-XXXXX`
