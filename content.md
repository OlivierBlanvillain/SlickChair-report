# ABSTRACT {-}

Online \cmss have turned into an essential tool in the organisation of scientific conferences. Their main objective is to minimise the cost of operation and communication while maintaining the high quality of the peer review process. As the web becomes the primary platform to run software, users develop new expectations: one-click authentication, revision history and mobile device support; these are examples of features one expects to see in today's web applications. In this document, we report our experience building SlickChair: a modern \cms built with state-of-the-art web application technologies. We present the authentication mechanism of SlickChair, describe the structure of a conference workflow, introduce a new API to manipulate versioned data, and present our implementation of an automatic paper-reviewer assignment algorithm.


# Introduction

Peer reviewing is central in the process of organising scientific conferences. It involves multiple interactions between its participants: *authors* share their work with the *program committee* which is then in charge of reviewing these submissions. A *program chair* is designated to coordinate the process and decide on final submission acceptance. While this process could be implemented by exchanging emails, manually gathering submissions, assigning them to the program committee, aggregating the reviews and finally notifying the authors would require substantial efforts. The use of a dedicated software, called \cms, can greatly simplifies this process.

Over the last few years, numerous web \cms have been developed. A recent comparative study @parra2013 shows that Easychair @easychair is by large the most popular platform, having been used in about 68% of conferences organized with online \cms. The next four systems have a much smaller market share in term of number of conferences: EDAS @edas was estimated at 8.5%, Open Conference Systems @ocs at 6%, START V2 @startv2 at 5.7% and ConfTool @conftool at 5.3%. However, out of these five systems, only Open Conference Systems is open-source. Easychair and ConfTool offer free licenses for restricted versions and START V2 and EDAS are only commercial available.

The importance and confidentiality of the data manipulated by a \cms implies that security is a major concern. Closed source solutions are mainly available as hosted services, therefore requiring conference organisers to trust quality of the code, the robustness of the infrastructure and the respect of data privacy. The open-source solutions we considered where not providing satisfactory levels of security. For example, Open Conference Systems sends a copy of user passwords by email in plain text when the registration is completed. HotCRP @hotcrp has a similar flaw: it sends login links to users with passwords as part of the URLs. As none of the available solutions meet our requirements, we decided to build our own.

We present SlickChair, an open-source \cms written in Scala. Built with the Play framework and the Slick database access library, SlickChair provides a highly flexible and extensible solution to manage peer review processes. This report makes the following contributions:

- We briefly introduce SlickChair and present the functionalities that differentiate it from already established \cmss such as EasyChair. In particular, we discuss the ability to authenticate using a Facebook or Google account, and the high flexible of the workflow engine.

- We present a new API to manipulate versioned data on an append only database. Built on top of the Slick, our API combine the power of Scala type-checking with the benefits of using an immutable database.

- We show how we implemented SlickChair's automatic paper-reviewer assignment algorithm. Unfortunately, accounting for both \pcm preferences and conflicts of interest leads to an NP-Complete problem. While aiming for the optimal assignment is not possible, we opted for a very simple heuristic algorithm implemented with the help of the OscaR operational research library. 


# Overview of SlickChair

We begin our discussion with a short overview of SlickChair. We first present the three supported authentication mechanisms, Facebook, Google and email address, followed by an outline of the different user interfaces. The section concludes with a description of the default peer-review workflow and an example of customisation.

### User authentication

The first contact between a system and its users is often via an authentication interface. Most \cmss require users to create new accounts, which implies filling a form, receiving a validation email and memorizing a new password. Recent web browser implement mechanisms to reduced the pain of filling form and memorizing passwords, but these solutions are limited to the use of a single computer.

In SlickChair, we address this problem by supporting three types of authentication mechanisms:

- Authentication via Facebook account

- Authentication via Google account

- Authentication via email address

The implementation of Facebook and Google login uses the OAuth 2.0 protocol @oauth2, and is provided by the SecureSocial authentication module for the Play framework @securesocial.

