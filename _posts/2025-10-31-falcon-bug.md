---
title: "ACM CCS 2025 Spotlight: How Bad Was The Falcon Bug of 2019?"
date: 2025-10-31
excerpt:  # shown on the Blog page; defaults to the first paragraph
tags: [PQC, Falcon, ACM CCS, Bug]
---

Last week, I had the honour and privilege of attending the ACM SIGSAC CCS conference in beautiful, yet at times blazingly hot, Taipei (Taiwan). The conference, which is widely regarded as a premier venue for cutting edge security and privacy research, took place over the span of 5 days (13 - 17 October) [1].

In addition to the main programme, which featured 316 (!) accepted papers, 21 co-located workshops took place that week. It is at one of those events, at QRSec, that I had the pleasure of listening to a talk by Mehdi Tibouchi (NTT Social Informatics Laboratories, Japan) on the mysterious 'Falcon Bug of 2019' [2].

More specifically, in August 2019, while the NIST PQC standardization process was happening in full force, the team of one of the candidates (Falcon) released a constant-time implementation of the scheme. Shortly after, bugs were discovered in the implementation of a key component (Gaussian sampler), resulting in an **incorrect distribution of the produced signatures that potentially leaked information related to the secret key**. The mistake was acknowledged by the implementer and a patch was quickly released. However, seemingly nobody has tried to answer the following questions: *How could such a bug come about and initially remain undetected? How problematic was this bug? Could the resulting leakage be used to mount a concrete key recovery attack?*

This blogpost will revisit this peculiar piece of PQC (standardization) history. We recall the events leading up to the infamous Falcon bug and the recent findings of Calvin Abou Haidar and Mehdi Tibouchi, who performed the (missing) full analysis of the buggy implementation and tried to develop a concrete attack against it.

# PQC Standardization

In December of 2016, the National Institute of Standards and Technology (NIST) published a call for proposals for their upcoming Post-Quantum Cryptography Standardization process [3]. 69 'complete and proper' candidates and 4 evaluation rounds later, NIST has recently started publishing their first PQC standards, most notably ML-KEM (formerly known as CRYSTALS-Kyber) and ML-DSA (CRYSTALS-Dilithium) [4].

However, if you take a close look at the selected PQC algorithms, you will see that the Falcon scheme [5] was also selected as a winner way back in July 2022, but that its standard is currently still pending [6].

![Figure 1:  NIST PQC Digital Signature Algorithm Standards. FIPS 204 (CRYSTALS-Dilithium) is published, Falcon has no FIPS yet. (Screenshot, [4])](/images/selected-algorithms.png)

We can assume that there are several factors contributing to this ‘delay’. For one, the US government is, as of writing this, still in shutdown with all ‘non-essential’ personnel having been temporarily furloughed. From looking at their webpage, it seems the NIST PQC team has been impacted.

![Figure 2: Due the US Government shutdown (October 2025), the NIST PQC Standardization effort seems to be paused. (Screenshot, [4])](/images/shutdown.png)

However, it is safe to assume that the leading cause for the delay in releasing the FN-DSA (FIPS 206) standard, has to do with the Falcon algorithm itself.

In July 2022, NIST clarified their choice to standardize Falcon in 'IR 8413: Status Report on the Third Round of the NIST Post-Quantum Cryptography Standardization Process' [7]. They state that “*... NIST has confidence in its [Falcon] security […] and because its small bandwidth may be necessary in certain applications*”. Indeed it is true that the combined public key + signature size of Falcon is the lowest among selected PQC standards. However, the scheme poses significant challenges on the implementation side.

# The Falcon Scheme

Falcon is a lattice-based digital signature algorithm, based on the theoretical framework of Gentry, Peikert and Vaikunanathan (GPV) [5] over NTRU lattices. It is a hash-and-sign signature scheme, which relies on a Gaussian sampling algorithm for sampling a short lattice vector close to the hash of a message that is being signed. In this case, the resulting signature is the difference between the hashed message and the nearby (sampled) lattice point. It can be formally proven that using these techniques in this framework, the produced signatures are **statistically independent of the secret key** and only dependent on the lattice itself. This was a key breakthrough for the construction of secure lattice-based signature schemes.

