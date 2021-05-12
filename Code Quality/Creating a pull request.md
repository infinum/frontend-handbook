> If the reviews hurt they're probably right on some level. - Sean Lennon

In the Infinum JavaScript team we love code reviews. We love getting them and giving them. This document aims to show how a good pull request should look like (no matter what SCM solution or tool). The main idea is to give your reviewer as much context as possible to allow them to focus on the feature or fix you have delivered.

## Creating a pull request

Your pull request should have **all of the below**:

-   The code should have been linted and prettified before the code review
-   Pull requests should not be large and should contain a single feature
    -   Create multiple pull requests in smaller chunks if needed
-   Task ID (Productive, Jira, etc.)
-   A title
-   A description that includes
    -   Links to the design files that you used
    -   Relevant links to documentation outlining your work
    -   A screenshot of a feature or fix you delivered

The following are extra but highly recommended:

-   A summary of changes and reasoning behind them
-   A TODO list of missing details on large pull requests
-   A list of open questions if there are any
-   A video demonstration of the feature

## Example

![Pull request example](/img/pr_example.png)

-   Marked **A** title showing a concise description along with a task ID
-   Marked **B** description containing relevant documentation and links
-   Marked **C** a screenshot of the developed feature

## Implementing feedback

Once you fix up a comment leave a response that you've fixed that specific comment. This will help you reviewer do another pass on the feature when you hand it over for a re-review.

Going the extra mile would be to link the commit that fixes the comment.

If the pull request becomes stale in the mean time (diverges a lot form the base branch) consider merging the base branch back in so you get the review on possible merge conflicts as well.

## When and who merges the pull request?

When the code review is completed with an appropriate number of approvers having signed off the feature the code can be merged.

You, as the developer, know best when and how this code should be merged. You press the green merge button!
