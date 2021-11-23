# Production Deployments

When deploying to production work with your tech lead to ensure that your branch is ready to push live. Review the change log, test the new functionality again, explain your code to another developer and deploy to staging one more time.

Once deployed tag your release.

`git tag -a v1.2 9fceb02 -m "Message here"`

Where `9fceb02` is the beginning part of the commit id.

You can then push them up using `git push --tags origin master`
