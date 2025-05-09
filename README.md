# testing-frameworks: a curated collection of software testing frameworks

# CONCEPTS

## Formal Verification

Formal verification is the gold standard of software testing. When designing data models and functions, consider how the algorithm could encounter various error states, bugs, spurious results, and so on. A program is also a proof, each variable a logical proposition with logical consequences.

```go
func Ratio(x int, y int) int {
    return x/y
}
```

The unprotected code snippet is vulnerable to division by zero, which triggers application crashes in the majority of programming languages.

```go
import (
    "errors"
)

func Ratio(x int, y int) (error, int) {
    if y == 0 {
        return errors.New("zero division")
    }

    return nil, x/y
}
```

The protected code snippet closes the gap, so that edge cases are handled in a predictable, documented way.

In this scenario, we use a runtime check to *validate* input/state space. However, for many edge cases, the data model itself provides stellar performance and reliability guarantees. For example, operations using signed integers are often vulnerable to surprising negative values, whereas unsigned integers categorically rule out the possibility of inputting negative values.

There's a longrunning joke that you can either write a million Ruby tests, or write just one line in Go/Rust/etc. To spoil the joke, static type systems rule out enormous collections of bugs in one fell swoop. For this reason, statically typed, compiling programming languages are preferable compared to interpreted languages. Interpreted languages fail quite late in the Software Development Life Cycle (SDLC) on hilariously simple problems, everything from function arity, to scalar data types, vs. vector data types, to misspellings in variable names. Just because everyone is jumping off of an ECMAScript bridge doesn't mean you have to follow.

In terms of safety in information technology systems, understanding edge cases is crucial to staving off computer glitches. Assume that a random cosmic ray flips bits en route to various code sections. How would your code react in such cases?

Another approach to formal verification is thinking like an adversarial attacker: If your mission were to exploit the current system, which edge cases are technically available to exploit?

1. Identify low hanging fruit with potential for malicious/accidental code misbehavior.
2. Apply type signatures, type contracts, and/or validation logic so that each possible input or program state has a well defined, predictable behavior.
3. GOTO 1.

For highly sensitive algorithms such as cryptography, engineers often use proof assistants to generate foolproof code. For example, SHA-256 may be written in Coq as the primary source of truth. Then, the proof assistant code is used to generate safe implementations downstream in conventional programming languages.

https://en.wikipedia.org/wiki/Proof_assistant

Although formal verification provides the strongest possible guarantee of software quality, there are several lightweight techniques that provide practical compromises between productivity and quality, detailed below.

Type contracts appear to have lost momentum over the past decades. If you are forced to use interpreted languages, consider contributing to type contract frameworks.

## Test Driven Development (TDD)

Test driven development is a tenet of modern software development. In short, test driven development is the discipline of scaling out the test suite as the number of code changes grows.

Projects lacking comprehensive tests, should introduce a growing test suite, in order to catch bugs before users do.

Warning: Do not inflict strict testing policies upon software projects. Demands for too many tests, too complex tests, and too much automation triggers a developer reflex where instead of actually testing program behavior, the developer ends up writing junk tests (NO-OPS).

```go
import (
    "testing"
)

func SystemTest(t *testing.T) {
    if 2 + 2 != 4 {
        t.Error(errors.New("logical fault"))
    }
}
```

This test always passes. It never analyzes any behavior of the library or application. The test provides *negative* quality, because it distracts from genuine efforts to develop good code.

## Fail Early and Loudly

The principle "Fail early and loudly" provides a strong foundation for safe, capable programs. In the event of a problem, trampoline back up the call stack, cancelling other suboperations, and let the user know enough about the nature of the error to facilitate troubleshooting.

Note that shell scripts, as opposed to general programming languages, do not terminate by default, for any error conditions. This leads to data corruption, missiles firing by mistake, etc. etc. When in doubt, enable more safety options available to your shell implementation.

## Fault-Tolerance

Fault-tolerance exhibits more nuanced behavior than basic failing early and often.

This includes failing gracefully, in some cases completing suboperations which do not corrupt the user's data, or treating minor problems as warnings rather than errors.

