# git工作流

## 个人理解

公司日常开发中，一般会有两类分支，分别是**主分支**和**辅分支**。主分支包括master(能用于生产环境部署的稳定无错分支)和develop(最新开发成果的分支)。而辅分支主要开发、提测某个功能或是解决某个bug而开的一个临时性的分支。   


## 转载[GIT工作流](http://fe.firstshare.cn/#/articles/574d4a7eb56346d5156cfa9c)

git工作流中定义了主分支和辅分支两类分支。

### 主分支

主分支是核心分支，所有辅分支的代码最终都会合并到主分支上，主分支不能被删除。主分支分为master分支和develop分支。

#### master分支

master 分支是随时可在生产环境中部署的代码（Production Ready state）。
master分支不能有提测代码和开发中代码。

代码合并到master的时机是：本版本内测完毕，要发全网的时候，从develop合入（此时的develop必须是干净的），并用master发布全网，最后打上product tag（比如v5.3），方便代码跟踪。

线上bug的修复，要基于master拉出hotfix-*分支，bug修复完毕，部署hotfix-*在测试环境测试，测试通过后，将hotfix-*合并回master，并用master发布线上，最后打上product tag（比如v5.3.1），并将hotfix-*删除（删除之前要先合并到develop）。

合并hotfix-*代码到master，请在gitlab或命令行发起merge request，并通知有master权限的同学合并代码。

#### develop分支

develop 分支是保存当前最新开发成果的代码，也是代码最齐全的一个分支，属于开发时主分支。

理论上任何人并不需要在此分支上直接写代码，而是基于develop拉出自己的feature-*分支，开发自己的功能。当自己的功能开发完毕之后，并经过简单的单元测试，就可以合并回develop，并将feature-*删除。这样做的目的是避免别人尚在开发中的功能给测试带来干扰。

当某个feature-*功能都开发完毕，需要独立发测试（如果不需要发独立测试，则合并到develop等待集成测试），就需要基于这个feature-*打上test tag（比如tag-5.3.1，具体描述见提测tag节点），QAD则使用这个test tag部署测试环境测试独立功能。独立测试期间都在本feature-*上修复bug，当功能已测试通过，则将此feature-*合并回develop，并删除之。

当某个版本的所有功能都已开发完毕，需要进行集成测试了，则先将所有feature-*都合并回develop，再给develop打上test tag，QAD使用这个test tag部署测试环境集成测试。集成测试期间都在develop修复bug（此时不能再在develop开发本版本以外的功能，如果需要开发本版本以外的功能，就要开一个新的feature-*分支开发，保证develop代码是干净的）。

当集成测试全部测试完毕并可以部署生产环境时（包括内测完毕），将develop合并到master（必须保证develop是没有被污染的），用master发布线上，并打上product tag，比如v5.3。

修复的线上bug hotfix分支，在发布之后，也要合并到develop。

### 辅分支

辅分支是为了开发某个功能、提测、解决某个bug而开的分支。
辅分支包括：

用于开发新功能的feature分支
用于提测的tag（仅供QAD用）
用于线上bug修复的hotfix分支
feature分支

开发新功能的分支，feature分支的代码最终要合并回develop，或者干脆抛弃（比如试验性或效果不好的代码变更）。
使用规范：

基于develop拉出
代码必须合并回develop
分支命名：feature-*或feature/*（*是所要开发的功能名字，比如feature/new-emoji）

#### 提测tag

独立测试或集成测试，都需要打一个test tag，提供给QAD同学部署测试环境。
使用规范：

tag命名：tag-版本号，比如tag_5.3.1，最后一位小版本“1”表示第一次提测，每次提测累积最后一个数字
独立测试的tag基于feature-*
集成测试的tag基于develop

#### hotfix分支

修复线上bug的分支，修复完毕并上线后，删除之。
使用规范：

基于master拉出
代码必须合并回master，并同时合并到develop
分支命名：hotfix-*（*可以是bug系统的bug编号，比如hotfix-#123）

### 结束语

git工作流几个原则：

master主分支只跟踪和部署生产环境代码，不能直接写代码，也不能把未测试代码合并进来
develop主分支不能删除，是开发时主脉络，理论上也不应该在develop直接写代码，拉出feature-*分支开发，互不干扰
feature分支代码必须合并回develop
tag分test tag和product tag，test tag提供给QAD提测，product tag留存和跟踪线上历史版本代码
hotfix分支必须合并到master和develop