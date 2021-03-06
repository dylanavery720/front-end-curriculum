---
title: Better Workflow with Git Hooks
length: 2 hours
tags: git, workflow, agile
module: 4
---

### Goals

By the end of this lesson, you will:

* Understand how to create and use git hooks to automate common developer workflow processes

## Automating the Grunt Work
As developers, we're constantly looking for ways to automate tedious tasks. We're essentially trying to put ourselves out of the job by writing scripts that will do it for us.

## Git Hooks
Another tool we can leverage that works nicely with CI are [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks). Git hooks are custom scripts that you can write to execute particular tasks during certain points of the git workflow process. Similar to the lifecycle methods in React or other JavaScript frameworks, you'll recognize some lifecycle patterns within the git version control system. For example, every time we make a commit, there are four distinct phases: 

* `pre-commit` - runs before the commit is even attempted, can be used to do a quick QA evaluation on the code that's about to be committed
* `prepare-commit-msg` - runs before the commit message is made but after a default message is created, e.g. merge commits are auto-generated and can be adjusted at this point in the cycle
* `commit-msg` - runs after the commit message has been made, can be used to verify that your message follows a required pattern (e.g. capital first letter, no punctuation, command-style sentence)
* `post-commit` - runs after the entire commit process is completed, can be used to run another script based on information from the most recent commit

Git allows us to effectively 'pause' the commit cycle at each of these four phases through the use of hooks. We just saw how CircleCI builds will sometimes fail if we have a failing test or our code doesn't pass the linting rules we've set. Builds can take significant time when we have complex applications, so we want to minimize the chances that we'll start up a build that's going to fail. One way we can do that is by creating a `pre-commit` hook that prevents us from committing code that doesn't conform to our linting and testing standards.

### Creating a `pre-commit` hook
Whenever we create a new local git repo, a `.git` directory is included in our project. If we were to `cd` into `.git/hooks` we can actually see a couple of sample hooks that git provides us with by default. You'll notice the filenames all end with `.sample`. This is what's preventing them from actually running. If we want to implement one of these hooks, we could simply remove the `.sample` suffix. 

Let's rename `pre-commit.sample` to `pre-commit` and open it in your text editor. We're not going to use any of the example functionality that was included, so we can remove everything in this file and replace it with the following:

```bash
#!/bin/sh

echo "\Running tests:\n"

npm run test --silent
```

Now if we have a failing test, our pre-commit hook should catch that and prevent the commit from going through. This hook is super simple right now, because a failing test automatically causes the process to exit with an error. If we were to add linting to this hook:

```bash
#!/bin/sh

echo "\Running tests:\n"

npm run test --silent
npm run lint --silent
```

We could technically still get our commit through as long as our tests are passing. This is because the linting process doesn't actually exit with an error if there are linting problems. In order to exit the process after reporting linting errors, we need to make our pre-commit hook a little more advanced:

```bash
#!/bin/sh

echo "\Running tests:\n"

npm run test --silent

files=$(git diff --cached --name-only --diff-filter=ACM | grep "\.js$")
if [ "$files" = "" ]; then 
    exit 0 
fi

lintPass=true

echo "\nValidating JavaScript:\n"

for file in ${files}; do
    result=$(npm run lint ${file} --silent | grep "${file} is OK")
    if [ "$result" != "" ]; then
        echo "\t\033[32mJSHint Passed: ${file}\033[0m"
    else
        echo "\t\033[31mJSHint Failed: ${file}\033[0m"
        lintPass=false
    fi
done

echo "\nJavaScript validation complete\n"

if ! $lintPass; then
    echo "\033[41mCOMMIT FAILED:\033[0m Your commit contains files that should pass JSHint but do not. Please fix the JSHint errors and try again.\n"
    exit 1
else
    echo "\033[42mCOMMIT SUCCEEDED\033[0m\n"
fi
```

*Note: In order for a hook file to run, it must be made executeable. You can make a file executeable by running:* 

```bash
chmod +x hooks/filename
```

### More Hooks
There are additional hooks for facilitating a custom email workflow and manipulating other git actions such as rebasing. These aren't used quite as often as the commit hooks, but it's good to be aware they exist.

### Sharing Hooks
By default, git hooks exist in the `.git/hooks` directory of your local repo. You'll notice that this isn't a directory that gets pushed to github, so when new contributors pull down your repo, they won't have the same hooks in place that the rest of the team might be using. There are a couple of ways to get around this.

As of Git 2.9, you can set a git configuration option `core.hooksPath` that will override the default `.git/hooks` directory. This would allow you and your team to put your hooks in an internal, standalone repository that each developer could clone down and reference with the `core.hooksPath` option.

In earlier versions of Git, you could implement this same strategy by creating a symlink from the `.git/hooks` directory to your local repository of your team's git hooks.

## Resources

* [Git Hooks Guide](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
* [Git Hooks Reference](https://git-scm.com/docs/githooks)
* [Github Issues from Command Line](https://github.com/stephencelis/ghi)
* [Node.js pre-commit](https://github.com/observing/pre-commit)
* [Fit Commit](https://github.com/m1foley/fit-commit)
