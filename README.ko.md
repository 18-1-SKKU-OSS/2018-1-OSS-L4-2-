\# SmartThings Developer Documentation

\# SmartThings 개발자 문서

This is the repository for the public SmartThings developer
documentation.

이 저장소는 SmartThings 개발자 공문서 용 저장소입니다.

This documentation is written using
\[reStructuredText\](http://docutils.sourceforge.net/rst.html), powered
by \[Sphinx\](http://www.sphinx-doc.org/en/stable/), and hosted on
\[ReadTheDocs\]([*http://readthedocs.org*](http://readthedocs.org)).

이 문서는
\[restructuredText\]([*http://docutils.sourceforge.net/rst.html*](http://docutils.sourceforge.net/rst.html))을
이용해 작성되었으며,
\[Sphinx\]([*http://www.sphinx-doc.org/en/stable/*](http://www.sphinx-doc.org/en/stable/))
기술을 이용하고
\[ReadTheDocs\]([*http://readthedocs.org*](http://readthedocs.org))에서
호스팅 되었습니다.

\#\# Building the docs

\#\# 문서 작성하기

Follow these steps to build the documentation locally:

문서를 로컬로 작성하려면 아래 단계를 따르세요:

1\. \[Install
virtualenv\]([*https://virtualenv.pypa.io/en/latest/installation.html*](https://virtualenv.pypa.io/en/latest/installation.html)).

1\. \[virtualenv를 설치하세요\](
[*https://virtualenv.pypa.io/en/latest/installation.html*](https://virtualenv.pypa.io/en/latest/installation.html)).

2\. Create an isolated environment in a different location than the
Documentation directory: \`virtualenv --no-site-packages .venv\`

2\. 문서 디렉토리와 다른 곳에 독립된 환경을 만드세요: \`virtualenv
--no-site-packages .venv\`

3\. Activate the environment: \`source .venv/bin/activate\`

3\. 환경을 활성화시키세요: \`source .venv/bin/activate\`

4\. Install dependencies: \`(.venv)\~/Documentation\$ pip install -r
requirements.txt\`

4\. 디펜던시를 설치하세요: \`(.venv)\~/Documentation\$ pip install -r
requirements.txt\`

5\. Build HTML: \`(.venv)\~/Documentation\$ make html\`

5\. HTML을 작성하십시오: \`(.venv)\~/Documentation\$ make html\`

6\. Open \`\_build/html/index.html\` in a web browser.

6\. 웹 브라우저에서 \`\_build/html/index.html\`을 여세요.

To see the available make targets, simply execute \`make\`.

이용 가능한 make 목적파일을 보고싶다면, \`make\`을 실행하세요.

\#\# Contributing

\#\# 기여

We love contributions! If you find a typo, error, or think something can
be communicated better, fork this repository and make a pull request.

우리는 여러분의 기여를 환영합니다! 오타, 에러나 더 발전시킬 수 있을 것
같은 부분이 있다면 이 저장소를 fork하고 pull request를 보내주세요

If you have a larger change that might involve a lot of new content or
organization, let us know in advance by creating an issue.

새로운 내용이나 조직이 많이 포함된 변화를 주고자 한다면 이슈를 만들어
저희에게 먼저 알려주세요.

\#\#\# Style guide

\#\#\# 형식 안내

For documentation formatting and syntax, please see the \[Writing the
Docs
Guide\]([*http://docs.smartthings.com/en/latest/contributing/style-guide.html*](http://docs.smartthings.com/en/latest/contributing/style-guide.html)).

문서의 양식이나 문법은 \[Writing the Docs
Guide\]([*http://docs.smartthings.com/en/latest/contributing/style-guide.html*](http://docs.smartthings.com/en/latest/contributing/style-guide.html))을
꼭 참고해주세요.

\#\#\# Pull request guidelines

\#\#\# Pull request 안내

We use the fork-and-pull GitHub flow.

우리는 fork와 pull의 GitHub flow를 사용합니다.

You should fork this repository, configure a remote for your fork, and
keep your fork in sync.

This is all documented in the \[Working with
forks\](https://help.github.com/articles/working-with-forks) GitHub
documentation.

Once your fork is configured and synced with the latest, you should make
your changes on a topic branch, commit those changes, and push the
branch to make a Pull Request:

1\. Checkout a topic branch based off your master branch (this should be
synced with the SmartThingsCommunity/Documentation repository):

\`\$ git checkout -b topic-branch-name origin/master\`

Where "topic-branch-name" is descriptive of your changes (for example,
"fix-scheduling-broken-link").

2\. Once you are confident in your changes, you can add them to your
index and commit them. Be sure to include a meaningful commit message.
Each PR should have exactly one commit. If you have multiple commits on
your branch, squash them before making the PR.

\*\*NOTE:\*\* SmartThings employees should include the JIRA ticket
number at the beginning of the commit message (e.g., "\[DOCS-234\] add
documentation for passing data to scheduler method callbacks").

3\. Push your topic branch to your fork's remote:

\`\$ git push origin topic-branch-name\`

4\. Visit your fork's remote in a web browser. There will be a yellow
banner indicating new changes have been pushed, with the option to make
a pull request from them. Click that link.

5\. On the Pull Request page, verify your changes look correct, add any
description necessary, and submit the pull request. The documentation
team will always review any pull requests, but if anything requires the
attention of someone specific, \`@\` them in the pull request
description.

6\. If there are changes requested as part of the pull request review,
you can either add new commits to the PR, or amend your existing commit.
If you add new commits, be sure to \[squash them to one
commit\](https://github.com/ginatrapani/todo.txt-android/wiki/Squash-All-Commits-Related-to-a-Single-Issue-into-a-Single-Commit)
before merging.

7\. Once the PR has a +1 (thumbs-up emoji), a member of the docs team
will merge your PR.

\#\# Contact

You can reach us at &lt;mailto:docs@smartthings.com&gt; with any
feedback or questions.