![Figure 3: the logo of the Falcon scheme, submitted to the initial NIST PQC standardization process. (Screenshot, [5])](/images/falcon-logo.png)

However, as a wise person once said, **cryptography is only as secure as its implementation**. In the case of Falcon, this is especially tricky: the Gaussian sampler is best implemented using floating-point arithmetic for achieving the best performance (read: no emulating floating-point operations). Additionally, NIST has explicitly acknowledged that implementing the scheme is highly complex [8], while explicitly stating that the security of Falcon heavily relies on the scheme (and its Gaussian sampler) being correctly implemented [7]. 

We know NIST has been actively working on the Falcon standard, or FN-DSA as it will be known once standardized. They have provided us with several minor updates, including on how they plan to tackle the complexities of floating-point algorithms/implementations. According to a recent post to the pqc-forum by Dustin Moody, member of the NIST PQC team, NIST has even reached an agreement with the Institute of Electrical and Electronics Engineers (IEEE) to reprint parts of their technical standard for floating-point arithmetic (IEEE 754) [9, 10] in the upcoming FIPS 206.

# The 2019 Falcon Bug

Going back to 2019, while the initial NIST PQC standardization effort was ongoing, it was announced that Falcon had proceeded to the second round of the ‘competition’ on January 30th [11]. This was a result of, among other reasons, no obvious flaws being found in the (mathematical) security analysis of Falcon, increasing trust within the community that the scheme is secure.   

During Round 2, the focus of the evaluation shifts more towards the implementation aspects of the candidates. For example, the community tries to compare the latency (clock cycles) or required stack memory size between candidates when implemented on modern or embedded CPUs. This is important to consider as the selected algorithms would be, apart from being secure, also very efficient to deploy in the real world.

A well-known principle for implementing crypto securely is constant-time code: it is required that the execution time of an algorithm is independent of the value of any secrets that are involved in the computation. Otherwise, a malicious actor could simply observe the time it takes to execute the algorithm and derive the secret key. This is not acceptable for cryptography that needs to be used in the real world.

On August 3rd 2019, Thomas Pornin (member of Falcon team) announced the release of a constant-time implementation of the Falcon scheme on the pqc-forum, demonstrating the feasibility of Falcon in a real-world scenario [12]. Shortly after, Markku-Juhani O. Saarinen found two bugs in the code and notified the Falcon team after doing further statistical analysis [13].

On September 18th 2019, Thomas Pornin acknowledged the bugs and published a fixed implementation. Further, the Falcon team also announced a more comprehensive framework for statistical tests on samplers would be developed, to avoid such mistakes in the future (which is great, in my opinion).

![Figure 4: Thomas Pornin on pqc-forum on September 18th 2019, acknowledging existing bugs in a prior constant-time implementation of Falcon and releasing a fix. (Screenshot, [13])](/images/pornin-forum.png)

The Falcon team simply states that “produced signatures were valid but leaked information on the private key” [13], without any further details or analysis. This is exactly what Calvin Abou Haidar and Mehdi Tibouchi have tried to do: perform the missing analysis and figure out if indeed a concrete key recovery was possible on this buggy implementation.

# What Went Wrong?

Introduced previously, the GPV framework has enabled the construction of provably secure lattice-based digital signature schemes, including Falcon. It achieves this by, surprise surprise, *harnessing the power of randomness*: lattice vectors/points are sampled from a spherical Gaussian distribution, which only depends on the lattice itself (and not any secret basis/key).

It turns out that the initial Round 2 implementation had two issues [14], leading to an incorrect distribution in the generated signatures: 
1. The implementation relied on sampling from the Gaussian distribution using a table, which contained wrong values,
1. and the sampler relies on rejection sampling, which was using a wrongly computed probability.

