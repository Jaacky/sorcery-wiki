# Changelog

***
# 0.1.3

TODO

# 0.1.2

TODO

# 0.1.1

TODO

# 0.1.0

TODO

## Core Features:
* login/logout, optional redirect on login to where the user tried to reach before, configurable redirect for non-logged-in users.
* password encryption, algorithms: bcrypt(default), md5, sha1, sha256, sha512, aes256, custom(yours!), none. Configurable stretches and salt.
* configurable attribute names for username, password and email.
## User Activation:
* User activation by email with optional success email.
* configurable attribute names.
* configurable mailer.
* Optionally prevent active users to login.
## Password Reset:
* Reset password with email verification.
* configurable mailer, method name, and attribute name.
## Remember Me:
* Remember me with configurable expiration.
* configurable attribute names.
## Session Timeout:
* Configurable session timeout.
* Optionally session timeout will be calculated from last user action.
## Brute Force Protection:
* Brute force login hammering protection.
* configurable logins before ban, logins within time period before ban, ban time and ban action.