# PFSense

Log of PFSense notes.

## PFBlockerNG

Make sure your DNS servers on linux are actually pointing to your PFSense router.

```bash
cat /etc/resolv.conf
```

Check to see if a domain is blocked or not by using the `drill` command from the ldns package.

The result of a non blocked website will return its correct IP from its domain name.

Example of successful domain resolution (domain **isnt** blocked)

```bash
drill isitblocked.org
```

```bash
;; ANSWER SECTION:
isitblocked.org.	3126	IN	A	74.208.236.124
```

Example of unsuccessful domain resolution (domain **is** blocked).

The returned IP is the virtual IP of PFBlockerNG.

```bash
drill analytics.163.com
```

```bash
;; ANSWER SECTION:
analytics.163.com.	59	IN	A	10.10.10.1
```