In addition to associating each visitor to a unique identity, authentication allows to get a valid email address contact users. When users login with Facebook or Google account, we trust the email address returned by the OAuth protocol, given users of these networks are required to confirm their email address. In the case of login via email, the process is obviously more complex. After providing his email address, a new user has to open his email client and follow a validation link. From here, he is asked his first name, last name, and a new password (of minimum 8 characters) before completing the registration. Passwords are hashed using the bcrypt algorithm and stored in the database. SlickChair also offers options to change password or recover an account in case of forgotten password. Although authentication via email address adds complexity to the system, it is useful to users that use different email accounts for their personal and professional life.

In SlickChair, we identify each user by a single email address. Some other \cmss provide the ability to link multiple email addresses to a single identity and to multiple merge accounts into one. We believe that this functionality can sometimes be confusing, and adds unnecessary complexity to the system. As a side note, this design makes it possible for a user to close the Facebook or Google account he used to login, and claim his identity by going through the process of logging in using the corresponding email address.

### User interfaces

[^essential]: The authors of @mauro2005 identified nine *typical functionalities* of \cmss which roughly correspond to the functionalities provided by SlickChair interfaces.

Building SlickChair, our focus was on creating a simple and extensible system rather than offering a lot of configuration options. As a system becomes more complete, its complexity is likely to go up, and maintenance and extensions becomes harder. SlickChair provides the essential components[^essential] to run an online peer review process, listed below as interfaces available for each user role:

- Author:
  
    - Make new submissions

    - Edit submissions

    - See reviews

- \PCM:

    - Bid on submissions

    - Write reviews

    - Write comments

- Program Chair:

    - Assign submissions for review

    - Decide on submissions acceptance

    - Change user roles

    - Send emails and change conference phases

The *Change user roles* interface allows a \pc to designate certain users as co-chairs or members of the program committee. While it is also possible to define all \pcms and chairs in advance in a configuration file, using the *Change user roles* interface has the advantage to allow each user to chose his favourite login method. This accounts for the typical situation of users that forward messages sent to their professional email addresses to another service such as Gmail.

The *Send emails and change conference phases* interface is related to the conference workflow, discussed in the following subsection. *Assign submissions for review* is discussed in detail #paper-reviewer-assignment. Other interfaces are straightforward and wont be described for brevity. 

### Workflow

In order to coordinate the overall conference peer review process, we organised it as a chronological sequence on phases. Each phase has a set of interfaces enabled during the time frame allocated to this phase. We identified the following seven phases that may correspond to the workflow of a small conference: *Setup, Submission, Bidding, Assignment, Review, Notification and Finished*.

In addition to the configuration of enabled interfaces, each phase is defined with a function to generate emails, and function to generate an optional warning.

    case class Phase(
      configuration: Configuration,
      emails: Database => List[Email],
      warning: Database => Option[String]
    )

SlickChair uses the `emails` function before transitioning to a given phase to help the \pc in the writing of notification emails, by suggesting recipients, subject and content of messages to be sent when transitioning to this phase. If a warning is returned by the second function, it will be displayed to the \pc before he confirms the change of phase.

Conferences of different sizes and topics might want to use different workflows. In the current stage of the project, such configuration has to be done at source code level. As an example, we will show the changes needed to add a new component to the conference workflow. Suppose that some \pcms are careless about their reviewing responsibilities and do not respect the delays set by the \pc. The \pc might want to send a reminder to the \pcms that have not yet completed their reviews. This could be implemented by adding a *Review Reminder*phase between *Review* and *Notification*.

    Phase(
      Configuration("Review Reminder",
        pcmemberCanReview=true,
        pcmemberCanComment=true,
        chairCanDecideOnAcceptance=true),
      { db => Email(
        lateReviewerEmails(db),
        "Reminder: review deadline",
        "Dear Program Committee Member, ...")},
      noWarning
    )

