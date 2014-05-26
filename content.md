
# Introduction

Peer reviewing is a central process in the organisation of scientific conferences. It involves multiple interactions between it's participants: authors share their work with the program committee, which is then in charge of the review process. A program chair is ordinarily designated to ensure good coordination and decide on final submission acceptance. While this process could be implemented by simply exchanging emails between participants, a lot of effort would have to be put gathering, coordinating and aggregating submissions and reviews. The use of dedicated software, called conference management system, can greatly simplifies this process.

  - Why a new systems (e.i. what's wrong with easychair)

  - Introducing SlickChair, plan/contributions
 
# Overview of SlickChair

  - login
  - actors
  - phases (workflow)

Among his responsibilities, the assignment of submissions to \pcms can be a complex task. To be fair to all authors, submissions usually receive the same number of reviews, and this work has to be well distributed among \pcms so that no one overloaded. Moreover, \pcms might have conflicts of interests with certain submissions and different levels of knowledge depending on the topics. These constraints add up for

# Play architecture/Why scala

  - compilers, typechecking everywhere
  - server as a function
  - case classes all the way

# Data model

datastorage database
do not need transactions for most operations
still the go to product because of the abstraction layer, the backup capabilities and the power of the query engine.

  - functional, immutable database
  - database as a value, queries as a function
  - timestamp implementation
  - allows for concurrent read/write
  - single thread alternative for

# Future work:

  - Macros
  - Scala.js

# Conclusion

  - ...
