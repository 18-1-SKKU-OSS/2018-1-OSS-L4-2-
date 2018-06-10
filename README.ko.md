# SmartThings 개발자 문서

이 저장소는 SmartThings 개발자 공문서 용 저장소입니다.

이 문서는 [restructuredText](http://docutils.sourceforge.net/rst.html)을
이용해 작성되었으며, [Sphinx](http://www.sphinx-doc.org/en/stable/) 기술을 이용하고
[ReadTheDocs](http://readthedocs.org) 에서 호스팅 되었습니다.

## 문서 작성하기

문서를 로컬로 작성하려면 아래 단계를 따르세요:

1. [virtualenv를 설치하세요](https://virtualenv.pypa.io/en/latest/installation.html).

2. 문서 디렉토리와 다른 곳에 독립된 환경을 만드세요: `virtualenv--no-site-packages .venv`

3. 환경을 활성화시키세요: `source .venv/bin/activate`

4. 디펜던시를 설치하세요: `(.venv)~/Documentation$ pip install -r
requirements.txt`

5. HTML을 작성하십시오: `(.venv)~/Documentation$ make html`

6. 웹 브라우저에서 `_build/html/index.html`을 여세요.

이용 가능한 make 목적파일을 보고싶다면, `make`을 실행하세요.

## 기여

우리는 여러분의 기여를 환영합니다! 오타, 에러나 더 발전시킬 수 있을 것
같은 부분이 있다면 이 저장소를 fork하고 pull request를 보내주세요

새로운 내용이나 조직이 많이 포함된 변화를 주고자 한다면 이슈를 만들어
저희에게 먼저 알려주세요.

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

이 저장소를 fork하고 fork에 대한 원격저장소 환경을 설정해야 하며 fork를 동기화 상태로 유지해야합니다.

This is all documented in the \[Working with
forks\](https://help.github.com/articles/working-with-forks) GitHub
documentation.

이 모든 것은 \[Working with
forks\](https://help.github.com/articles/working-with-forks) 깃허브 문서에 작성되어 있습니다.

Once your fork is configured and synced with the latest, you should make
your changes on a topic branch, commit those changes, and push the
branch to make a Pull Request:

fork가 최신 정보로 환경 설정 및 동기화되면 토픽 브랜치에서 변경사항을 만들고, 변경사항을 커밋해야 하며 Pull Request를 하기 위해 push해야합니다.

1\. Checkout a topic branch based off your master branch (this should be
synced with the SmartThingsCommunity/Documentation repository):

1\. master 브랜치를 기반으로 토픽 브랜치를 checkout 하세요. (SmartThingsCommunity/Documentation 저장소와 동기화되어야 합니다.):

\`\$ git checkout -b topic-branch-name origin/master\`

Where "topic-branch-name" is descriptive of your changes (for example,
"fix-scheduling-broken-link").

여기서 "topic-branch-name"이 변경사항 내용입니다. (예: 스케쥴링과 끊어진 링크 수정)

2\. Once you are confident in your changes, you can add them to your
index and commit them. Be sure to include a meaningful commit message.
Each PR should have exactly one commit. If you have multiple commits on
your branch, squash them before making the PR.

2\. 변경 사항에 대해 확신이 들면 색인에 변경사항을 추가하고 커밋하세요. 변경사항의 내용을 잘 전달하는 커밋 메시지를 포함하도록 하세요. 
각 PR에는 하나의 커밋만 있어야합니다. 브랜치에 여러 커밋이 있는 경우, PR을보내기 전, 여러 커밋 로그를 하나로 묶으세요.

\*\*NOTE:\*\* SmartThings employees should include the JIRA ticket
number at the beginning of the commit message (e.g., "\[DOCS-234\] add
documentation for passing data to scheduler method callbacks").

\*\*참고사항:\*\* SmartThings 직원은 JIRA 티켓 번호를 커밋 메시지의 앞에 포함해야합니다. (예: "\[DOCS-234\] 스케쥴러 메소드 회신에 데이터를 전달하기 위한 문서 추가")

3\. Push your topic branch to your fork's remote:

3\. fork의 원격 저장소에 토픽 브랜치를 push 하세요.

\`\$ git push origin topic-branch-name\`

4\. Visit your fork's remote in a web browser. There will be a yellow
banner indicating new changes have been pushed, with the option to make
a pull request from them. Click that link.

4\. 웹 브라우저에서 fork의 원격저장소에 들어가세요. push된 새로운 변경사항을 알리는 노란 배너가 있을 것입니다. 이 배너로 변경사항들의 pull request를 만들 수 있습니다. 해당 링크를 클릭하세요.

5\. On the Pull Request page, verify your changes look correct, add any
description necessary, and submit the pull request. The documentation
team will always review any pull requests, but if anything requires the
attention of someone specific, \`@\` them in the pull request
description.

5\. Pull Request 페이지에서 변경 내용이 올바른 지 확인하고, 필요하다면 설명을 덧붙이십시오. 그리고 pull request를 제출하세요. 저희 문서 팀은 항상 모든 pull request를 검토하겠지만 특정 팀원의 검토가 필요하다면 pull request에서 그들을 \`@\`를 이용해 언급하세요.

6\. If there are changes requested as part of the pull request review,
you can either add new commits to the PR, or amend your existing commit.
If you add new commits, be sure to \[squash them to one
commit\](https://github.com/ginatrapani/todo.txt-android/wiki/Squash-All-Commits-Related-to-a-Single-Issue-into-a-Single-Commit)
before merging.

6\. pull request를 올린 것 중 변경 사항이 있다면 PR에 새로운 커밋을 추가하거나 기존 커밋을 수정할 수 있습니다.
새 커밋을 추가하는 경우 \[merge 전에 커밋 로그를 하나로 묶으세요.\](https://github.com/ginatrapani/todo.txt-android/wiki/Squash-All-Commits-Related-to-a-Single-Issue-into-a-Single-Commit)

7\. Once the PR has a +1 (thumbs-up emoji), a member of the docs team
will merge your PR.

7\. PR에 +1 (엄지를 든 모양의 이모티콘)이 있다면 문서 팀이 당신의 PR을 merge 해줄 것입니다.

\#\# Contact

\#\# 연락처

You can reach us at &lt;mailto:docs@smartthings.com&gt; with any
feedback or questions.

&lt;mailto:docs@smartthings.com&gt; 를 통해 어떤 피드백이나 문의사항을 저희에게 전달하실 수 있습니다.
