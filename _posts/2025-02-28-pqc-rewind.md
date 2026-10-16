---
title: "PQC Rewind (2024 Edition): Stay Agile, Stay Ahead!"
date: 2025-02-28
excerpt: "This blog post will cover some of the most interesting results and remarkable things to happen in the field of Post-Quantum Cryptography in 2024."  # shown on the Blog page; defaults to the first paragraph
tags: [PQC, Q-Day, Migration]
---

This blog post will cover some of the most interesting results and remarkable things to happen in the field of Post-Quantum Cryptography in 2024. It is by no means an exhaustive overview, but the list contains some of my personal favorites. The goal of this post is to pause for just a small moment and take a fun look back at the past year, share some cool anecdotes and maybe learn a few lessons to take into 2025. So grab a warm coffee, sit back and enjoy.

### February

## To hybrid, or not to hybrid?

BSI published their technical guideline TR-02102, containing updated recommendations for cryptographic algorithms and parameter sets. Two points stand out, compared to the recommendations of their American counterpart NIST.

First, it is clear BSI is taking a conservative approach to PQC standardization. For Key Encapsulation Mechanisms (KEM), they recommend FrodoKEM and Classic McEliece which are based on “more secure” and better studied hard problems. Remarkably, they announced they will only recommend the two highest security versions of NIST standard ML-KEM. The impact of different governmental standardization bodies selecting different (versions of) algorithms, remains to be seen. What it will not do, is cause a lack of content for future blog posts.

Second, and maybe more remarkable as their view differs from NIST, BSI also supports the use of hybrid key derivation with a classical and post-quantum protocol. Similar to ANSSI’s (France) view, as long as one is secure, the key derivation remains secure. And directly replacing classical algorithms with post-quantum alternatives is considered too risky, due to their immaturity. It will be interesting to see how industry adapts and which choices they make.

