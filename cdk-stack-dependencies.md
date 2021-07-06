+++
title = "Stack Dependencies"
date = "2021-05-09"
tags = ["CDK"]
+++

```
const aStack = new AStack(this, 'AStack') 
const bStack = new BStack(this, 'BStack')
aStack.addDependency(bStack)
```

てなってて

```
cdk deploy AStack
```

するとBStackが先にデプロイされてからAStackがデプロイされる。

```
cdk destroy BStack
```

とするとAStackが先に削除されてからBStackが削除される。

ただこの依存性はcdkを使った場合にだけ機能するものでAWSコンソールのCloudFormationの画面ではBStackだけを削除するといったことができる。

[@aws-cdk/core module · AWS CDK](https://docs.aws.amazon.com/cdk/api/latest/docs/core-readme.html)