# Penetration Test of the P-Cart Web Application
**Tester:** Dylan Leggio
**Client:** Fabio Codebue c/o P-Soft
**Report Date:** Wednesday, December 27 2017

## Executive Summary
On December 25, 2017, Fabio Codebue of P-Soft Web Services contracted Dylan Leggio to perform a penetration test against the P-Cart framework in exchange for the rights to the domain name “legg.io”. P-Cart is a Content Management System (CMS) and e-commerce framework built by Fabio and his company, P-Soft. The purpose of the penetration test was to find security flaws and issues that could result in monetary and legal harm for P-Soft.

## Findings and Recommendations
P-Cart was developed using the **Codeigniter** web framework, and most of the security issues found in this test had to do with the configuration of the web framework itself. **Codeigniter** abstracts a lot of the security implementation away from the programmer, and automatically implements security mechanisms to protect the web application from attackers. These security mechanisms can be enabled or disabled in the application configuration files. Most of these were **disabled** or not implemented in P-Cart’s configuration files. In a production environment, this would leave P-Cart vulnerable to Cross-Site Scripting and Cross-Site Request Forgery attacks, among others. Included in this report are **Codeigniter** security mechanisms that P-Cart did not enable, detailed desciptions of attacks that this would leave P-Cart vulnerable to, instructions on how to edit the configuration files so that these mechanisms are enabled, and other recommendations that would make P-Cart more secure for its users.

## Issues & Vulnerabilities

### `/p-cart.lan/index.php`

#### Application Environment
The `environment` variable of the application should be set to `production`, so that PHP errors are not displayed to users. This limits possible informative errors that can give an attacker more information about a web server to further his attack.
##### Fix: Edit the following line in the `index.php` file:
**Uncomment the line:**
```php
define('ENVIRONMENT', isset($_SERVER['CI_ENV']) ? $_SERVER['CI_ENV'] : 'production');
```
**Comment the line:**
```php
//define('ENVIRONMENT', isset($_SERVER['CI_ENV']) ? $_SERVER['CI_ENV'] : 'development');
```

#### Access Control
The `/p-cart.lan/p-cms` and `/p-cart.lan/system` folders should be moved above the web root so that they are not directly accessible by user’s browsers. If these are kept in the web root, an attacker would be able to access sensitive files that could compromise the web application.

##### Fix:
**My web root (yours may differ):** `/var/www/http/`
**Move** `/var/www/http/p-cart.lan/p-cms` **->** `/var/www/p-cms`
**Move** `/var/www/http/p-cart.lan/system` **->** `/var/www/system`
**Edit the following lines in the `index.php` file:**
```php
$system_path = '/var/www/system';
$application_folder = '/var/www/p-cms';
```
More information can be found at https://codeigniter.com/user_guide/general/security.html#hide-your-files

### `p-cart.lan/p-cms/config/config.php`


#### Allowed URL Characters
The list of permitted URL characters was left blank. This would allow an attacker to use any characters in URL’s, which could allow them to evade security filters and potentially execute Cross-Site Scripting attacks. Cross-Site Scripting (https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)) is an attack where an attacker injects his own malicious JavaScript code into the web page, which is executed on a victim’s machine, resulting in the attacker stealing the victim’s session cookies or temporarily keylogging them.

##### Fix: Edit the following line into the `config.php` file:

```php
$config['permitted_uri_chars'] = 'a-z 0-9~%.:_-';
```

#### Encryption Key
The key used in the configuration file `IoM8Gugk1K3lkYb6Vba8wJo0EjN64ZHW` was also found to be in use at https://github.com/havok89/Hoosk/blob/master/hoosk/hoosk0/config/config.php#L308. A new key should be set so that data is securely encrypted and no one else has the key except you, the owner.
##### Fix:
```php
// Get a hex-encoded representation of the key:
$key = bin2hex($this->encryption->create_key(32));

// Put the same value in your config with hex2bin(),
// so that it is still passed as binary to the library:
$config['encryption_key'] = hex2bin(<your hex-encoded key>);
```

#### Session Variables
The session cookies were configured in a way that they expired after too long an amount of time, did not expire when a user closed their browser, and were not matched with a user’s IP. This could allow an attacker on a public machine to use another user’s session after they logged off the machine, or use their account remotely if they grabbed the victim’s session cookies through a Cross-Site Scripting attack. It is recommended to decrease the expiration time of session cookies and make them expire when a user closes their browser. Additionally, the session cookie was not configured to be encrypted, which could allow an attacker to brute force someone else’s session cookie and take over their account.
##### Fix: Edit the following lines into the `config.php` file:
```php
$config['sess_expiration']      = 3600;
$config['sess_expire_on_close'] = TRUE;
$config['sess_encrypt_cookie']  = TRUE;
$config['sess_match_ip']        = TRUE;
```

#### Session Related Variables
The session cookies were not set with the `HTTP_ONLY` flag. If an attacker found a Cross-Site Scripting vulnerability, this would allow them to steal a victim’s cookies over JavScript. Enabling this flag makes this impossible, and forces the cookies to only be sent over HTTP. JavaScript won’t be able to view the cookies if this is set.
##### Fix: Edit the following line into the config.php file:
```php
$config['cookie_httponly'] 	= TRUE;
```

#### Cross Site Request Forgery
Cross-Site Request Forgery (CSRF) protection and CSRF cookies were not enabled in the configuration file. This would allow an attacker to target victims with CSRF attacks, in which an attacker sends the victim a link, which makes them perform an unwanted action on the web site (purchase items, change their password to the attacker’s password, etc). More about CSRF can be read at https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF). CSRF cookies prevent this attack by using a random, secret token for each form that an attacker cannot guess.
##### Fix: Edit the following into the `config.php` file:
```php
$config['csrf_protection'] = FALSE;
```