Where `noWarning` a dummy function that returns no warning, and `lateReviewerEmails` is a function returning the email addresses of all late reviewers. In order to obtain this information, we need to use three tables of the database: the `assignments` and `reviews` tables are relations between \pcms and submissions, and the `persons` table contains personal information of SlickChair users.

    def lateReviewerEmails(db: Database) = {
      val assignmentPairs = db.assignments
        .filter(_.value)
        .map(a => (a.personId, a.paperId))
      val reviewsPairs = db.reviews
        .map(r => (r.personId, r.paperId))
      (assignmentPairs diff reviewsPairs)
        .join(db.persons).on(_._1 is _.id)
        .map(_._2.email).list(db.s)
    }

This function illustrates the use of the Slick database query library. Aside from `.join().on()` and `.list(db.s)`, the code would be identical if manipulating sets from the Scala collections. In reality, the entire body of this function is compiled into a SQL query. The `.join().on()` construct allows to join two tables on certain columns. The `.list(db.s)` wraps up the query definition and returns the result of execution as a list of Scala objects. 


# Data model

An original requirement of SlickChair was the ability to log all the actions and events that occurred in the system. To do so, a traditional approach would be to use the logging functionalities built into the Play framework. Concretely, this would correspond to appending a timestamp and a message to a text file whenever an interesting event occurs. We took an alternative approach of memorizing every change at the level of the database.

The idea of append only databases is not new, but it is becoming more and more relevant as the prices of storage go down. The Datomic database @datomic, which recently entered the market, is built around this idea of never altering or deleting data. Instead, changes are made by adding new versions of the existing data or marking it as being invalid. Datomic is a proprietary software and does not fit well the open-source platform SlickChair is build upon. In order to suit our needs, we built a small layer on top of Slick to manipulate versioned data. The semantic of the API presented in this section are inspired by Datomic's Clojure API @datomicapi.

### Database as a value

One of the core concept of functional programming is the manipulation of immutable data. But how could a database, this shared and always mutating entity, be seen as immutable? We define a `Database` to be a view of the data at a particular time `date`.

    case class Database(
        val date: DateTime, val s: Session) {
      val persons = table[Person]
      val personRoles = table[PersonRole]
      val papers = table[Paper]
      val paperAuthors = table[PaperAuthor]
      val paperDecisions = table[PaperDecision]
      val comments = table[Comment]
      val reviews = table[Review]
      val files = table[File]
      val emails = table[Email]
      val bids = table[Bid]
      val assignments = table[Assignment]
      val configurations = table[Configuration]
    }

Manipulations of `Database` objects, such as the function define in #workflow, will see the world as it was at time `date`. Hence, calling this function multiple times with the same argument will always return the same result: the query is a pure function and the `Database` is a value.

A `Connection` allows to retrieve the  current value of the database, and insert new elements in the database. The `insert` method returns two `Database` objects, the value just before and just after the insertion.

    case class Connection(s: Session) {
      def currentDatabase(): Database
      def insert(x: List[Model[_]]): (Database,
                                      Database)
    }

The last element of our API is the `Model` trait, which enforces that elements stored in tables are case classes with a `metadata` field providing the appropriate information.

    case class Id[M](value: UUID)

    type Meta[M] = (Id[M], DateTime, String)

    trait Model[M] {
      this: M { def metadata: Meta[M] } =>
      val (id, updatedAt, updatedBy) = metadata
    }

This API takes full advantage of Scala and Slick type-checking capabilities. Thanks to the type parameter on `Id`, the compiler will detect errors such as misusing the `Id` of a person as the `Id` of a submission.

### Implementation

Under the hood, SlickChair creates multiple database records for a given *natural key*. When using the `Database` object, the user queries is composed with a temporal filter that extracts the most recent records that where created before the time `date`. This corresponds to the *Type 2 slowly changing dimension management methodology*, as described in @kimball2002.

This approach has the advantage of not requiring any update, new records are simply appended to tables and only become visible for the newly accessed `Database` values. It also minimises the number of tables, which is beneficial because tables are directly manipulated by the user via Slick's domain-specific language. We developed this API with H2 and PostgreSQL, but as the implementation uses the Slick generic `JdbcDriver`, it should be compatible with all database systems supported by Slick.

