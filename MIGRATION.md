Migration
=========


MySQL
-----

```
ALTER TABLE user CHANGE geburtstag geburtstag DATE NOT NULL DEFAULT '1970-01-01';
```

POSTFIX
-------

```
cp -f /etc/services /data/spool/postfix/etc/
```