For the first bug, Calvin Abou Haidar and Mehdi Tibouchi discovered why the table contained wrong values. More specifically, it appears the table present in the implementation contains values of the targeted probability distribution (PD) instead of the reverse cumulative distribution (RCD), required for the correct computation. Interestingly, both values/tables are present in the Falcon spec, even appearing side-to-side… It is easy to understand how such a mishap could occur.

![Figure 5: Table 3.1 in the Falcon spec, containing the probability/cumulative and reverse cumulative distribution tables. (Screenshot, [15])](/images/table-falcon.png)

# To Recover a Key, or Not?

Now there just remains one question, can the resulting leakage in the signatures, produced by the buggy implementation, be successfully exploited?

Until recently, the best method for recovering a secret key in this scenario was proposed by Nguyen and Regev in 2006 [16]. The technique exploits a non-Gaussian distribution, in this case caused by the two bugs, to recover a good basis of the lattice (e.g. the secret key). It achieves this through rescaling the distribution, obtained from the collected signatures, so that it has identity covariance and finding a local maximum in the fourth moment of the distribution over the unit hypersphere.

However, in the case of the signatures produced by this Falcon implementation, performing this attack would require collecting billions of signatures and access to massive amounts of memory and compute infrastructure. So, while theoretically possible, this attack would probably not be able to practically break the Falcon implementation!

It turns out that the missing key to a practical attack was recently discovered. Earlier this year, CROWHAMMER by Calvin Abou Haidar, Quentin Payet and Mehdi Tibouchi was published at CRYPTO [17]. Using techniques presented in that work, in combination with the original Nguyen-Regev attack, are now shown to be an effective way to attack the buggy implementation.

On a very high level, the missing step is to not rescale the distribution obtained from the collected signatures so that it has identity covariance, but exploit the information contained in the covariance. More specifically, the eigenvectors of the sample covariance matrix can reveal information related to the secret basis. By using the eigendecomposition of the signature covariance matrix to approximate a low-dimensional subspace containing a close approximation of the secret basis, a successful key recovery can be performed.

Abou Haidar and Tibouchi show that using only 50 million signatures, a low-dimensional subspace can be estimated, on which the Nguyen-Regev optimization can be performed to successfully recover the secret key using a reasonable amount of compute and memory.

# Conclusion and Lessons Learnt

**Implementing cryptography efficiently and securely is hard**. Implementing Post-Quantum cryptography efficiently and securely is proving to be even more difficult. It is clear that we, as a community, have made significant progress in the past few years towards solving some of these challenges. However, we are not there yet and there is much work left to do.

In the words of Thomas Pornin [13]:

> “The fact that these bugs existed in the first place shows that the traditional development methodology (i.e. ‘being super careful’) has failed. I had previously predicted that lattice-based cryptography would entail many implementation issues because samplers are hard to test; at that time I did not envision to find myself at the short end of that prediction, but here we are.”

This specific Falcon bug is an (another) example of how mathematically sound cryptography can be broken if the implementation has (minor) mistakes. Increased and more rigorous (machine-assisted) testing of these implementations as well as the development of automated frameworks, as proposed by the Falcon team after this incident [14], are critical. The approach of the Falcon team should be commended and serve as a starting point for your own playbook: open-sourcing cryptographic implementations for public review, actively engaging with the community, requesting feedback, actively fixing/improving their code and investigating mistakes once detected. 

The road to widespread PQC deployment will be bumpy and many open challenges remain, as it has been proven that mistakes could lead to serious consequences. But, as the saying goes: more challenges == more fun!

---

This concludes my ACM CCS ‘25 spotlight blogpost. I hope you enjoyed learning a bit about the difficulties of securely and efficiently implementing PQC! Finally, I’m already looking forward to CCS ‘26, which is set to take place a bit closer to home in Den Hague (Netherlands)!

