
[中文版](./README-zh.md)
 | [日本語版](./README-ja.md)
 | [한국어](./README-ko.md)

[<img src="./images/elsewhen-logo.png" width="180" height="180">](http://elsewhen.co/)


# Project Guidelines &middot; [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)
> While developing a new project is like rolling on a green field for you, maintaining it is a potential dark twisted nightmare for someone else.
Here's a list of guidelines we've found, written and gathered that (we think) works really well with most JavaScript projects here at [elsewhen](http://elsewhen.co).
If you want to share a best practice, or think one of these guidelines should be removed, [feel free to share it with us](http://makeapullrequest.com).
- [Git](#git)
    - [Some Git rules](#some-git-rules)
    - [Git workflow](#git-workflow)
    - [Writing good commit messages](#writing-good-commit-messages)
- [Documentation](#documentation)
- [Environments](#environments)
    - [Consistent dev environments](#consistent-dev-environments)
    - [Consistent dependencies](#consistent-dependencies)
- [Dependencies](#dependencies)
- [Testing](#testing)
- [Structure and Naming](#structure-and-naming)
- [Code style](#code-style)
    - [Some code style guidelines](#code-style-check)
    - [Enforcing code style standards](#enforcing-code-style-standards)
- [Logging](#logging)
- [API](#api)
    - [API design](#api-design)
    - [API security](#api-security)
    - [API documentation](#api-documentation)
- [Licensing](#licensing)

<a name="git"></a>
## 1. Git
![Git](/images/branching.png)
<a name="some-git-rules"></a>

### 1.1 Some Git rules
There are a set of rules to keep in mind:
* Perform work in a feature branch.
    
    _Why:_
    >Because this way all work is done in isolation on a dedicated branch rather than the main branch. It allows you to submit multiple pull requests without confusion. You can iterate without polluting the master branch with potentially unstable, unfinished code. [read more...](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow)
* Branch out from `develop`
    
    _Why:_
    >This way, you can make sure that code in master will almost always build without problems, and can be mostly used directly for releases (this might be overkill for some projects).

* Never push into `develop` or `master` branch. Make a Pull Request.
    
    _Why:_
    > It notifies team members that they have completed a feature. It also enables easy peer-review of the code and dedicates forum for discussing the proposed feature.

* Update your local `develop` branch and do an interactive rebase before pushing your feature and making a Pull Request.

    _Why:_
    > Rebasing will merge in the requested branch (`master` or `develop`) and apply the commits that you have made locally to the top of the history without creating a merge commit (assuming there were no conflicts). Resulting in a nice and clean history. [read more ...](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

* Resolve potential conflicts while rebasing and before making a Pull Request.
* Delete local and remote feature branches after merging.
    
    _Why:_
    > It will clutter up your list of branches with dead branches. It ensures you only ever merge the branch back into (`master` or `develop`) once. Feature branches should only exist while the work is still in progress.

* Before making a Pull Request, make sure your feature branch builds successfully and passes all tests (including code style checks).
    
    _Why:_
    > You are about to add your code to a stable branch. If your feature-branch tests fail, there is a high chance that your destination branch build will fail too. Additionally, you need to apply code style check before making a Pull Request. It aids readability and reduces the chance of formatting fixes being mingled in with actual changes.

* Use [this](./.gitignore) `.gitignore` file.
    
    _Why:_
    > It already has a list of system files that should not be sent with your code into a remote repository. In addition, it excludes setting folders and files for most used editors, as well as most common dependency folders.

* Protect your `develop` and `master` branch.
  
    _Why:_
    > It protects your production-ready branches from receiving unexpected and irreversible changes. read more... [Github](https://help.github.com/articles/about-protected-branches/), [Bitbucket](https://confluence.atlassian.com/bitbucketserver/using-branch-permissions-776639807.html) and [GitLab](https://docs.gitlab.com/ee/user/project/protected_branches.html)

<a name="git-workflow"></a>
### 1.2 Git workflow
Because of most of the reasons above, we use [Feature-branch-workflow](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow) with [Interactive Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing) and some elements of [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow) (naming and having a develop branch). The main steps are as follows:

* For a new project, initialize a git repository in the project directory. __For subsequent features/changes this step should be ignored__.
   ```sh
   cd <project directory>
   git init
   ```

* Checkout a new feature/bug-fix branch.
    ```sh
    git checkout -b <branchname>
    ```
* Make Changes.
    ```sh
    git add
    git commit -a
    ```
    _Why:_
    > `git commit -a` will start an editor which lets you separate the subject from the body. Read more about it in *section 1.3*.

* Sync with remote to get changes you’ve missed.
    ```sh
    git checkout develop
    git pull
    ```
    
    _Why:_
    > This will give you a chance to deal with conflicts on your machine while rebasing (later) rather than creating a Pull Request that contains conflicts.
    
* Update your feature branch with latest changes from develop by interactive rebase.
    ```sh
    git checkout <branchname>
    git rebase -i --autosquash develop
    ```
    
    _Why:_
    > You can use --autosquash to squash all your commits to a single commit. Nobody wants many commits for a single feature in develop branch. [read more...](https://robots.thoughtbot.com/autosquashing-git-commits)
    
* If you don’t have conflicts, skip this step. If you have conflicts, [resolve them](https://help.github.com/articles/resolving-a-merge-conflict-using-the-command-line/)  and continue rebase.
    ```sh
    git add <file1> <file2> ...
    git rebase --continue
    ```
* Push your branch. Rebase will change history, so you'll have to use `-f` to force changes into the remote branch. If someone else is working on your branch, use the less destructive `--force-with-lease`.
    ```sh
    git push -f
    ```
    
    _Why:_
    > When you do a rebase, you are changing the history on your feature branch. As a result, Git will reject normal `git push`. Instead, you'll need to use the -f or --force flag. [read more...](https://developer.atlassian.com/blog/2015/04/force-with-lease/)
    
    
* Make a Pull Request.
* Pull request will be accepted, merged and close by a reviewer.
* Remove your local feature branch if you're done.

  ```sh
  git branch -d <branchname>
  ```
  to remove all branches which are no longer on remote
  ```sh
  git fetch -p && for branch in `git branch -vv --no-color | grep ': gone]' | awk '{print $1}'`; do git branch -D $branch; done
  ```

<a name="writing-good-commit-messages"></a>
### 1.3 Writing good commit messages

Having a good guideline for creating commits and sticking to it makes working with Git and collaborating with others a lot easier. Here are some rules of thumb ([source](https://chris.beams.io/posts/git-commit/#seven-rules)):

 * Separate the subject from the body with a newline between the two.

    _Why:_
    > Git is smart enough to distinguish the first line of your commit message as your summary. In fact, if you try git shortlog, instead of git log, you will see a long list of commit messages, consisting of the id of the commit, and the summary only.

 * Limit the subject line to 50 characters and Wrap the body at 72 characters.

    _why_
    > Commits should be as fine-grained and focused as possible, it is not the place to be verbose. [read more...](https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c)

 * Capitalize the subject line.
 * Do not end the subject line with a period.
 * Use [imperative mood](https://en.wikipedia.org/wiki/Imperative_mood) in the subject line.

    _Why:_
    > Rather than writing messages that say what a committer has done. It's better to consider these messages as the instructions for what is going to be done after the commit is applied on the repository. [read more...](https://news.ycombinator.com/item?id=2079612)


 * Use the body to explain **what** and **why** as opposed to **how**.

 <a name="documentation"></a>
## 2. Documentation

![Documentation](/images/documentation.png)

* Use this [template](./README.sample.md) for `README.md`, Feel free to add uncovered sections.
* For projects with more than one repository, provide links to them in their respective `README.md` files.
* Keep `README.md` updated as a project evolves.
* Comment your code. Try to make it as clear as possible what you are intending with each major section.
* If there is an open discussion on github or stackoverflow about the code or approach you're using, include the link in your comment. 
* Don't use comments as an excuse for a bad code. Keep your code clean.
* Don't use clean code as an excuse to not comment at all.
* Keep comments relevant as your code evolves.

<a name="environments"></a>
## 3. Environments

![Environments](/images/laptop.png)

* Define separate `development`, `test` and `production` environments if needed.

    _Why:_
    > Different data, tokens, APIs, ports etc... might be needed in different environments. You may want an isolated `development` mode that calls fake API which returns predictable data, making both automated and manual testing much easier. Or you may want to enable Google Analytics only on `production` and so on. [read more...](https://stackoverflow.com/questions/8332333/node-js-setting-up-environment-specific-configs-to-be-used-with-everyauth)


* Load your deployment specific configurations from environment variables and never add them to the codebase as constants, [look at this sample](./config.sample.js).

    _Why:_
    > You have tokens, passwords and other valuable information in there. Your config should be correctly separated from the app internals as if the codebase could be made public at any moment.

    _How:_
    >
    `.env` files to store your variables and add them to `.gitignore` to be excluded. Instead, commit a `.env.example`  which serves as a guide for developers. For production, you should still set your environment variables in the standard way.
    [read more](https://medium.com/@rafaelvidaurre/managing-environment-variables-in-node-js-2cb45a55195f)

* It’s recommended to validate environment variables before your app starts.  [Look at this sample](./configWithTest.sample.js) using `joi` to validate provided values.
    
    _Why:_
    > It may save others from hours of troubleshooting.

<a name="consistent-dev-environments"></a>
### 3.1 Consistent dev environments:
* Set your node version in `engines` in `package.json`.
    
    _Why:_
    > It lets others know the version of node the project works on. [read more...](https://docs.npmjs.com/files/package.json#engines)

* Additionally, use `nvm` and create a  `.nvmrc`  in your project root. Don't forget to mention it in the documentation.

    _Why:_
    > Any one who uses `nvm` can simply use `nvm use` to switch to the suitable node version. [read more...](https://github.com/creationix/nvm)

* It's a good idea to setup a `preinstall` script that checks node and npm versions.

    _Why:_
    > Some dependencies may fail when installed by newer versions of npm.
    
* Use Docker image if you can.

    _Why:_
    > It can give you a consistent environment across the entire workflow. Without much need to fiddle with dependencies or configs. [read more...](https://hackernoon.com/how-to-dockerize-a-node-js-application-4fbab45a0c19)

* Use local modules instead of using globally installed modules.

    _Why:_
    > Lets you share your tooling with your colleague instead of expecting them to have it globally on their systems.


<a name="consistent-dependencies"></a>
### 3.2 Consistent dependencies:

* Make sure your team members get the exact same dependencies as you.

    _Why:_
    > Because you want the code to behave as expected and identical in any development machine [read more...](https://medium.com/@kentcdodds/why-semver-ranges-are-literally-the-worst-817cdcb09277)

    _how:_
    > Use `package-lock.json` on `npm@5` or higher

    _I don't have npm@5:_
    > Alternatively you can use `Yarn` and make sure to mention it in `README.md`. Your lock file and `package.json` should have the same versions after each dependency update. [read more...](https://yarnpkg.com/en/)

    _I don't like the name `Yarn`:_
    > Too bad. For older versions of `npm`, use `—save --save-exact` when installing a new dependency and create `npm-shrinkwrap.json` before publishing. [read more...](https://docs.npmjs.com/files/package-locks)

<a name="dependencies"></a>
## 4. Dependencies

![Github](/images/modules.png)

* Keep track of your currently available packages: e.g., `npm ls --depth=0`. [read more...](https://docs.npmjs.com/cli/ls)
* See if any of your packages have become unused or irrelevant: `depcheck`. [read more...](https://www.npmjs.com/package/depcheck)
    
    _Why:_
    > You may include an unused library in your code and increase the production bundle size. Find unused dependencies and get rid of them.

* Before using a dependency, check its download statistics to see if it is heavily used by the community: `npm-stat`. [read more...](https://npm-stat.com/)
    
    _Why:_
    > More usage mostly means more contributors, which usually means better maintenance, and all of these result in quickly discovered bugs and quickly developed fixes.

* Before using a dependency, check to see if it has a good, mature version release frequency with a large number of maintainers: e.g., `npm view async`. [read more...](https://docs.npmjs.com/cli/view)

    _Why:_
    > Having loads of contributors won't be as effective if maintainers don't merge fixes and patches quickly enough.

* If a less known dependency is needed, discuss it with the team before using it.
* Always make sure your app works with the latest version of its dependencies without breaking: `npm outdated`. [read more...](https://docs.npmjs.com/cli/outdated)

    _Why:_
    > Dependency updates sometimes contain breaking changes. Always check their release notes when updates show up. Update your dependencies one by one, that makes troubleshooting easier if anything goes wrong. Use a cool tool such as [npm-check-updates](https://github.com/tjunnone/npm-check-updates).

* Check to see if the package has known security vulnerabilities with, e.g., [Snyk](https://snyk.io/test?utm_source=risingstack_blog).


<a name="testing"></a>
## 5. Testing
![Testing](/images/testing.png)
* Có một môi trường `test` nếu cần.

_Tại sao:_ 
    > Trong khi thỉnh thoảng end to end testing trong chế độ `production` mode có thể dường như là đủ , vẫn có một số trường hợp ngoại lệ: Một ví dụ là bạn có thể không muốn cho phép phân tích thông tin trên một chế độ 'production' và xấu bảng điều khiển của ai đó với dữ liệu test. Ví dụ khác là API của bạn có thể đạt tới tỉ lệ gới hạn trong chế độ `production` và bị khoá lời gọi test của bạn sau một lượng request nhất định. 

* Đặt tên file test của bạn bên cạnh module đã được test sử dụng quy ước đặt tên `*.test.js` or `*.spec.js` , giống như `moduleName.spec.js`.

    _Tại sao:_
    > Bạn không muốn lục cả cấu trúc thư mick để tìm một unit test. [Đọc thêm...](https://hackernoon.com/structure-your-javascript-code-for-testability-9bc93d9c72dc)
    

* Đặt các file test bổ sung vào thư mục test riêng để tránh nhầm lẫn.

    _Tại sao:_
    > Một vài file test không có sự liên quan đặc biệt để bất kỳ file được thực hiện đặc biệt. Bạn phải đặt nó vào trong một thư mục nơi mà có thể tìm được bởi các developer khác.: thư mục `__test__` . Tên này: `__test__`  cũng là chuẩn bây giờ và được chọn bởi ccacs thư viện Javascript testing.

* Viết code có thể test được, tránh các ảnh hưởng phụ, các ảnh hưởng mở rộng, viết các hàm thuần tuý.

    _Tại sao:_
    > Bạn muốn kiểm tra một luồn làm việc logic như tách thành các phần riêng biệt. Bạn phải "giảm thiểu sự xung đột giữa tiến trình ngẫu nhiên và không ngẫu nhiên trên độ tin cậy code của bạn". [xem thêm...](https://medium.com/javascript-scene/tdd-the-rite-way-53c9b46f45e3)
    
    > Một hàm thuần tuý là một hàm mà luôn luôn trả về đầu ra tương tự khi cho đầu vào tương tự. Ngược lại, một hàm xấu là một hàm mà trong đó có thể có các ảnh hưởng phụ hoặc phụ thuộc điều kiện bên ngoài để tạo ra giá trị. Điều đó làm cho nó khó dự đoán. [xem thêm...](https://hackernoon.com/structure-your-javascript-code-for-testability-9bc93d9c72dc)

* Dùng một bộ kiểm tra kiểu tĩnh.

    _Tại sao:_
    > Thỉnh thoảng bạn có thể cần một bộ kiểm tra kiểu tĩnh. Nó đem đến các mức độ chắc chắc cho độ tin tưởng code của bạn. [đọc thêm...](https://medium.freecodecamp.org/why-use-static-types-in-javascript-part-1-8382da1e0adb)


* Chạy test ở dưới máy trước khi thực hiện bất kì các pull request nào về `develop`.

    _Tại sao:_
    > Bạn không muốn là người mà gây ra việc thất bại khi chạy một nhánh đã sẵn sàng của chế độ production. Chạy các test của bạn sau `rebase` của bạn và trước khi đẩy lên nhánh tính năng của bạn đến một remote repository.

* Tài liệu các test của bạn bao gồm các hướng dẫn trong phần liên quan của file `README.md` của bạn.

    _Tại sao:_
    > Nó là một chú ý tiện dụng bạn để lại cho các developer khác hoặc các chuyên gia DevOps hoặc QS hoặc bất kỳ ai người mà có đủ may mắn để làm việc trên code của bạn.

<a name="structure-and-naming"></a>
## 6. Kiến trúc và đặt tên
![Structure and Naming](/images/folder-tree.png)
* Tổ chức các file của bạn xung quanh các tính năng / các trang / các thành phần của sản phẩm, không bắt buộc. Như vậy đặt các file test của bạn bên cạnh nơi thực hiện chúng.


    **Xấu**

    ```
    .
    ├── controllers
    |   ├── product.js
    |   └── user.js
    ├── models
    |   ├── product.js
    |   └── user.js
    ```

    **Tốt**

    ```
    .
    ├── product
    |   ├── index.js
    |   ├── product.js
    |   └── product.test.js
    ├── user
    |   ├── index.js
    |   ├── user.js
    |   └── user.test.js
    ```

    _Tại sao:_
    > Thay vì một danh sách dài các file, bạn sẽ tạo các module nhỏ nơi mà đóng gói với nhiệm vụ bao gồm cả test nó và đi tiếp. Nó dễ dàng điều hướng hơn và mọi thứ có thể tìm trong nháy mắt.

* Đặt các file test bổ sung của bạn vào một thư mục riêng biệt để tránh nhầm lẫn.

    _Tại sao:_
    > Nó tiết kiệm thời gian cho các developer khác hoặc các chuyên gia DevOps trong nhóm của bạn.

* Dùng thư mục `./config` và không tạo các file cấu hình khác nhau cho các môi trường khác nhau.

    _Tại sao:_
    >Khi bạn phá vỡ một file cấu hình cho các mục đích khác nhau (database, API và hơn thế nữa); đặt chúng vào một thư mục với một cái tên dễ nhận biết như `config` có ý nghĩa. Chỉ cần nhớ không tạo ra các file cáu hình khác nhau cho các môi trường khác nhau. Nó không mở rộng dễ, khi triển khai thêm ứng dụng đã được tạo ra, tên môi trường mới là cần thiết. Giá trị được sử dụng trong các file cấu hình nên được cung cấp bởi các biến môi trường. [xem thêm...](https://medium.com/@fedorHK/no-config-b3f1171eecd5)
    

* Đặt các đoạn mã script vào một thư mục `./scripts` . Nó bao gồm các đoạn mã script `bash` và `node` .

    _Tại sao:_
    >Rất có thể bạn kết thúc với nhiều hơn một script, xây dựng product, xây dựng development, tạo dữ liệu, đồng bộ dữ liệu và hơn thế nữa.
    

* Đặt đầu ra quá trình chạy của bạn trong một thư mục `./build`. Thêm `build/` vào `.gitignore`.

    _Tại sao:_
    >Đặt tên cho những gì bạn thích, `dist` cũng giảm bớt. Nhưng đảm bảo rằng giữ nó phù hợp với nhóm của bạn. Những gì lấy được ở đó rất có thể được tạo ra  (đóng gói, biên dịch, chuyển đổi) hoặc di chuyển chúng. Những gì bạn có thể tạo ra, thì đồng đội của bạn cũng nên có thể tạo ra như thế, vì vậy không đặt vị trí commit chúng vào trong remote repository của bạn. Trừ khi bạn muốn đặc biệt.

<a name="code-style"></a>
## 7. Code style

![Code style](/images/code-style.png)

<a name="code-style-check"></a>
### 7.1 Một số hướng dẫn về code style

* Dùng giai đoạn-2 và cú pháp JavaScript cao hơn (hiện đại) cho những dự án mới. Đối với dự án cũ vẫn phù hợp với cũ pháp đã tồn tại trừ khi bạn có ý định nâng cấp dự án.

    _Tại sao:_
    > Tất cả tuỳ thuộc vào bạn. Chúng tôi sử dụng transpilers để sử dụng tính năng nâng cao của cú pháp mới. giai đoạn-2 thì nhiều khả năng để cuối cùng trở thành một phần đặc biệt với chỉ sửa đổi nhỏ.. 

* Bao gồm code style kiểm tra trong quá trình xây dựng của bạn.

    _Tại sao:_
    > Phá vỡ sự xây dựng của bạn là cách bắt buộc định định dạng code trong code của bạn. Nó ngăn ngừa bạn từ việc nó ít nghiêm trọng đi. Làm nó cho cả code của client and server-side. [Đọc thêm...](https://www.robinwieruch.de/react-eslint-webpack-babel/)

* Dùng [ESLint - Pluggable JavaScript linter](http://eslint.org/) để thực hiện định dạng code.

    _Tại sao:_
    > Chúng tôi chỉ đơn giản thích `eslint` hơn, bạn có thể không phải. No có nhiều quy tắc được hỗ trợ hơn, có khả năng để cấu hình những quy tắc đó, và có khả năng thêm các quy tắc tuỳ biến.

* Chúng tôi sử dụng [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) cho JavaScript, [Xem thêm](https://www.gitbook.com/book/duk/airbnb-javascript-guidelines/details). Sử dụng hướng dẫn javascript style được yêu cầu bở dự án hoặc nhóm của bạn.

* Chúng tôi sử dụng [Flow type kiểm tra các quy tắc cho ESLint](https://github.com/gajus/eslint-plugin-flowtype) khi sử dụng [FlowType](https://flow.org/).

    _Tại sao:_
    > Flow giới thiệu vài cú pháp rằng cũng cần phải làm theo code style và được kiểm tra.

* Sử dụng `.eslintignore` để loại trừ các file và các thư mục từ việc kiểm tra code style.

    _Tại sao:_
    > Bạn không phải làm xấu code của bạn với bình luận `eslint-disable` mỗi khi bạn cần loại trừ một cặp các file từ quá trình kiểm tra định dạng.

* Xoá bất kỳ bình luận đã vô hiệu hoá các bình luận `eslint` của bạn trước khi thực hiện yêu cầu Pull.

    _Tại sao:_
    > Nó bình thuường để vô hiệu hoá style kiểm tra trong khi làm việc trên một khối code để tập trung nhiều hơn vào logic. Chỉ cần nhớ để xoá những bình luận `eslint-disable` và theo các quy tắc.

* Tuỳ thuộc vào độ lớn của nhiệm vụ sử dụng  `//TODO:` bình luận hoặc mở một thẻ.

    _Tại sao:_
    > Sau đó bạn có thể nhắc nhở chính mình và những người khác về một nhiệm vụ nhỏ (giống như sắp xếp một hàm hoặc cập nhập một bình luận). Cho nhiệm vụ lớn sử dụng `//TODO(#3456)` được thực hiện bởi một dấu `#` và một con số là một thẻ được mở.


* Luôn luôn bình luận và giữ chúng thích hợp với các thay đổi của code. Xoá bỏ các khối comment của code.
    
    _Tại sao:_
    > Code của bạn nên càng có thể đọc được càng tốt, bạn nên loại bỏ mọi thứ làm mất tập trung. Nếu bạn sắp xếp lại một hàm, không chỉ comment lại code cũ, xoá bỏ nó.

* Tránh bình luận không liên quan hoặc hài hước, logs hoặc đặt tên.

    _Tại sao:_
    > Trong khi xây dựng tiến trình của bạn có thể(nên) loại bỏ chúng, thỉnh thoảng mã nguồn code của bạn có thể được bàn giao cho công ty khác, khách hàng khác và họ không thể đưa ra lời chế nhạo.

* Đặt tên của bạn có thể tìm kiếm được với sự phân biệt có ý nghĩa tránh tên ngắn. Với hàm sử dụng dài, mô tả tên. Một tên hàm nên là một động từ hoặc một cụm động từ, và nó cần truyền đạt ý nghĩa của mình.

    _Tại sao:_
    > Nó làm cho việc đọc mã nguồn trở nên tự nhiên hơn.

* Tổ chức các hàm của bạn trong một file theo quy tắc step-down. Hàm ở mức cao hơn nên ở trên và thấp hơn nên ở dưới.

    _Tại sao:_
    > Nó làm cho việc đọc mã nguồn trở nên tự nhiên hơn.

<a name="enforcing-code-style-standards"></a>
### 7.2 Tuân theo các chuẩn code style

* Sử dụng file [.editorconfig](http://editorconfig.org/) cái mà giúp các developer xác định và bảo trì các coding style thích hợp giữa các editor và IDE khác nhau trên dự án.

    _Tại sao:_
    > Dự án EditorConfig bao gồm một file định dạng cho xác định các codeing style và một tập hợp các plugin của editor để cho phép các editor để đọc file định dạng tuân theo các style được xác định. Các file EditorConfig dễ dàng có thê đọc được và chúng làm việc thú vị với các phiên bản của hệ thống kiểm soát.

*  Editor của bạn thông báo với bạn về các lỗi của code style. Sử dụng [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier) và [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier) với cấu hình ESLint đã có. [Đọc thêm...](https://github.com/prettier/eslint-config-prettier#installation)

* Xem xét sử dụng các Git hook.

    _Tại sao:_
    > Git hooks tăng đáng kể năng suất của các nhà phát triển sản phẩm. Các thay đổi được thực hiện, commit và push để môi trường staging hoặc production mà không phá vớ cấu trúc đã xây dựng. [đọc thêm...](http://githooks.com/)

* Dùng Prettier với hook trước khi commit.

    _Tại sao:_
    > Trong khi `prettier` bản thân nó có thể rất mạnh mẽ, nó không phải rất hiệu quả để chạy đơn giản giống như một nhiệm vụ đơn lẻ của npm mỗi khi định dạng code. Đây là `lint-staged` (và `husky`) để vào chạy. Đọc thêm nhiều cấu hình `lint-staged` [đây](https://github.com/okonet/lint-staged#configuration) và trên cấu hình `husky` [đây](https://github.com/typicode/husky).


<a name="logging"></a>
## 8. Logging

![Logging](/images/logging.png)

* Avoid client-side console logs in production

    _Why:_
    > Even though your build process can (should) get rid of them, make sure that your code style checker warns you about leftover console logs.

* Produce readable production logging. Ideally use logging libraries to be used in production mode (such as [winston](https://github.com/winstonjs/winston) or
[node-bunyan](https://github.com/trentm/node-bunyan)).

    _Why:_
    > It makes your troubleshooting less unpleasant with colorization, timestamps, log to a file in addition to the console or even logging to a file that rotates daily. [read more...](https://blog.risingstack.com/node-js-logging-tutorial/)


<a name="api"></a>
## 9. API
<a name="api-design"></a>

![API](/images/api.png)

### 9.1 API design

_Why:_
> Because we try to enforce development of sanely constructed RESTful interfaces, which team members and clients can consume simply and consistently.  

_Why:_
> Lack of consistency and simplicity can massively increase integration and maintenance costs. Which is why `API design` is included in this document.


* We mostly follow resource-oriented design. It has three main factors: resources, collection, and URLs.
    * A resource has data, gets nested, and there are methods that operate against it.
    * A group of resources is called a collection.
    * URL identifies the online location of resource or collection.
    
    _Why:_
    > This is a very well-known design to developers (your main API consumers). Apart from readability and ease of use, it allows us to write generic libraries and connectors without even knowing what the API is about.

* use kebab-case for URLs.
* use camelCase for parameters in the query string or resource fields.
* use plural kebab-case for resource names in URLs.

* Always use a plural nouns for naming a url pointing to a collection: `/users`.

    _Why:_
    > Basically, it reads better and keeps URLs consistent. [read more...](https://apigee.com/about/blog/technology/restful-api-design-plural-nouns-and-concrete-names)

* In the source code convert plurals to variables and properties with a List suffix.

    _Why_:
    > Plural is nice in the URL but in the source code, it’s just too subtle and error-prone.

* Always use a singular concept that starts with a collection and ends to an identifier:

    ```
    /students/245743
    /airports/kjfk
    ```
* Avoid URLs like this: 
    ```
    GET /blogs/:blogId/posts/:postId/summary
    ```

    _Why:_
    > This is not pointing to a resource but to a property instead. You can pass the property as a parameter to trim your response.

* Keep verbs out of your resource URLs.

    _Why:_
    > Because if you use a verb for each resource operation you soon will have a huge list of URLs and no consistent pattern which makes it difficult for developers to learn. Plus we use verbs for something else.

* Use verbs for non-resources. In this case, your API doesn't return any resources. Instead, you execute an operation and return the result. These **are not** CRUD (create, retrieve, update, and delete) operations:

    ```
    /translate?text=Hallo
    ```

    _Why:_
    > Because for CRUD we use HTTP methods on `resource` or `collection` URLs. The verbs we were talking about are actually `Controllers`. You usually don't develop many of these. [read more...](https://byrondover.github.io/post/restful-api-guidelines/#controller)

* The request body or response type is JSON then please follow `camelCase` for `JSON` property names to maintain the consistency.
    
    _Why:_
    > This is a JavaScript project guideline, where the programming language for generating and parsing JSON is assumed to be JavaScript.

* Even though a resource is a singular concept that is similar to an object instance or database record, you should not use your `table_name` for a resource name and `column_name` resource property.

    _Why:_
    > Because your intention is to expose Resources, not your database schema details.

* Again, only use nouns in your URL when naming your resources and don’t try to explain their functionality.

    _Why:_
    > Only use nouns in your resource URLs, avoid endpoints like `/addNewUser` or `/updateUser` .  Also avoid sending resource operations as a parameter.

* Explain the CRUD functionalities using HTTP methods:

    _How:_
    > `GET`: To retrieve a representation of a resource.
    
    > `POST`: To create new resources and sub-resources.
   
    > `PUT`: To update existing resources.
    
    > `PATCH`: To update existing resources. It only updates the fields that were supplied, leaving the others alone.
    
    > `DELETE`: To delete existing resources.


* For nested resources, use the relation between them in the URL. For instance, using `id` to relate an employee to a company.

    _Why:_
    > This is a natural way to make resources explorable.

    _How:_

    > `GET      /schools/2/students ` , should get the list of all students from school 2.

    > `GET      /schools/2/students/31` , should get the details of student 31, which belongs to school 2.

    > `DELETE   /schools/2/students/31` , should delete student 31, which belongs to school 2.

    > `PUT      /schools/2/students/31` , should update info of student 31, Use PUT on resource-URL only, not collection.

    > `POST     /schools` , should create a new school and return the details of the new school created. Use POST on collection-URLs.

* Use a simple ordinal number for a version with a `v` prefix (v1, v2). Move it all the way to the left in the URL so that it has the highest scope:
    ```
    http://api.domain.com/v1/schools/3/students 
    ```

    _Why:_
    > When your APIs are public for other third parties, upgrading the APIs with some breaking change would also lead to breaking the existing products or services using your APIs. Using versions in your URL can prevent that from happening. [read more...](https://apigee.com/about/blog/technology/restful-api-design-tips-versioning)



* Response messages must be self-descriptive. A good error message response might look something like this:
    ```json
    {
        "code": 1234,
        "message" : "Something bad happened",
        "description" : "More details"
    }
    ```
    or for validation errors:
    ```json
    {
        "code" : 2314,
        "message" : "Validation Failed",
        "errors" : [
            {
                "code" : 1233,
                "field" : "email",
                "message" : "Invalid email"
            },
            {
                "code" : 1234,
                "field" : "password",
                "message" : "No password provided"
            }
          ]
    }
    ```

    _Why:_
    > developers depend on well-designed errors at the critical times when they are troubleshooting and resolving issues after the applications they've built using your APIs are in the hands of their users.


    _Note: Keep security exception messages as generic as possible. For instance, Instead of saying ‘incorrect password’, you can reply back saying ‘invalid username or password’ so that we don’t unknowingly inform user that username was indeed correct and only the password was incorrect._

* Use these status codes to send with your response to describe whether **everything worked**,
The **client app did something wrong** or The **API did something wrong**.
    
    _Which ones:_
    > `200 OK` response represents success for `GET`, `PUT` or `POST` requests.

    > `201 Created` for when a new instance is created. Creating a new instance, using `POST` method returns `201` status code.

    > `204 No Content` response represents success but there is no content to be sent in the response. Use it when `DELETE` operation succeeds.

    > `304 Not Modified` response is to minimize information transfer when the recipient already has cached representations.

    > `400 Bad Request` for when the request was not processed, as the server could not understand what the client is asking for.

    > `401 Unauthorized` for when the request lacks valid credentials and it should re-request with the required credentials.

    > `403 Forbidden` means the server understood the request but refuses to authorize it.

    > `404 Not Found` indicates that the requested resource was not found. 

    > `500 Internal Server Error` indicates that the request is valid, but the server could not fulfill it due to some unexpected condition.

    _Why:_
    > Most API providers use a small subset HTTP status codes. For example, the Google GData API uses only 10 status codes, Netflix uses 9, and Digg, only 8. Of course, these responses contain a body with additional information.There are over 70 HTTP status codes. However, most developers don't have all 70 memorized. So if you choose status codes that are not very common you will force application developers away from building their apps and over to wikipedia to figure out what you're trying to tell them. [read more...](https://apigee.com/about/blog/technology/restful-api-design-what-about-errors)


* Provide total numbers of resources in your response.
* Accept `limit` and `offset` parameters.

* The amount of data the resource exposes should also be taken into account. The API consumer doesn't always need the full representation of a resource. Use a fields query parameter that takes a comma separated list of fields to include:
    ```
    GET /student?fields=id,name,age,class
    ```
* Pagination, filtering, and sorting don’t need to be supported from start for all resources. Document those resources that offer filtering and sorting.

<a name="api-security"></a>
### 9.2 API security
These are some basic security best practices:

* Don't use basic authentication unless over a secure connection (HTTPS). Authentication tokens must not be transmitted in the URL: `GET /users/123?token=asdf....`

    _Why:_
    > Because Token, or user ID and password are passed over the network as clear text (it is base64 encoded, but base64 is a reversible encoding), the basic authentication scheme is not secure. [read more...](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

* Tokens must be transmitted using the Authorization header on every request: `Authorization: Bearer xxxxxx, Extra yyyyy`.

* Authorization Code should be short-lived.

* Reject any non-TLS requests by not responding to any HTTP request to avoid any insecure data exchange. Respond to HTTP requests by `403 Forbidden`.

* Consider using Rate Limiting.

    _Why:_
    > To protect your APIs from bot threats that call your API thousands of times per hour. You should consider implementing rate limit early on.

* Setting HTTP headers appropriately can help to lock down and secure your web application. [read more...](https://github.com/helmetjs/helmet)

* Your API should convert the received data to their canonical form or reject them. Return 400 Bad Request with details about any errors from bad or missing data.

* All the data exchanged with the REST API must be validated by the API.

* Serialize your JSON.

    _Why:_
    > A key concern with JSON encoders is preventing arbitrary JavaScript remote code execution within the browser... or, if you're using node.js, on the server. It's vital that you use a proper JSON serializer to encode user-supplied data properly to prevent the execution of user-supplied input on the browser.

* Validate the content-type and mostly use `application/*json` (Content-Type header).
    
    _Why:_
    > For instance, accepting the `application/x-www-form-urlencoded` mime type allows the attacker to create a form and trigger a simple POST request. The server should never assume the Content-Type. A lack of Content-Type header or an unexpected Content-Type header should result in the server rejecting the content with a `4XX` response.

* Check the API Security Checklist Project. [read more...](https://github.com/shieldfy/API-Security-Checklist)

<a name="api-documentation"></a>
### 9.3 API documentation
* Fill the `API Reference` section in [README.md template](./README.sample.md) for API.
* Describe API authentication methods with a code sample.
* Explaining The URL Structure (path only, no root URL) including The request type (Method).

For each endpoint explain:
* URL Params If URL Params exist, specify them in accordance with name mentioned in URL section:

    ```
    Required: id=[integer]
    Optional: photo_id=[alphanumeric]
    ```

* If the request type is POST, provide working examples. URL Params rules apply here too. Separate the section into Optional and Required.

* Success Response, What should be the status code and is there any return data? This is useful when people need to know what their callbacks should expect:

    ```
    Code: 200
    Content: { id : 12 }
    ```

* Error Response, Most endpoints have many ways to fail. From unauthorized access to wrongful parameters etc. All of those should be listed here. It might seem repetitive, but it helps prevent assumptions from being made. For example
    ```json
    {
        "code": 403,
        "message" : "Authentication failed",
        "description" : "Invalid username or password"
    }   
    ```


* Use API design tools, There are lots of open source tools for good documentation such as [API Blueprint](https://apiblueprint.org/) and [Swagger](https://swagger.io/).

<a name="licensing"></a>
## 10. Licensing
![Licensing](/images/licensing.png)

Make sure you use resources that you have the rights to use. If you use libraries, remember to look for MIT, Apache or BSD but if you modify them, then take a look at the license details. Copyrighted images and videos may cause legal problems.


---
Sources:
[RisingStack Engineering](https://blog.risingstack.com/),
[Mozilla Developer Network](https://developer.mozilla.org/),
[Heroku Dev Center](https://devcenter.heroku.com),
[Airbnb/javascript](https://github.com/airbnb/javascript),
[Atlassian Git tutorials](https://www.atlassian.com/git/tutorials),
[Apigee](https://apigee.com/about/blog),
[Wishtack](https://blog.wishtack.com)

Icons by [icons8](https://icons8.com/)

