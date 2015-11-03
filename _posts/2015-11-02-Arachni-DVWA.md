---
layout: post
title: Scanning DVWA With Arachni
---

DVWA (Damn Vulnerable Web App) is a puposefully vulnerable web
application that is a teaching tool for exploiting common web application
vulnerabilities. These vulnerabilties include SQL Injection,
Cross-site Scripting, Remote File Inclusion, Command Execution, and
various Information Disclosure vulnerabilities. Although DVWA was not
designed to be used to evaluate web application vulnerability scanners,
it may be typically used and scanners can run into configuration issues
preventing a successfuly scan of DVWA. We will look at some of the
common configuration issues you may run into with scanning DVWA with the
Arachni web application vulnerability scanner.

The first issue that you may run into is authenticating to DVWA with
Arachni. The auto-login plugin can be challenging to get configured
correctly and it is typically easier to use a login script to
authenticate Arachni:

login.rb:

```
response = http.post( 'http://127.0.0.2/dvwa/login.php',
    parameters:     {
        'username'   => 'user',
        'password' => 'user',
        'Login' => 'Login'
    },
    mode:           :sync,
    update_cookies: true
)

framework.options.session.check_url     = to_absolute(
response.headers.location, response.url )
framework.options.session.check_pattern = 'Logout'
```

Various pages throughout DVWA can trigger conditions where the session
will be invalidated or where web application defensive measures (i.e.
PHPIDS) can be triggered. By running the Arachni command line client with
the following exclusions, you can typically get great results:

```
./bin/arachni --plugin=login_script:script=/tmp/login.rb \
 --checks=* \
 --scope-exclude-pattern='\/dvwa\/logout\.php' \
 --scope-exclude-pattern='\/dvwa\/vulnerabilities\/csrf\/' \
 --scope-exclude-pattern='\/dvwa\/security\.php' \
 --scope-exclude-pattern='\/dvwa\/setup\.php' \
 --http-cookie-string="security=low" \ 
 --report-save-path '~/DVWA.afr' \
 http://127.0.0.2/dvwa/
```

Here's an example of some of the vulnerabilities found:
![arachni dvwa scan results]({{ site.baseurl }}/images/arachni.png
"Arachni Results")

