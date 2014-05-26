
# Introduction

Peer reviewing is a central process in the organisation of scientific conferences. It involves multiple interactions between it's participants: authors share their work with the program committee, which is then in charge of the review process, finally allowing the program chair to decide on submission acceptance. While this process could be implemented by the exchanging emails between participants, the use of dedicated IT system greatly simplifies the process.

Among the responsibilities of the program chair, the assignment of submissions to \pcms can prove to be a very complex task. To be fair among authors, submissions usually receive the same number of reviews, and this work has to be well distributed among \pcms so that no one overloaded. Moreover, \pcms might have conflicts of interests with certain submission and different levels of knowledge depending on the topics. Conference management systems can help a program chair for the coordination of the program committee, and offer a centralised system to upload submissions, reviews and discussions between program committee members.


  - Why a new systems (e.i. what's wrong with easychair)

  - Introducing SlickChair, plan/contributions
 
# Overview of SlickChair

  - login
  - actors
  - phases (workflow)

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