Another form of fault-tolerance involves retry logic. The supervisor pattern establishes a parent process which automatically restarts a child application process in the event of the application crashing. A Highly Available (HA) cluster of resources can often continue to provide service in the event that one or more replicas experiences problems.

Note that many retry behaviors should implement exponential backoff and retry, in order to free up valuable computing resources, for both servers and clients.

## Red Green Refactor

Bad tests are worse than no tests.

Red Green Refactor provides a rule of thumb for how to write great tests.

1. Red: Before writing library/application code, write the code that will test it. Ensure that the test currently registers a failing status, bonus points if the test suite refuses to compile.
2. Green: Implement the library/application code sufficient to change the test status from failing to passing. In version control terms, now is a good time to stage the code changes, for example in git.
3. Refactor: Polish the code. Re-run the test suite, ensuring that it passes again. Stage, commit, and push the code to a repository clone somewhere.

Omitting the first step results in bad tests, because unfortunately it is all too easy to accidentally write bad tests, and easy to accidentally write library/application code that is not amenable to testing by design. The Red step verifies the quality of the test suite itself. Developers do not begin their careers instantly knowing how to write good tests. But the Red step helps engineers at all levels of experience, to write better and better tests, and design software for maximum testability.

## Unit Tests

The humble unit test is the foundation of modern software testing. A unit test is designed to be exceptionally fast, providing developers with quick information about the presence or absence of bugs.

Software projects of every size, should include a unit test suite, with each unit test corresponding to a particular application behavior.

One fantastic strategy when designing unit test suites, is to establish at least one unit test per operation available to the user. This way, core application logic is guaranteed to have tests.

