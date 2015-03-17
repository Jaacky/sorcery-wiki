Welcome to the Sorcery wiki!

This wiki is a work in progress, and constantly improving.


[[Known incompatibilities]]


### Summary:


Sorcery is a stripped-down, bare-bones authentication library, with which you can write your own authentication flow.
It was built with a few goals in mind:

* Less is more - less than 20 public methods to remember for the entire feature-set make the library easy to 'get'.
* No built-in or generated code - use the library's methods inside *your own* MVC structures, and don't fight to fix someone else's.
* Magic yes, Voodoo no - the lib should be easy to hack for most developers.
* Configuration over Confusion - Simple & short configuration as possible, not drowning in syntactic sugar.
* Keep MVC cleanly separated - DB is for models, sessions are for controllers. Models stay unaware of sessions.


Hopefully, I've achieved this. If not, let me know.



You can see how controllers, models, migrations and views can be used in Rails 3 with Sorcery by checking out [this example application](https://github.com/NoamB/sorcery-example-app).

**There is also a mongoid branch at the same URL, showing how you can use Mongoid as your ORM.**

Rails Tutorials:

[[Simple Password Authentication]] | [[Remember Me]] | [[Session Timeout]]

[[User Activation]] | [[HTTP Basic Auth]] | [[Reset Password]] | [[Password-less Activation]]

[[Activity Logging]] | [[Brute Force Protection]] | [[External]]

[[Routes Constraints]] | [[Fetching currently active users]]

Testing Tutorials:

[[Testing Rails]]

Tutorials in another sites:

[Magical Authentication with Sorcery](http://www.sitepoint.com/magical-authentication-sorcery/) by Ilya Bodrov on Sitepoint.com