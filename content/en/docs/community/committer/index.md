---
title: "Committer Guide"
linkTitle: "Committer Guide"
weight: 4
date: 2021-08-12
description: >
  How to be a bRPC Committer
---
# Introduction

Participants in the Apache community have the following roles: **Contributor**, **Committer**, and **PMC(Project Member Committee) member**.

When an individual contribution is accepted by the project, he/she will automatically become a Contributor.
Committer and PMC members are invited by the PMC after a consensus vote.

Here we will only discuss some guidelines for the bRPC community to invite Committer and PMC member in order to be able to effectively estimate developer participation in the community.

### The Apache Way:
Before anyone can become a Committer or PMC member of an Apache project, they should first understand "[What's TheApacheWay](https://apache.org/theapacheway/index.html)".

### Guidance for Committer:
Have significant feature contributions (not limited to code), or long-term participation in community building (bug fixing, code review, documentation translation and proofreading, community outreach, etc.)
Participate in community discussions in public domain and have a positive impact.
### Guidance for PMC member:
Be able to actively participate in community maintenance work, such as answering emails, organizing wiki, release management, code review, etc.
Recognize the Apache community philosophy and be able to actively promote the community development.
### Peer Review:
The above requirements are highly subjective and cannot be measured quantitatively. Therefore, the PMC needs to form a regular review mechanism to discuss and invite people who meet the requirements.

Conduct a review every 1-2 months to nominate and discuss suitable candidates

---

# Detailed Procedures

## 1. How to develop committers

### Preconditions

1. More than 10 commits from the contributor
2. Contributor is willing to accept the invitation to become a committer
3. Contributor subscribe to dev@brpc.apache.org and introduce himself by sending an email to dev@apache.org

### The journey to become a committer

1. The nominator sends an email to private@brpc to initiate discussion and begins to voting. If the voting is passed, it is OK (at least 3 +1, +1 >- 1). (See the Vote email template https://community.apache.org/newcommitter.html#committer-vote-template)
2. The nominator sends a close vote email to private@brpc and private@incubator , the title can be subject [RESULT] [VOTE]. (See the close email template https://community.apache.org/newcommitter.html#close-vote)
3. The nominator sends an invitation letter to the nominee and prompts him to submit ICLA after receiving a reply. (See the template in https://community.apache.org/newcommitter.html#committer-invite-template)
4. The nominee fills in ICLA（ https://www.apache.org/licenses/contributor-agreements.html ）, individual contributors need to download [ICLA]（ https://www.apache.org/licenses/icla.pdf ）Fill in personal information and sign, and send the electronic version to secretary@apache.org 。 (Note: ICLA needs to fill in complete information, including mailing address and signature, otherwise it will be returned by ASF's secretary.) The personal information entry (except for the signature) can be filled in with a PDF reader or browser, and then saved for signature. Signature method support:
   * Print pdf documents and scan them into electronic version after handwritten signature;
   * Electronic signature using devices that support handwriting;
   * Use 'gpg' for electronic signature, that is, to operate the pdf file filled with personal basic information (a public key/key pair matching the registered mailbox needs to be generated in advance): 'gpg -- armor -- detach sign icla. pdf';
   * Use 'DocuSign' to sign;
5. The nominator sends an announcement email to dev@brpc.apache.org

### How to grant a committer permission on github

1. Add as committer ([https://whimsy.apache.org/roster/ppmc/brpc](https://whimsy.apache.org/roster/ppmc/brpc))
2. Let him set github id  ([https://id.apache.org/](https://id.apache.org/))
3. Let him visit the website and get github permission([https://gitbox.apache.org/setup/](https://id.apache.org/))

### Documents related to the new committer on the official Apache website

* [https://community.apache.org/newcommitter.html](https://community.apache.org/newcommitter.html)
* [https://infra.apache.org/new-committers-guide.html](https://infra.apache.org/new-committers-guide.html)
* [https://juejin.cn/post/6844903788982042632](https://juejin.cn/post/6844903788982042632)

### Suggested steps from secretary@apache.org

Please do these things:
1. Hold the discussion and vote on your private@ list. This avoids any issues related to personnel, which should remain private.
2. If the vote is successful, announce the result to the private@ list with a new email thread with subject [RESULT][VOTE]. This makes it easier for secretary to find the result of the vote in order to request the account at the time of the filing of the ICLA.
3. Only if the candidate accepts committership, announce the new committer on your dev@ list.

Doing these things will make everyone's job easier.

## 2. How to upgrade a committer into a PPMC

### Process reference: Apache official website document

* https://incubator.apache.org/guides/ppmc.html#voting_in_a_new_ppmc_member
* https://community.apache.org/newcommitter.html
* https://incubator.apache.org/guides/ppmc.html#podling_project_management_committee_ppmc

### Actual process

1. Initiate discussion in the private@brpc . If there is no objection, continue
2. Open a Vote in private@brpc 
3. If the vote is successful, sends a message to the PPMC private email list about the vote result
4. send a message to the IPMC (private@incubator.apache.org) with a reference to the vote result
5. Announce new PPMC On private@brpc 
6. Set his authority by visiting https://whimsy.apache.org/roster/ppmc/brpc
7. Help him subscribe to a private email group. See https://whimsy.apache.org/committers/moderationhelper.cgi
