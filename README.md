Written here are collection of ideas and realizations that have emerged from my experiences as a software engineer.

## Over Abstraction
There are 3 main considerations we make when dealing with code:
- _Velocity_ - it is when we prioritize shipping the changes as quick as possible
- _Adaptability_ - it is when we write extensible code to lessen the friction for code changes
- _Performance_ - it is when we make optimized code

**Abstraction** is a critical aspect of creating adaptable code. However, excessive use of abstraction can negatively impact velocity and performance.

### Premature Abstraction
Often times we make abstractions because we assume that the implementation will be reused in the future. Take for example:

