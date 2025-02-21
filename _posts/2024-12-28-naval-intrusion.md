---
title: Naval Intrusion
date: 2024-12-28 00:00:01 +0800
categories: [Hacktoria, Contracts]
tags: [osint]     # TAG names should always be lowercase
image:
    path: /assets/hacktoria/naval_intrusion/Badge-Naval-Intrusion.png
---

# Briefing

```
Greetings, Special Agent.

We have a special case on our hands today. As tensions around the island nation of Taiwan rise by the day, allied nations reach out frequently for our help. Today is no exception. This morning at 09:43 Taipei Standard Time, the Taiwanese Navy identified a foreign vessel entering it’s waters.

Upon making contact, the ship left westward for international waters. However, a UAV was dispatched and managed to snatch a picture of the vessel. The ship did not identify, nor does it match the database of known vessels operated by any Navy around the world. It comes close to several, but appears to be altered for a different purpose.

The Taiwanese government signed a contract for our services to identify the original make and model of the vessel.

As always, Special Agent. The Contract is yours, if you choose to accept.
```

![naval_intrusion ](/assets/hacktoria/naval_intrusion/materials-naval-intrusion.png)


# Answer Instruction

Use the answer to unlock the flagfile, this will reward you with your badge.

Answer Format: type-123q-nato-reporting-name

Answer Sample: type-555d-kingkong-IV


# Steps

- Doing some google lens search with the image provided i discovered an image similar to the one provided

![naval_intrusion ](/assets/hacktoria/naval_intrusion/Pasted%20image%2020241227005734.png)

- Going further to analyze the image discovered i found the name of the shape `Yueyang (FF 575)`

![naval_intrusion ](/assets/hacktoria/naval_intrusion/Pasted%20image%2020241227005857.png)

- So i researched more about the ship and found the class and type it belongs to which is `type 054A frigate`

![naval_intrusion ](/assets/hacktoria/naval_intrusion/Pasted%20image%2020241227010059.png)

- We have the name of the ship and it's type, now we have to find it's nato reporting name
- Doing some research on what nato reporting name is i found this

```
NATO reporting names are assigned by NATO to military equipment of countries that are not NATO members. These names are used by NATO intelligence and military personnel to identify and classify foreign military equipment.
```

and 

```
The NATO reporting name is based on the ship's class and type, rather than its individual name. This allows NATO to easily identify and track similar ships, regardless of their specific names or hull numbers.
```

Since the reporting name is based on class and type i just google the nato reporting name of the class our ship Yueyang FF 575 belongs to and i found it `Jiangkai II`

![naval_intrusion ](/assets/hacktoria/naval_intrusion/Pasted%20image%2020241227011135.png)

