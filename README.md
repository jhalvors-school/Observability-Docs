# 1. Observability Basics
Suppose we have network of computers. On each computer, there is some sort of security monitoring software, designed to find traces of cyber attacks. For instance, one machine might be logging process creation events. Others might be looking for behavior on the filesystem. Some may be monitoring the entire network, rather than the behavior of a single machine.

With many ways to monitor the behavior of software, a question arises of what monitoring is helpful, and what monitoring is a hindrance. Suppose we ran every monitoring tool ever developed on every machine. This would likely lag the machines, and in any case, generate so much data that processing it to find any attacks would be impractical. It could take very long, and there could be many false positives. An apt metaphor for this scenario is the classic "needle in a haystack" problem, and our solution is to remove as much hay as possible. Our other extreme, however, is to log nothing, in which case our "needle" becomes invisible. It's still there, and it can still hurt you, but now you don't know where it is.

Observability is a metric for a network of computers and an associated set of monitoring tools on that network, to be evaluated on how well attacks can be observed were they to occur within that system. For our needle in a haystack metaphor, this would mean we're measuring how easy it is to find needles.

# 2. TOMATO Framework
As one may imagine the above section, the definition of observability seems a bit vague. So how do we measure it, and what are its units? Well... this is where I plug a paper I published in 2019 with fellow researchers from WSU. This paper is available from IEEE Access (an open access journal) from the following link:

https://ieeexplore.ieee.org/abstract/document/8788508

This paper is effectively one of the first attempts at creating an observability metric. It draws upon an existing framework called ATT&CK, which is a classification system of "tactics" and "techniques" that are used by attackers to gain access to and compromise a system. A good resource for learning this framework can be found at:

https://attack.mitre.org/

## 2.1 Description of Observability Score

Observability in this paper is a 3-dimensional matrix. Every pair of machines has a score for each tactic to be observed occuring between those machines. If we have 4 machines on a network, and 3 tactics to observe (we'll call these "lateral movement", "execution", and "discovery"), this creates a 4x4x3 matrix, or 48 separate values representing observability. A score at index (1,2,3) in this matrix might be our ability to observe a discovery-type attack launched from machine 1 at machine 2. If the two machines in this index are the same, we are measuring observability of on-host attacks.

Of note, the observability score employed here does not have any proper units. Rather, we can tell that a score of 0 means an attack could never be observed, and anything else is a relative score. A value of 600 is clearly better than 500, but as for what such an absolute value means only makes sense within the context of an instance of an experiment.

## 2.2 Computing Observability in TOMATO

As for how observability is computed, this is done with two matrices:
* `F_tactic`, which represents how often a particular tactic is conducted on some pair of hosts, in a simulated environment. In our experiment, we conduct 1000 simulated attacks, making values in this matrix be between 0 and 1000.
* `P_cpd`, which is a conditional probability distribution for tactics occurring within the dataset. **This can be very confusing to understand, so read carefully**. The value of `P_cpd` for some pair of machines (i,j) on some tactic f, is 0 if no data exists at all to measure that tactic on those machines, and otherwise 1 - p, where p is the probability of a false positive occurring. This probability, p, is determined by finding the proportion of logs in our dataset which contain the features of that tactic.

From these two matrices, we use a hadamard product to produce the observability matrix. And that's it, that's your score.

Except... that definition of `P_cpd` is probably still confusing. What is meant by "contain the features of that tactic". This is where I will direct the reader to tables 9 through 11 of my paper and ashamedly tell you that this is where **a decent amount of manual intellectual labor was performed**. Here, I went through each of the tactics on the Mitre ATT&CK website that I wanted to observe, and for each one, read the definition of each technique, and defined some observable that could be found using my monitoring techniques. For instance, I knew that the "Remote Desktop Protocol" technique involved communicating over port 3389, and that Netflow (one of the monitoring techniques I was using) could report on ports used, so I would consider that anywhere that I saw port 3389 in my logs, this would be a false positive for the Remote Desktop Protocol technique, which is an instance of the Lateral Movement tactic. **Porting this project to another environment will likely involve re-designing these tables to be tailored to what you are measuring**.

By contrast, `F_tactic` is just a random walk on some graph that has the same topology as our network, where we assume our attacks is performing attack tactics with relatively equal probability. 

## 2.3 Running The TOMATO Software
From the IEEE Access link earlier in this section, there is an area at the bottom of the page titled "Code & Datasets". Clicking on it will give access to a Code Ocean repository where the original program can be run on the original dataset. This repository is effectively frozen for modification (Code Ocean is for providing reproducible runs of programs), so modifying this codebase would require [Opening the capsule on Code Ocean](https://codeocean.com/capsule/7979393/tree/v1), and using one of the options in the Capsule menu (export or clone via git) to download it.

You will find this program is written in Ruby, with the source code being located in the `code/` directory, and the datasets being located in the `data/` directory. Notably, this program was not originally designed to be extensible. It uses a very procedural design, and the codebase handles both preprocessing the dataset, as well evaluating the observability metric. Accordingly, changing the dataset would require significant changes to the codebase itself.

A colleague of mine has created a rewrite of this project in Python which may be easier to work with. It can be accessed at the following repository:

https://github.com/TorNATO-PRO/TOMATO