P.S. Many thanks to Calvin Abou Haidar and Mehdi Tibouchi for the interesting work, please go and read their paper if you found this blogpost of interest and want more technical details. Also thank you to Jan-Pieter D'Anvers for proofreading this blogpost.

# References

[1] [https://www.sigsac.org/ccs/CCS2025/](https://www.sigsac.org/ccs/CCS2025/) (accessed on 20/10/2025)

[2] [https://sites.google.com/view/acmqrsec25](https://sites.google.com/view/acmqrsec25) (accessed on 21/10/2025)

[3] [https://csrc.nist.gov/CSRC/media/Projects/Post-Quantum-Cryptography/documents/call-for-proposals-final-dec-2016.pdf](https://csrc.nist.gov/CSRC/media/Projects/Post-Quantum-Cryptography/documents/call-for-proposals-final-dec-2016.pdf) (accessed on 21/10/2025)

[4] [https://csrc.nist.gov/Projects/post-quantum-cryptography/selected-algorithms](https://csrc.nist.gov/Projects/post-quantum-cryptography/selected-algorithms) (accessed on 21/10/2025)

[5] [https://falcon-sign.info/](https://falcon-sign.info/) (accessed on 21/10/2025)

[6] [https://csrc.nist.gov/News/2022/pqc-candidates-to-be-standardized-and-round-4](https://csrc.nist.gov/News/2022/pqc-candidates-to-be-standardized-and-round-4) (accessed on 21/10/2025)

[7] [https://csrc.nist.gov/pubs/ir/8413/upd1/final](https://csrc.nist.gov/pubs/ir/8413/upd1/final) (accessed on 21/10/2025)

[8] [https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/CsWPP35LJWw/m/ZAliOFiWBgAJ](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/CsWPP35LJWw/m/ZAliOFiWBgAJ) (accessed on 21/10/2025)

[9] [https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/CsWPP35LJWw/m/7pOF01tIAgAJ](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/CsWPP35LJWw/m/7pOF01tIAgAJ) (accessed on 21/10/2025)

[10] [https://standards.ieee.org/ieee/754/6210/](https://standards.ieee.org/ieee/754/6210/) (accessed on 21/10/2025)

[11] [https://csrc.nist.gov/News/2019/pqc-standardization-process-2nd-round-candidates](https://csrc.nist.gov/News/2019/pqc-standardization-process-2nd-round-candidates) (accessed on 21/10/2025)

[12] [https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/RuDwNXqO-20/m/kuDJQzoBCQAJ](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/RuDwNXqO-20/m/kuDJQzoBCQAJ) (accessed on 23/10/2025)

[13] [https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/7Z8x5AMXy8s/m/Spyv8VYoBQAJ](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/7Z8x5AMXy8s/m/Spyv8VYoBQAJ) (accessed on 23/10/2025)

[14] [https://falcon-sign.info/falcon-impl-20190918.pdf](https://falcon-sign.info/falcon-impl-20190918.pdf) (accessed on 23/10/2025)

[15] [https://falcon-sign.info/falcon.pdf](https://falcon-sign.info/falcon.pdf) (accessed on 23/10/2025)

[16] Phong Q. Nguyen and Oded Regev. 2006. Learning a Parallelepiped: Cryptanalysis of GGH and NTRU Signatures. In EUROCRYPT 2006 (LNCS, Vol. 4004), Serge Vaudenay (Ed.). Springer, Berlin, Heidelberg, 271–288. doi:10.1007/11761679_17

[17] Calvin Abou Haidar, Quentin Payet, and Mehdi Tibouchi. 2025. Crowhammer: Full Key Recovery Attack on Falcon with a Single Rowhammer Bit Flip. In CRYPTO 2025, Part V (LNCS, Vol. 16004), Seny Kamara and Yael Tauman Kalai (Eds.). Springer, Cham, 103–135. doi:10.1007/978-3-032-01901-1_4 