
# Introduction

Peer reviewing is a central process in the organisation of scientific conferences. It involves multiple interactions between it's participants: *authors* share their work with the *program committee* which is then in charge of reviewing these submissions. A *program chair* is designated to coordinate the process and decide on final submission acceptance. While this peer review process could be implemented by simply exchanging emails between participants, manually gathering submissions, assigning them to \pcms, aggregating the reviews and finally notifying the authors requires substantial efforts. The use of a dedicated software, called \cms, can greatly simplifies this process.

Over the last few years, numerous web based \cms have been developed. A recent comparative study @parra2013 shows that Easychair @easychair is by large the most popular platform, having been used in about 68% of conferences organized with online \cms. The following four systems in term of number of organized conferences are EDAS @edas with 8.5%, Open Conference Systems @ocs with 6%, START V2 @startv2 with 5.7% and ConfTool @conftool with 5.3%. However, out of these five systems, only Open Conference Systems is open-source. Easychair and ConfTool offer free licenses of their restricted versions and START V2 and EDAS are only available as a commercial product.

The importance and confidentiality of the data manipulated by a \cms imply that security a major concern when working with such system. Closed source solutions are often only available as hosted services, therefore requiring conference organisers to trust the company behind the product not only for the quality of the code, but also for the robustness of the infrastructure and the respect of data privacy. The open-source solutions we considered where not providing satisfactory levels of security. For example, Open Conference Systems sends a copy of the user passwords by email as plain text once the registration is completed. HotCRP @hotcrp, another open-source \cms, has a similar flaw: it sends login links to users with the password as part of the url. 

We present SlickChair, an open-source \cms written in Scala. Build with the Play framework and the Slick database access library, SlickChair provides a highly flexible and extensible solution to manage peer review processes. Our contributions are in particular:

  - The plan

 
# Overview of SlickChair

In this section gives an overview of functionalities of SlickChair. We first discuss how users.. (login, actors, phases (workflow))


### User authentication

The first contact between a system and its users is often via an authentication interface. Most \cmss require users to create new accounts, which usually implies filling a form, receiving a validation email and memorizing a new password. Recent web browser implement mechanisms to reduced the pain of filling form and memorizing passwords, but these solutions are usually limited to the use of a single computer.

In SlickChair, we address this common problem by supporting three types of authentication mechanisms:

- Authentication via Facebook account

- Authentication via Google account

- Authentication via email address

The implementation of Facebook and Google login uses the OAuth 2.0 protocol @oauth2, and is provided by the SecureSocial authentication module for Play Framework @securesocial.

In addition to associate each visitor with a unique identity, an objective of authentication is to get a valid email address to contact the user. This way, we avoid any chances of having typo in the email. When users login with Facebook or Google account, we trust the email address obtained via the OAuth protocol, given that both networks required their users to confirm their email address when they created their account. In the case of login via email, the process is obviously more complex. After providing his email address, a new user has to open his email client follow a validation link. From here, the user is asked his first name, last name, and a new password (of minimum 8 characters) before completing the registration. Passwords are hashed using the bcrypt algorithm @bcript and stored in the database. SlickChair also gives its users the usual options to change their password or recover their account in case of forgotten password. Although authentication via email address add complexity to the system, it is appreciated by users that have separated email accounts for their personal and professional life.

In SlickChair, we identify each user by a single email address. Some other \cmss provide the ability to link multiple email addresses to a single identity and to multiple merge accounts into one. We believe that such functionality can sometimes be confusing, and adds a lot of complexity to the system. As a side note, this design makes it possible for a user to close the Facebook or Google account he used to login, and still claim his identity by going thought the process of logging in using the corresponding email address.


### SlickChair interfaces

[^essential]: The authors of @mauro2005 identified nine *typical* functionalities offered by \cmss to run online peer-review processes, which roughly correspond to the functionalities provided by SlickChair interfaces.

Building SlickChair, our focused was on creating a flexible and extensible system rather than offering customization options via configuration. As a system becomes more complete, it's complexity is likely to go up, and maintaining and extending the system becomes harder. \TODO{Add ref to #evaluation.} SlickChair provides the essential[^essential] components to run an online peer-review process, listed below as the user interfaces of available for each user role:

- Author
  
    - Make new submissions

    - Edit submissions

    - See reviews

- \PCM

    - Bid on submissions

    - Write reviews

    - Write comments

- Program Chair

    - Assign submissions for review

    - Decide on submissions acceptance

    - Change user roles

    - Send emails and change conference phases

Most interfaces are very straightforward and will not be discussed here for brevity. The *change user roles* interface allows a program chair to design certain users as being co-chairs or \pcms. While it is also possible to define all \pcms and chairs in a configuration file before setting up a SlickChair instance, using this *change user roles* interface might be preferred to allow each user to chose his favourite login method. This accounts for the typical situation of users that forward messages send to their professional email addresses to other services such as Gmail.

The *Send emails and change conference phases* interface is related to the conference workflow, discussed in the following subsection.


### Workflow

code of Phase case class

Setup, Submission, Bidding, Assignment, Review, Notification, Done

- Submission of abstracts and papers by Authors
- Submission of reviews by the Program Committee Members (PCM)
- Download of papers by Program Committee (PC)
- Handling of reviewers preferences and bidding
- Web-based assignment of papers to PCMs for review
- Review progress tracking
- Web-based PC meeting
- Notification of acceptance/rejection
- Sending e-mails for notifications

1. Submission of papers
2. Assignment to reviewers
3. Mailings to PC members
4. Submissions of reviews
5. Reviewer comment threads
6. Distributed PC meeting
7. Letters to authors

http://www.texample.net/tikz/examples/simple-flow-chart/

texdoc pgfgantt

Among his responsibilities, the assignment of submissions to \pcms can be a complex task. To be fair to all authors, submissions usually receive the same number of reviews, and this work has to be well distributed among \pcms so that no one overloaded. Moreover, \pcms might have conflicts of interests with certain submissions and different levels of knowledge depending on the topics. These constraints add up for


# Paper-reviewer assignment

It's NP-Hard :(
@garg10
<http://www.cs.uky.edu/~goldsmit/papers/GodlsmithSloanPaperAssignment.pdf>

# Data model

- Logs where a requirement
- Inspired from Datomic @datomic data model
- Re-implemented some of Datomic's functionalities as a layer on top of Slick/AnySQL
- Functionality and implementations are detailed in this section.

datastorage database
do not need transactions for most operations
still the go to product because of the abstraction layer, the backup capabilities and the power of the query engine.

  - functional, immutable database
  - database as a value, queries as a function
  - timestamp implementation
  - allows for concurrent read/write
  - single thread alternative for

# Evaluation

Some subset of @ocs, @act, @symposion, @openconferenceware...

  - compilers, typechecking everywhere
  - server as a function
  - case classes all the way

# Future work:

  - Macros
  - Scala.js

# Conclusion

  - ...
