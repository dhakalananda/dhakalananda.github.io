---
title: "The Silent Death of Open-Source Bug Bounty"
description: "Open-source bug bounty programs are on the verge of dying. Nobody is talking about it."
date: 2026-04-24
---

My thoughts on the current state of open-source bug bounty along with a peek into Patchstack bug bounty program.

## Introduction

It all started in late November 2025, when Claude Opus 4.5 launched and people quickly realized it was capable of finding security issues. Everyone started using it to find bugs. By December, the flood had begun.

What we're seeing right now is the aftermath of the consecutive launches of Claude Opus 4.6 and 4.7, with people using them to flood bug bounty programs.

## Curl - The First Victim

The first bug bounty program to get hit by the AI-security era was cURL. On January 26, the maintainer of the cURL bug bounty program published a blog post: [The end of cURL bug bounty][curl-end]. The primary reason for closure was simple: tons of crap that looked like valid submissions were being filed to the program. Due to the overload, they decided to close their bug bounty program.

The community took this as a negative thing since the slops weren't adding any value to the bug bounty program.

## The Series of Closure Continues

What looked like an isolated incident quickly turned into a pattern. Fast forward three months. On April 2, NodeJS pushed out a blog post announcing the closure of its bug bounty program: [Security Bug Bounty Program Paused Due to Loss of Funding][nodejs-close]. But it wasn't just NodeJS. It was the program that funded NodeJS BBP.

[Internet Bug Bounty][ibb], one of the most interesting bug bounty programs, funded by HackerOne, was paused. The scope covered some of the most used software in the world such as NodeJS, Ruby, Rust, OpenSSL, Nginx, Django, Apache Tomcat, and so on. The program had **paid out over $1.5M in bounties**.

Unlike cURL, the reason for closure was different. HackerOne mentioned: "The discovery landscape is changing. **AI-assisted research is expanding vulnerability discovery across the ecosystem, increasing both coverage and speed.** We have a responsibility to the community to ensure this program effectively accomplishes its ambitious dual purpose: discovery and remediation."

No mention of slops anywhere. They acknowledged that new vulnerabilities are being found at a rapid pace that HackerOne and the open-source maintainers can't keep up with. Slop also played a huge role which's obvious.

At the time of writing this article, one more open-source bug bounty program closed down: [Nextcloud][nextcloud]. The reason was "so many slops that it's difficult for us to identify high-quality reports".

## Case Study: Patchstack Bug Bounty

Let's set aside the speculations and public reasonings for a second and take a look at a place I have full visibility over.

If you don't know about Patchstack, it's the biggest vulnerability processor in the WordPres ecosystem.

The things have started to look very different since last December. The first three months of 2026 were us just working through the backlog of thousands of reports submitted to our program. At one point, we were a full month behind due to the sheer volume of submissions.

Even though Patchstack had been processing thousands of CVEs per year, the volume of submissions we started receiving since last December was too much to handle.

### Patchstack Bug Bounty Stats

| Month | Accepted | Rejected | Duplicate | Total | Acceptance Rate |
| --- | --- | --- | --- | --- | --- |
| November 2025 | 748 | 165 | 211 | 1,124 | 66.5% |
| December 2025 | 618 | 315 | 172 | 1,105 | 55.9% |
| January 2026 | 699 | 724 | 240 | 1,663 | 41% |
| February 2026 | 772 | 1145 | 224 | 2,141 | 35.98% |
| March 2026 | 251 | 1145 | 429 | 1,825 | 13.5% |

A few things to keep in mind before digging into the data:

- In November, we had a promotional event along with the monthly competition.
- From December, there were no events or special promotions.
- From January, we started tightening the scope every month forward.

Here's what the data says:

- **December** - Acceptance rate dropped 10% while total submissions held steady.  
  *Signal: AI-generated noise starting to enter the pipeline.*

- **January** - 500 more reports despite a tighter scope; valid count slightly up but acceptance rate fell ~15%.  
  *Signal: Volume shock begins with a surge in slops.*

- **February** - All-time high submissions. More submissions than the November promo, with tighter scope.  
  *Signal: Opus 4.6 was released to the public. AI tools are now genuinely finding real bugs, not just flooding with slop.*

- **March** - Accepted reports collapsed. Duplicates spiked sharply.  
  *Signal: Super-tightened scope. LLMs found the same bugs; now's the time for dupe fest.*

To fight against the volume, we HAD to stop accepting low-impact vulnerabilities. **Not just because of slops**, but also because there were just too many of those valid vulnerabilities.

We did not want to drown ourselves and the open-source maintainers with the alert fatigue.

We slowly shifted towards accepting high-impact vulnerabilities only. Each month, we kept increasing the minimum bar for acceptance and now the triage is under control. Only because we don't have to spend time validating low impact issues along with their potential slop variants anymore.

## Is It All Slop?

By now, we must be clear that it's not. Noise has always been a problem in bug bounties. Scanner dumps, template reports, out-of-scope submissions are not a new thing.

What's different now is that AI-generated noise looks exactly the same as a high-quality report. Same structure, same technical language, code traces showing the full flow of the vulnerability. Triagers have no way to distinguish except spend time working on it.

Somewhere in between those slops, there are truly high-quality vulnerabilities being found and reported by researchers too.

Complex SQLi in very popular plugins that have been tested over and over again, chains of multiple vulnerabilities to achieve maximum impact and other such cases are being seen more often than before.

## The Economy of Bug Bounty

Up until this point, bug bounty hunters were rewarded for their time and expertise to uncover the security issues the company couldn't find internally. Bug bounty literally exists to cover the gap of time and attention from a limited pool of experts.

However, for AI-generated **valid reports** in open-source programs, the financial incentive is no more there. The majority of AI-generated findings that bug hunters are reporting can be found by the internal team as well using the same tools.

After all, it's not only about the slops. For an average open-source bug bounty program, who would want to pay a bounty for someone asking Claude to find bugs when they themselves can do it?

However, this only applies to open-source. Billion-dollar companies can't get away with security by just pointing Claude at their entire infrastructure and call it a day, at least for now.

Let's not forget that running Claude Code on an open-source repo is wayyy easier and token-efficient with potential results than asking Claude to blindly poke at a subdomain.

## Are We Cooked?

The pre-2026 era of bug bounty for open-source projects seems to be in a very bad position right now. Even if companies find all the valid bugs using the frontier models and only truly creative findings remain hidden, the slop factor will still be there.

Each time a newer and more advanced LLM drops, the race to find that new bug begins. At this point, it looks like there is no point in having a BBP for open-source unless they are explicitly accepting high-value vulnerabilities only that could have direct impact on end-users or the service.

This is going to be the case until LLMs plateau, if they do. Who knows if it gets to a point when it starts finding new vuln classes on its own. However, once it does plateau, the cost of finding vulns will get back to what it was. BUT, with the bar being set too high. In terms of bug bounty, things will look very scarce.

## Ending Thoughts

Honestly, this is scary and exciting at the same time. The times have changed and there is no going back. Whatever the future might look like, one thing is clear: "Either adapt or die"

One thing to keep in mind is that technical skills are still equally needed. Otherwise, you'd just be one of those people contributing to the slops.

See you on the next one!

[curl-end]: https://daniel.haxx.se/blog/2026/01/26/the-end-of-the-curl-bug-bounty/
[nodejs-close]: https://nodejs.org/en/blog/announcements/discontinuing-security-bug-bounties
[ibb]: https://hackerone.com/ibb
[nextcloud]: https://hackerone.com/nextcloud/