More: [Technical Guideline - Cryptographic mechanisms: Recommendations and key lengths](https://www.bsi.bund.de/EN/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/Technische-Richtlinien/TR-nach-Thema-sortiert/tr02102/tr02102_node.html). BSI. 02/02/2024.

## 1, 2, (PQ)3: iMessage gets a post-quantum upgrade.

Early in the year, Apple also made headlines by announcing their “groundbreaking post-quantum cryptographic protocol” PQ3, used in their end-to-end secure messaging (iMessage). The PQ3 protocol is a great example of post-quantum cryptography being  deployed at scale. PQ3 reaches Level 3 security, which might be a confusing label as the protocol relies on NIST Category V ML-KEM-1024. Instead, Level 3 clearly(…) refers to both PQC key establishment and (ongoing) PQC rekeying. For illustration, Signal’s PQXDH protocol is classified as Level 2 as it only contains quantum-secure key establishment.

PQ3 is also a demonstration of crypto-agility through hybrid key establishment. More specifically, their protocol relies on both ML-KEM-1024 and P-256 ECC. In other words, if one algorithm/key gets broken, the security is guaranteed through the other one. The construction allows swapping in any other scheme instead of one of the current two in the future. Such hybrid constructions are favoured by some governments more than others, but the consensus remains that they will only increase adoption speed and thus are net-positive.

Unfortunately, this is where the analysis of PQ3 ends, as the protocol is proprietary. It is clear that the choice of Apple to not make their protocol available for public scrutiny, is a big security risk in itself. The amount of eyes of security researchers on the protocol is far less than in an open-access scenario and has proven to be problematic in the past. So Apple, if you really want to claim Level 3, you know what to do!

More: [iMessage with PQ3: The new state of the art in quantum-secure messaging at scale](https://security.apple.com/blog/imessage-pq3/). Apple Security Blog. 21/02/2024.

### April

## 666.

We had to include the infamous IACR ePrint 555 paper. The author claimed to “show a polynomial time quantum algorithm for solving the learning with errors problem (LWE) with certain polynomial modulus-noise ratios”. As two of (currently) three NIST standards are based on LWE, this result was cause for significant worry for many people in the community. At least for a few days it got people scrambling, after which it became clear there was a mistake in one of the proofs. For now, LWE lives to fight another day. But I do miss the feeling of scrolling Twitter and seeing everyone’s reaction during those few days in April.

More: [Quantum Algorithms for Lattice Problems](eprint.iacr.org/2024/555.pdf). YiIei Chen. 10/04/2024.

### June

## ML-KEM keeps leaking.

Around the summer, another compiler-induced timing vulnerability was found in the ML-KEM reference implementation. In this case, Clang undid countermeasures in the poly_frommsg function (written in C), which produces a polynomial based on the bits of the message m. The compiler generates machine code which results in secret-dependent behaviour. 

In an excellently written blog post, the details of how this leads to a practically exploitable timing leak are explained. Luckily, the issue was quickly patched and impacted libraries were notified.

More importantly, this is not the first time such an issue has been found in the ML-KEM source code (e.g., KyberSlash). Probably, it will also not be the last. It is clear more research is needed towards techniques and methods for secure implementations of post-quantum cryptography.

More: [PQShield plugs timing leaks in Kyber/ML-KEM to improve PQC implementation maturity](https://pqshield.com/pqshield-plugs-timing-leaks-in-kyber-ml-kem-to-improve-pqc-implementation-maturity/). PQShield. 03/06/2024.

### October

## NIST announces Additional DS Round 2 candidates: let the games begin!

After a bit more than one year, NIST announced the 14 candidates advancing to the second round of the Additional Digital Signatures standardization effort. Originally, 40 teams submitted their proposal and were in contention (accepted submissions). The remaining schemes are mostly non-lattice-based, which is why the competition is so interesting.

Schemes based on more exotic mathematical hard problems will (most likely) get standardized at the end of the road. These include multivariate, isogeny-based and MPC-in-the-Head-based schemes, which have passed their first test. 2025 is certain to be as (if not more) interesting, as security researchers and cryptographers will take a more focused look at the selected schemes and determine which are most suitable and secure. Interesting times ahead!

More: [Status Report on the First Round of the Additional Digital Signature Schemes for the NIST Post-Quantum Cryptography Standardization Process](https://csrc.nist.gov/pubs/ir/8528/final). NIST. 24/10/2024.

### November

## Out with the old, in with the new.

Towards the end of the year, NIST came out with a clear statement regarding their recommendations for the upcoming PQC transition. In Internal Report 8547, they stated that 2035 is the year all classical public-key crypto (RSA & ECC) will be disallowed. This puts a big timer on industry, as it only leaves around 10 years. Remarkably, NIST recognizes that such a full transition will probably take upwards of 20 years (based on previous transitions). The topic of hybrid constructions remains tricky, but it looks like NIST will approve hybrid modes in which at least one algorithm is approved/standardized by them. For industry it will remain important to ensure their systems are agile, allowing them to replace their chosen algorithms with alternate ones. The ecosystem will rapidly evolve: algorithms might get broken and others might turn out to be more efficient. Anticipation is the key.

More: [Transition to Post-Quantum Cryptography Standards](https://csrc.nist.gov/pubs/ir/8547/ipd). NIST. 12/11/2024.

### December

## Willow

In a field which is steadily moving forward, Google announced their new breakthrough quantum chip: Willow. It “demonstrates error correction and performance, which paves the way to a useful, large-scale quantum computer”. For the post-quantum community, such announcements are a constant reminder that Q-day is coming: the day where a large-scale quantum computer is developed. We are at a pivotal moment in time where a lot of work is left to do to ensure continued security and privacy in the future. I wonder if any significant quantum computer progress or announcements will be made in 2025?!

More: [Meet Willow, our state-of-the-art quantum chip](https://blog.google/technology/research/google-willow-quantum-chip/). Google. 09/12/2024.

–

Thank you all for reading. To my colleagues who helped by proof reading and sharing comments: thank you!

Until (January** of) next year! ;)

