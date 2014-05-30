
# Introduction

Peer reviewing is a central process in the organisation of scientific conferences. It involves multiple interactions between it's participants: authors share their work with the program committee which is then in charge of reviewing these submissions. A program chair is designated to coordinate the process and decide on final submission acceptance. While this peer review process could be implemented by simply exchanging emails between participants, 
manually gathering submissions, assigning them to \pcms, aggregating the reviews and finally notifying the authors requires substantial efforts. The use of a dedicated software, called \cms, can greatly simplifies this process.

Over the last few years, numerous web based \cms have been developed. A recent comparative study @parra2013 shows that Easychair @easychair is by large the most popular platform, having been used in about 68% of conferences organized with online \cms. The following 4 systems in term of number of organized conferences are EDAS @edas with 8.5%, Open Conference Systems @ocs with 6%, START V2 @startv2 with 5.7% and ConfTool @conftool with 5.3%. However, out of these 5 systems, only Open Conference Systems is open-source, restricted versions of Easychair and ConfTool are available for free while START V2 and EDAS are only available as commercial product.

  - Why a new system? Sending passwords by email, hard to modify and maintain

We present SlickChair, an open-source \cms written in Scala. Build with the Play framework and the Slick database access library, SlickChair provides a highly flexible and extensible solution to manage peer review processes. Our contributions are in particular:

  - The plan

 
# Overview of SlickChair

This section presents .. from the pov of different actors of the system: authors pcms and program chair. @mauro2005:

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


  Web-based assignment of papersWeb-based assignment of papers


  - authors
  - pcms
  - program chair

  - login
  - actors
  - phases (workflow)

Among his responsibilities, the assignment of submissions to \pcms can be a complex task. To be fair to all authors, submissions usually receive the same number of reviews, and this work has to be well distributed among \pcms so that no one overloaded. Moreover, \pcms might have conflicts of interests with certain submissions and different levels of knowledge depending on the topics. These constraints add up for

# Data model

@datomic

datastorage database
do not need transactions for most operations
still the go to product because of the abstraction layer, the backup capabilities and the power of the query engine.

  - functional, immutable database
  - database as a value, queries as a function
  - timestamp implementation
  - allows for concurrent read/write
  - single thread alternative for

# Evaluation/Why scala

Some subset of @ocs, @act, @symposion, @openconferenceware...

  - compilers, typechecking everywhere
  - server as a function
  - case classes all the way

# Future work:

  - Macros
  - Scala.js

# Conclusion

  - ...
