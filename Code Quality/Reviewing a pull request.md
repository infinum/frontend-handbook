> Optimism is an occupational hazard of programming; feedback is the treatment. - Kent Beck

This guide should help you review code better. A good code review will provide a lot of useful things, but some of them are:

-   a sense of security (that the feature works well),
-   a learning opportunity for both parties,
-   architectural check,
-   code quality check.

## Language of the code review

-   be concise
-   do not ask, advise
-   do not assume the level of knowledge
-   be open for discussion, but do not bike ßshed
-   recommend fixes instead of pointing out errors

### Good examples:

> this won't work because the function you're calling as `async`.

> this file is palaced in the incorrect folder, it should go to `components/shared`

> I'm not sure this does what you want it to do. Looking at the service as a whole you should probably allow the end user to construct this out of available functions.

### Bad examples:

> can you please update this function documentation as well?

> this is not according to our guidelines

> naming is not correct here

## When do I review pull requests?

Reviewing a pull request could (and should) take time. This is not to be rushed since you vouch for the quality of the code that's on review.

If you review code for either multiple projects or you work in a larger team you might want to think about assigning a certain fixed part of the day to do them (e.g. I review pull requests in the morning). Everything that comes during the day is to be reviewed first thing tomorrow. This should not include urgent things that block hot fixes, etc.

Code review is an integral part of the development cycle and therefore should be time tracked on the appropriate task/service on the project.

## How do I review a pull request?

Start by reading the feature that's delivered. Look at the design files, get to know the business logic behind it, understand the requirements.

Next up is scanning the files for common errors or typos, etc. - things that stand out right away. Comment them and if they are blocking enough return the pull request to the developer.

You should have the project you're reviewing checked out locally and you should run **both** the project and the tests on **every** pull request. Your review is not complete unless you click thru the feature and verify at least the happy path.

Take a look at the code coverage and look for missing things there, are all branched tested or even developed?

You, too, are taking part in the chain of quality assurance here.

## Common things to look out for

-   the code should have been linted and prettified before the code review
-   the code should be tested using unit tests
-   the code should not lower the code coverage on the project
-   the code should follow the design specification
-   tricky parts of the code that are hard to understand should have either inline documentation or a link to external documentation

## Responsibility

Every developer is responsible for their code. But - you as a reviewer share the responsibility as well when you sign off a feature and allow merge of it.