Finally, we should note that in its current stage, our *database as a value* API does not allow the expression of transactions. The timestamp based implementation is by nature atomic: the temporal filter used on every table ensures that the database will be viewed either before or after a group of insertions. However it is currently impossible to express a *transaction*, that is, to enforce that the database is not modified between the time its value is queried and the insertions are effective. Fortunately SlickChair does not require transactions.


# Paper-reviewer assignment

One of the main responsibility of the \pc is the assignment of submissions to \pcms for reviews. In order to provide the best quality reviews, the \pc typically asks \pcms to bid on submissions, and uses this information to guide the assignment. While this task could reasonably be done manually for a small number of submissions, it quickly becomes difficult as the number of submissions goes up. In fact, the mathematical formulation is known to be NP-Complete. We will see how we formulated this problem using the OscaR operational research library, which provides different search patters to explore the space of all possible assignments under limited time constraints.

### Mathematical formulation

This problem can be seen as a generalization of the stable marriage problem. On one side, the reviewers emitted preferences for the submissions in the form of bids, called the *interest preference*. On the other side, authors want their submissions to be reviewed by \pcms with topic knowledge, the *expertise preference*. The paper-reviewer-assignment problem is a generalization of the traditional stable marriage problem in the following ways:

- *Polyamory*: submissions and reviews are matched many-to-many instead of one-to-one.

- *Indifference*: *interest* and *expertise preferences* contain ties instead of being total ordered. 

- *Incomplete lists*: conflict of interest forbid certain assignment, removing elements from the candidate lists.

While each individual variant can be solved in polynomial time, the combination of the three generalizations is NP-complete @goldsmith2007. An approximation algorithm to maximize fairness among the reviewers is discussed in @garg2010, but it revolves around repetitively solving linear programmes, which is not a practical solution.

### OscaR formulation

OscaR is a Scala library for Operations Research @oscar. Amongst other techniques, it allows to formulate problems using constraint programming. Given a matrix of `preferences`, a list of `conflicts`, and four integers `nPapers`, `nReviewers`, `nReviewPerPaper` and `nReviewsPerReviewer`, the following program searches for an assignment maximizing the sum of reviewer satisfaction:

    val m = makeMatrix(nReviewers, nPapers, 0 to 1)

    0 until nPapers foreach { j =>
      add(sum(m col j) == nReviewPerPaper) }
    0 until nReviewers foreach { i =>
      add(sum(m row i) == nReviewsPerReviewer) }
    conflicts foreach {
      add(m(_, _) == 0) }

    maximize(weightedSum(preferences, m)) search {
      binaryMaxDegree(m.flatten.toSeq) }
    onSolution { return m } start()

In this code, the `add` function allows to add constraints to the definition of the problem. It is used to enforce that all submissions and reviews have the same number of reviews, and no conflicting assignment are made. The `maximize()` `search{}` and `onSolution{} start()` constructs are used to define the objective function, the search heuristic and start the resolution. Before getting to this stage, some preprocessing is required to aggregate and prepare the data, which notably includes the addition of dummy submissions so that repartition of reviews is perfectly balanced among reviewers.


# Conclusion and future work

In this report, we introduced the SlickChair \cms. By going through some of the internals of the system, we highlighted his main strengths compared to the existing solutions. In particular, we also presented our design for the manipulation versioned data using an immutable database. While a \cms could be labelled as rather simple IT system, we believe that the experience we gained building SlickChair can be beneficial to the future design of comparable applications.

Future work could investigate the integration of external tools to simplify the tasks of authors and \pcms. For instance, a PDF metadata extraction tool could be used to pre-fill the new submission using the title, abstract and list of authors extracted from an uploaded PDF file (see @lipinski2013 for an evaluation of the freely available solution). To help in the detection of conflicts of interest, we could use a co-authorship database such as DBLP @dblp to automatically mark conflicts between \pcms and authors that have published together.

We are confident that the simplicity and flexibility of the modern tools and techniques SlickChair is built upon will catch the attention of the community and exert a pull on the conference organisers to consider SlickChair for their future conferences. Interested readers are invited to try out the demo and check out the project source code at <http://slickchair.org/>.
