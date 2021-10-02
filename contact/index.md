# 

<!--
Simple contact me page
-->

The best way to get in touch is to email me :email:. 

There are a few [clever](https://github.com/martignoni/hugo-cloak-email) ways to [obfuscate](https://hugodutka.com/posts/email-obfuscation/) email addresses to prevent spam. Unfortunately, they all rely on javascript or otherwise can break screen readers and other tools that are often used by disabled persons.


So with that in mind, here's a simple algorithm to derive the best email to contact me

1. Visit the [home page]({{< ref "/" >}})
2. Split my name into `$first_name` and `$last_name`
3. Send your email to `contact@$fist_name$last_name.com`

If you had observed the name `Bobby Tables` on the home page, you would email `contact@bobbytables.com`.

### Security and Privacy related things

For sensitive topics, especially any security issues in any software I am responsible for, please use PGP to get in touch with me. The easiest way to do so is through [KeyBase](https://keybase.io/kquinsland)

