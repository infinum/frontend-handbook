> Optimism is an occupational hazard of programming; feedback is the treatment. - Kent Beck

This guide should help you review code better. A good code review will provide a lot of useful things, but some of them are:

-   a sense of security (that the feature works well),
-   a learning opportunity for both parties,
-   architectural check,
-   code quality check.

## Language of the code review

-   Be concise
-   Do not ask, advise
-   Do not assume the level of knowledge
-   Be open for discussion, but do not bikeshed
-   Recommend fixes instead of pointing out errors

### Good examples:

> this won't work because the function is `async`.

> this file should go to `components/shared` because this is atom component and can be reused on another page

> I'm not sure this does what it's meant to do. Looking at the service as a whole the code should probably allow the end user to construct this out of exported functions.

### Bad examples:

> can you please update this function documentation as well?

> this is not according to our guidelines

> naming is not correct here

> pls fix.

## When do I review pull requests?

Reviewing a pull request could (and should) take time. This is not to be rushed since you vouch for the quality of the code that's on review.

Pull requests should not stay unattended for more than 24 hours. This means you should pick them up and start the review in that period. When you do them however is entirely up to you and your team.

This can be specifically hard to do if you review multiple projects or work in a larger team. If this is the case you might want to think about assigning a certain part of the day to do them.

Be sure to communicate this to your project manager so it gets reflected in your sprint availability.

Code review is an integral part of the development cycle and therefore should be time tracked on the appropriate task/service on the project.

## How do I review a pull request?

Start by reading the feature that's delivered. Look at the design files, get to know the business logic behind it, understand the requirements.

Next up is scanning the files for common errors or typos, etc. - things that stand out right away. Comment them and if they are blocking enough return the pull request to the developer.

You should have the project you're reviewing checked out locally and you should run **both** the project and the tests on **every** pull request. Your review is not complete unless you click through the feature and verify at least the happy path.

Take a look at the code coverage and look for missing things there, are all branches tested or even developed?

You, too, are taking part in the chain of quality assurance here.

## Common things to look out for

-   If the code is not linted or reviewed return the pull request to the developer
-   The code should be tested using unit tests
-   The code should not lower the code coverage on the project
-   The code should follow the design specification
-   Tricky parts of the code that are hard to understand should have either inline documentation or a link to external documentation

Some of these checks can be automated on pull requests (e.g. code coverage). Talk to your resident devops to help you set this up.

## Responsibility

Every developer is responsible for their code. But - you as a reviewer share the responsibility as well when you sign off a feature and allow it to merge.