For projects that publish libraries / SDK's / API's / ABI's, this is especially helpful, as the unit tests double as executable documentation of the component's usage. Instead of too much dead documentation that risks getting out of date, unit tests are intended to keep continuously up to date throughout the lifecycle of the project. Unit tests are self validating with automated commands to check themselves, whereas static documentation is only as good as humans remembering to update them (they don't). Finally, unit tests are fully qualified, with essentially valid imports, constructors, and so on, in contrast with example code snippets that miss these vital usage details.

Unit tests can evaluate, for example, the behavior of API namespaces, functions, and data models.

A unit test suite often begins with a handful of fledgling, manual test cases converted into automated tests, using a modern unit test framework available for the project's primary programming language.

## Integration Tests

Engineers often use the ambiguous term "test" which can connote unit tests, integration tests, or a combination of both. When in doubt, request specifically *unit tests*.

Integration tests are distinct from unit tests. Whereas unit tests focus on evaluating code logic, whereas integration tests involve external resources. The need for real I/O, such as real file systems, databases, network connections, remote resources, and/or local devices, is a strong indication of an integration test.

For example, a test that connects to a real database would constitute an integration test, whereas a test that connects to a *mock* database would constitute a unit test.

A test that evaluates a CLI, GUI, Web, serverless function, etc. application, would constitute an integration test. A test of an underlying library that powers CLI, GUI, Web, serverless functino, etc. application, would constitute a unit test.

There are many pros and cons to unit tests vs. integration tests.

Integration tests are normally too slow for modern sofwtare development needs compared to unit tests. Some integration tests can identify subtle environment quirks that an API fails to clarify. However, unit tests overall provide the highest degree of both testing performance and software quality, due to the sheer number of tests that a unit test suite is able to execute in a short time frame.

## Performance Tests

Performance tests benchmark software for time sensitive requirements, such as latency and throughput. In computing, a slow operation can be just as bad as failing operation.

Many test frameworks include options to specify timeouts, to register a failure whenever the test is unable to terminate within a given time frame.

Software teams practicing Site Reliability Engineering are welcome to establish Service Level Agreements (SLA's), including specific Service Level Objectives (SLO's), such as "The bug reporting service should exhibit 2xx HTTP response status codes five nines of the year, or else we will prioritize maintenance tasks over feature work."

Stress tests work similarly to performance tests, but with a focus moreso on availability. Sometimes availability carries related SPI's. Sometimes stressing a system causes it to completely collapse. Bugs, or capacity issues in fundamental computing resources (CPU, RAM, disk, networking, databases, external services, etc.), can cause headaches when not updated to account for computer resource consumption trends.

Stress testing may simulate high or abnormally high service request traffic, in order to ensure that a service is prepared for customer activity spikes. For example, to prepare for the Digg going viral effect, or Black Friday events. Marketing likes to brag when high sales volumes crash our systems, but ideally we operate with automatic, elastically scaling resources.

Stress tests can help to reveal gaps in High Availability, where single points of failure trigger cascading problems.

Chaos Engineering takes stress testing one step further. Where stress testing ramps up client requests, chaos engineering simulates internal problems, such as a server becoming unavailable. Chaos engineering helps to evaluate the overall resilience of a system, identifying bottlenecks at multiple levels of abstraction. Gaps in High Availability create basic availability problems.

## API Publicity

Modern programming languages include a mechanism to distinguish between public API members vs. private API members. This tool is designed to steer API users toward the safest, most convenient interactions. A good way to increase quality, is to limit which elements of a computer system are included in the public API. Private members should still be tested, though the bulk of the risks arise from inputs and states in terms of the public members.

Warning: API publicity is not always foolproof. For programming library API's and ABI's, there are ways to coerce data into a corrupt state, bypassing API publicity rules and validation rules (in fact, that's a necessary feature of mocks). RAM can be struck by cosmic rays, flipping arbitrary bits around. Generally, re-apply validation logic at multiple levels of a system. Not only in frontend code, but in the backend service, and in client SDK's that connect to the backend service.

For remote API's such as REST API's, there are often too many ways to reference the same resources, enabling leaks and tricks when multiple remote calls are combined. Recall that the attack surface is the cumulative sum of all public API's. For this reason, it is vital to establish consistent, reusable data models and operations to replace snowflakes data models and operations.

## Mocks

Unit tests can effectively account for external resources by mocking them with in-memory equivalent implementations.

Many databases, clients, and file systems provide constructors for mock objects for this very purpose.

When premade mocks are unavailable, you can apply *spy* or *monitor* mock objects that wrap the underlying object. Spies and monitors maintain an audit trail of the various function arguments and return values, which can then feed into unit tests.

You can also write your own mocks: Place interfaces along the boundaries between code logic and external resources. Write one implementation of the interface for each supported real resource option (e.g. InfluxDB, Datadog, and generic OpenTelemetry options). Then, write a stub implementation that collects pertinent information useful for testing purposes.

## Code Coverage

Code coverage is a crude technique that considers what percentage of a codebase has at least one corresponding test reach each line of code.

When establishing code coverage policies, reduce expectations. A little code coverage goes a long way, but too high coverage perversely creates environments counterproductive to software development.

For example, code coverage in a project with exceptionally deep testing, is typically no higher than 80% covered on average. And each percentage point beyond that has diminishing returns, eventually incentivizing bad NO-OP tests.

Finally, do not prejudice old code vs. new code in terms of differential thresholds. Many organizations make the mistake of splitting a code coverage by the recency of the code change. As a consequence, old code gets a free pass to continue shoving bugs under the rug. And new code receives unrealistic expectations about coverage ratios. The threshold should be applied uniformly, in terms of the **total code** in the repository, as viewed commit by commit. Not in terms of the size of any particular code change.

## Test Data

Some tests involve complex objects, which require a slew of values to be populated to feed the test suite. Several approaches are available to obtain test data:

* Manually constructed test data
* Test fixtures
* Randomly generated test data

The first and simplest of these is manually constructed test data. Many test suites begin their life cycle with a handful of trivial examples, such as "Alice and Bob call each other. Ensure that the call terminates properly when either Alice or Bob hang up." Manual examples are fantastic for sketching out test suites, and double as usage examples to fill in many documentation gaps.

Next, text fixtures are auxilliary resources, such as files, Web assets, or database snapshots, that provision test data. Unfortunately, fixtures as a concept has a lot of problems of its own. Fixtures may not necessarily integrate well with text-based version control software. Fixtures often reside distant from the location of the corresponding unit test source code files, slowing navigation. Fixtures can take up a lot of space. Fixtures in the form of production snapshots, incur data leak risks (sanitization is not a panacea), and quickly become out of date. Fixtures give off a code smell that the API may not be designed for in-memory data, forcing wasteful I/O. Fixtures complicate test suites, compared to simply constructing the same data with ordinary code. JSON fixtures often break comments, a vital development tool. Fixtures give off a code smell of gaps in terms of database schemas. Small fixtures should be rewritten as in-memory data constructions. Intermediate fixtures should be synthesized by fuzzing. Large fixtures should be split apart into distinct tests with more focused checks.

Finally, randomly generated test data significantly raises the quality bar. This is also called *synthetic test data*. The related concept of fuzzing, makes it more convenient to manage randomly generated test data.

## Validation & Fuzzing

*Validation* applies type signatures, type contracts and/or runtime checks to safeguard operations against surprises in input/state space.

For example, Object Relational Mappers (ORM's) validate inputs in order to mitigate SQL injection attacks. Intentional nullable types use the absence of a pointer referent to clearly indicate the existence of errors. Intentional zero values in data models provide a meaningful data state with more defaults, as users tend to benefit from more zero conf than cumbersome boilerplate.

*Fuzzing* generates random test scenarios, evaluating logical properties of the code to identify latent edge cases.

Fuzzing can be done for all tests, however developers gain outsized value by targeting a few critical code sections thoroughly, rather than every code section scarcely. Start by fuzzing the most crucial code logic, and building our your fuzzing tests to eventually cover the entire API / attack surface.

The input/state space of a program is exponentially larger than the number of lines. So fuzzing finds more bugs than code coverage. Both are good to have, but if you want the most bang for your buck, prioritize fuzzing.

Fuzzing is often used to test for memory bugs, especially in programming languages that offer manual memory management.

## Programming Language Testing Tools

### *

[Cucumber](https://cucumber.io/) provides a flexible, and fairly portable Business Driven Development (BDD) framework for integration tests across very many programming languages and operating systems. Cucumber and its many implementations and plugins support integration tests for shell scripts, CLI tools, Web services, GUI applicatinons, etc. If you absolutely have to write an integration test as opposed to a unit test, then experiment with Cucumber.

[expect](https://core.tcl-lang.org/expect/index) provides an integration test framework for CLI tools, though I've personally never gotten expect to work.

[k6](https://k6.io/) provides performance testing for Web services.

[libFuzzer](https://llvm.org/docs/LibFuzzer.html) is a fuzzing framework with support for C and C++.

[make](https://en.wikipedia.org/wiki/Make_(software)) is a task runner / build system with support for a wide variety of programming languages and frameworks, especially when multiple programming language technology stacks are involved.

[QuickCheck](https://en.wikipedia.org/wiki/QuickCheck) is a fuzzing framework with support for a wide variety of programming languages.

[OSS-Fuzz](https://google.github.io/oss-fuzz/) supports fuzzing in C, C++, JVM/Java, Go, Python, and Rust projects.

[Valgrind](https://valgrind.org/) is a classic fuzzing tool that identifies bugs in various and sundry programming languages. Note that Valgrind itself contains memory bugs, and often crashes.

### C/C++

[cmake](https://cmake.org/) features task runners / build systems, package managers, and the `ctest` unit test tool for C/C++ projects.

[rez](https://github.com/mcandre/rez) wraps C/C++ build systems like cmake, enabling C/C++ projecst to implement build tasks in ordinary C/C++ code. When combined the C++'s standard library, the user gains a higher degree of cross-platform support for *building* the project, an oft-neglected aspect of software developments.

### D

[D](https://dlang.org/) supports writing [unit tests](https://tour.dlang.org/tour/en/gems/unittesting) beside the implementation code in the same source file, encouraging a high degree of TDD.

[Dale](https://github.com/mcandre/dale) is a make alternative enabling D projects to implement build tasks in ordinary D code.

### Erlang

[Erlang](https://www.erlang.org/) is a distributed-first functional, logic programming language designed for HA telecom systems. It is said to take a different approach to errors, emphasizing fault tolerance and zero downtime software upgrades over getting code perfect on the first try. The Erlang community seems to have invented the QuickCheck fuzzing framework.

[EUnit](https://www.erlang.org/doc/apps/eunit/chapter.html) is a conventional, base unit testing framework for Erlang projects.

### Go

[Go](https://go.dev/) includes built-in support for [unit tests](https://pkg.go.dev/testing) and [fuzzing](https://gist.github.com/mcandre/1ea8b4b534ed44e0e88c6f855f692d94).

[errcheck](https://github.com/kisielk/errcheck) is a notable linter that detects when Go code neglects to handle error return values, a significant cause of bugs in many applications.

[gucumber](https://github.com/gucumber/gucumber) provides an integration test framework for Go projects.

[mage](https://magefile.org/) is a make alternative enabling Go projects to implement build tasks in ordinary Go code.

### Haskell

(GHC) [Haskell](https://www.haskell.org/) is a functional programming language in the Meta Language (ML) lineage, with an exceptionally rich type system to ward off bugs.

Haskell supports a syntax variant called "Literal Haskell" which flips the normal comment vs non-comment source code behavior around. In Literal Haskell, developers are encouraged to describe algorithms in prose, using a special syntax to start and end code snippets. It's like API documentation but in reverse.

[HUnit](https://hackage.haskell.org/package/HUnit) is the conventional, base unit testing framework for Haskell projects.

[Shake](https://hackage.haskell.org/package/shake-cabal) is a make alternative enabling Haskell projects to implement build tasks in ordinary Haskell code.

### JVM

[cucumber-java](https://cucumber.io/docs/installation/java/) provides an integration test framework for JVM/Java projects.

[Gradle](https://gradle.org/) is the preiminent build system for most JVM programming languages.

[JUnit](https://junit.org/junit5/) is the industry standard base framework for writing tests in various JVM languages, including Java.

[Mockito](https://site.mockito.org/) is a prominent framework for managing mocks in JVM/Java unit tests.

[PowerMock](https://github.com/powermock/powermock) helps JVM/Java developers to test objects involving difficult-to-test publicity designs.

### Python

[Python](https://www.python.org/) is a dynamic, interpreted programming language with a built-in [unit test](https://docs.python.org/3/library/unittest.html) framework.

[invoke](https://pypi.org/project/invoke/) is a task runner enabling Python projects to implement build tasks in Python code.

### Ruby

[Ruby](https://www.ruby-lang.org/en/) is a dynamic, interpreted programming language.

[RSpec](https://rspec.info/) is the preeminent, base unit testing framework for Ruby projects.

### Rust

[Rust](https://www.rust-lang.org/) includes built-in support for [unit tests](https://doc.rust-lang.org/cargo/commands/cargo-test.html) and [fuzzing](https://rust-fuzz.github.io/book/cargo-fuzz.html). Rust is a high performance computing descendant of the OCaml/ML functional programming lineage, hence the joke that a Rust program can never compile, but once it does it's bulletproof.

Like D, Rust allows tests to be written in the same source file as the implementation.

[cucumber-rs](https://cucumber-rs.github.io/cucumber/main/) provides an integration test framework for Rust projects.

[tinyrick](https://github.com/mcandre/tinyrick) is a make alternative enabling Rust developers to implement build tasks in Rust code.

### sh

[beltaload](https://github.com/mcandre/beltaloada) provides a convention for implementing build tasks for sh projects in sh code.

# SEE ALSO

* [Chaos engineering](https://en.wikipedia.org/wiki/Chaos_engineering) identifies weaknesses in system fault-tolerance
* [CI/CD](https://en.wikipedia.org/wiki/CI/CD) services automate test suites, ensuring that tests pass before changes merge
* [High-performance computing](https://en.wikipedia.org/wiki/High-performance_computing) emphasizes minimal elapsed time computing, in various data scales
* [mcandre/linters](https://github.com/mcandre/linters) curates static analysis tools
* [Real-time computing](https://en.wikipedia.org/wiki/Real-time_computing) establishes predictable elapsed time computing specifications
