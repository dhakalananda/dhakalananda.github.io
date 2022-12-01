---
layout: post
title: "wpForo Forum Security Advisories"
categories: "codereview"
---

Technical analysis of multiple vulnerabilities found on wpForo Forum plugin in Wordpress.

This blog contains a technical analysis of multiple vulnerabilities found in wpForo wordpress plugin. wpForo Forum is a plugin that allows users to add forums in their Wordpress site. As per their [website][wpforo], they are the #1 Wordpress forum plugin.

- Vulnerabilities Found: CSRF, IDOR
- Vulnerable Plugin: [wpForo][wpforo]
- Active installations: 20,000+

---

### **CSRF - Account Deletion**

---

**Summary**

A Cross Site Request Forgery was found on the account delete functionality. Due to the lack of use of `wp_nonce` on the `user_delete` action, arbitrary users could have been deleted by a malicious actor.

- Affected Version: <=2.0.9
- Patched Version: 2.1.0
- CVSS Score: 7.5
- CVE: [CVE-2022-40192][cve-2022-40192]

**Vulnerability Discovery**

The vulnerability was relatively easy to find. I navigated to a user's profile from the admin's account and inspected the element for the Delete Account functionality. It was a simple URL without any CSRF protection.

![Inspect-Element](/assets/wpforo-inspect-element.png)

**Proof of Concept**

1. Go to the URL `http://VULNERBALE-HOST/participant/USER/?wpfaction=user_delete` from the admin's account where `USER` is the username of the account that needs to be deleted.

**PoC HTML Code**

```
<html>
  <body>
    <form action="http://localhost/wordpress/participant/USER/">
      <input type="hidden" name="wpfaction" value="user&#95;delete" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```

**Chaining for 0 user interaction**

In the account edit section of wpForo, there is a feature where you can upload your avatar via image. A user can enter any URL as their image URL. A glimpse of account settings:

![ImageURL](/assets/wpforo-image-url-upload.png)

<center><i>Fig: wpForo account edit settings</i></center>

<br>

If the attacker enters the URL `http://VULNERBALE-HOST/participant/test/?wpfaction=user_delete` in the image URL, a request is sent to the image URL when the admin visits `http://localhost/wordpress/wp-admin/users.php`. The admin's cookie is used for the request, leading to account deletion without any user interaction.

**Impact**

A successful exploitation can lead to account deletion of any user without any user interaction.

---

### **IDOR - Private/Public & Solve/Unsolve Topics**

---

**Summary**

Multiple IDOR vulnerabilities were found on functionalities that allowed a malicious actor to mark topics as private/public or solved/unsolved. Due to the lack of permission check whether the request is sent by an admin/creator or non-privileged user, the attack was possible.

This section discusses about private/public functionality only [[CVE-2022-40206][cve-2022-40206]], as the code and design of both functionality is the same.

- Affected Version: <=2.0.5
- Patched Version: 2.0.6
- CVSS Score: 6.3, 5.4
- CVE: [CVE-2022-40206][cve-2022-40206], [CVE-2022-40205][cve-2022-40205]

**Vulnerability Discovery**

After looking into the application and analyzing the source code side-by-side for quite a while, I came across a functionality that allows admin to mark topics private. The request looked like:

![Mark-Private-Topic](/assets/wpforo-mark-topic-private.png)

Looking at the `action` parameter in the above request, a request is sent to the server that calls the action `wpforo_private_ajax`. I looked for that action in source code of the plugin. It was hooked with the function `wpf_private`.

![Source-Code-Private-Function](/assets/wpforo-public-private-topic.png)

Analyzing the code of the function, I noticed that there were no access-control measures used. The function `wprivate` was executed to mark the topic as private. Access control might have been placed inside that function as well, so I had to analyze the code of `wprivate` as well.

![wprivate-code](/assets/wpforo-wprivate-code.png)

There was no check whether the request sender had the sufficient privilege to make the action, making it vulnerable to IDOR.

_As there is no `nonce `implemented, this is vulnerable to CSRF also. Almost all ajax functions were vulnerable to CSRF, but it was already reported by someone else, and a fix was released on the later version._

**Proof of Concept**

1. Capture the request of making your own topic private
2. Change the `topicid` to another user's topic ID
3. The topic will be marked as private
4. The same applies to making topics public, solving and, unsolving topics

**Impact**

If brute-forced all IDs, the exploitation of attack can lead to all topics being private/public or solved/unsolved.

**Mitigation**

wpForo added permission check in the function (Line: 711 - 713).

![IDOR-Fix](/assets/wpforo-idor-patch.png)

_There is an extra line of code on line number 701 `wpforo_verify_nonce( 'wpforo_private_ajax')`. This successfully fixes the CSRF vulnerability._

---

One more CVE was assigned for a CSRF on ajax function to delete post/topic [CVE-2022-40632][cve-2022-40632] as it was missed in the mass CSRF report/fix.

Thank you for reading!

[wpforo]: https://wordpress.org/plugins/wpforo
[cve-2022-40206]: https://patchstack.com/database/vulnerability/wpforo/wordpress-wpforo-forum-plugin-2-0-5-insecure-direct-object-references-idor-vulnerability
[cve-2022-40205]: https://patchstack.com/database/vulnerability/wpforo/wordpress-wpforo-forum-plugin-2-0-5-insecure-direct-object-references-idor-vulnerability-2
[cve-2022-40632]: https://patchstack.com/database/vulnerability/wpforo/wordpress-wpforo-forum-plugin-2-0-5-cross-site-request-forgery-csrf-vulnerability-2
[wpforo]: https://wpforo.com/
[cve-2022-40192]: https://nvd.nist.gov/vuln/detail/CVE-2022-40192
