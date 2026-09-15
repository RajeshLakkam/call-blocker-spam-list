---
layout: default
title: Privacy Policy
---

# Privacy Policy

- **Effective date:** 15 September 2026
- **Application:** Call Blocker (`com.rlakkam.callblocker`)
- **Developer:** Rajesh Lakkam
- **Contact:** [rajesh.lakkam327@gmail.com](mailto:rajesh.lakkam327@gmail.com)

Call Blocker rejects telemarketing, spam and promotional calls before the phone rings. This policy
describes exactly what the app does with data. It is written to match the app's source code, which
is the authority; where this document and the code disagree, the code is what runs on your phone.

## The short version

Call Blocker has no user accounts, no analytics, no advertising, no tracking and no server of its
own. Screening happens entirely on your device. The only two times anything leaves your phone are
a daily download of the shared spam list, and a spam report that **you** compose and send yourself.

## What the app stores on your device

All of the following stays in the app's private storage and is never uploaded:

- **Blocked-call history** — the caller's number, the reason it was blocked and the time. This is
  kept for **8 days** and then deleted automatically.
- **Your allow list and block list** — numbers you have added by hand.
- **Numbers you have reported** as spam.
- **Your settings** — whether screening is on, whitelist-only mode, pattern rules and so on.

This data is excluded from Android cloud backup and from device-to-device transfer, so it does not
leave your phone that way either. Uninstalling the app deletes all of it.

## Contacts

If you grant the optional Contacts permission, the app checks an incoming number against your
contacts so that people you know are never blocked.

The lookup uses Android's `PhoneLookup` provider and asks one question: *does any contact have this
number?* The answer is a yes or a no. **No contact is ever copied into the app's database, read into
a list, or transmitted anywhere.** The permission is optional and the app works without it — without
it, contacts simply are not treated as an allow reason.

## Network activity

**1. Daily spam-list sync.** The app downloads a public, community-maintained list of spam numbers
from:

`https://raw.githubusercontent.com/RajeshLakkam/call-blocker-spam-list/HEAD/spamlist.json`

This is a plain download. **No information about you, your phone, your calls or your contacts is
sent with it.** As with any web request, GitHub (the host) can see the IP address the request comes
from; that is GitHub's processing, governed by [GitHub's Privacy Statement](https://docs.github.com/en/site-policy/privacy-policies/github-privacy-statement),
and the developer neither receives nor stores it.

**2. Spam reports — only when you choose to send one.** If you report a number as spam, the app
opens **your own email app** with a message pre-filled and addressed to the developer. Nothing is
sent until you press send in your mail app, and you can read and edit the message or discard it
first. The app holds no mailbox credentials and cannot send mail on your behalf.

A report contains: the reported phone number, the name the caller claimed, any description you
typed, the category you picked, and the date. Because it arrives as ordinary email, the developer
also sees **your email address** and can reply to it.

Reports are used only to maintain the public spam list. A reported number may be added to that
public list, which is then distributed to all users of the app. **Your email address is never
published, shared or added to any mailing list**, and is used only to reply to you about the report.
Do not include anything in the description field that you do not want the developer to read.

## Donations

The app's About screen can open a UPI app so you can send a voluntary tip. This is entirely
optional, unlocks no features, and gives the app no access to your payment details — the payment
happens inside your own UPI app, and the developer sees only what any UPI recipient sees.

## Permissions, and why each is needed

| Permission | Why |
|---|---|
| **Contacts** (optional) | To avoid blocking people in your contacts. On-device only. |
| **Notifications** | To tell you a call was blocked. |
| **Internet** / **Network state** | To download the shared spam list. |

Call blocking itself needs no permission: Android binds the app's call-screening service directly.
The app does **not** request call-log access, SMS access, microphone, or location, and cannot read
your call history, your messages, or where you are.

## Children

Call Blocker is a general-purpose utility and is not directed at children under 13. The developer
does not knowingly collect personal information from children.

## Data retention and deletion

Blocked-call history is deleted automatically after 8 days. Everything else the app stores stays
until you remove it in the app or uninstall. To have a spam report you emailed removed from the
public list, or from the developer's mailbox, write to rajesh.lakkam327@gmail.com.

## Security

App data is held in Android's private per-app storage, which other apps cannot read. The spam-list
download is made over HTTPS.

## Changes to this policy

If this policy changes, the revised version will be published at this address with a new effective
date, and material changes will be noted in the app's release notes.

## Contact

Questions about this policy, or requests about your data: **rajesh.lakkam327@gmail.com**
