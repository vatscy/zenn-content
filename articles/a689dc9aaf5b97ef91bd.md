---
title: "Azure DevOpsでPull Request時に自動で様々なチェックを実施する"
emoji: "🕵️‍♀️"
type: "tech"
topics: ["azuredevops","azure"]
published: false
---

Azure DevOps (正確には Azure Repos) で Pull Request を活用する際に、ブランチ単位で下記のようなことを設定できます。

- PR作成と同時に単体テストを実行し、失敗した場合はマージできないようにする
- 最低N人のコードレビューを受ける
- 特定のレビュアーを必ず含めるようにし、通知を送る
- レビュアーが残したコメントに応答しているか確認する
- 関連タスクが全て完了しているか確認する

これらを重要なブランチに設定することで品質担保に繋がります。この記事ではそれらの設定方法を解説します。

# ブランチポリシーの設定

ブランチ毎にポリシーを設定できます。ポリシー設定済みのブランチにはバッジがつきます。

**Repos > Branches**
![](https://storage.googleapis.com/zenn-user-upload/qaobqzqk2rzebpsm8dy2pss8lw1k)

ブランチポリシーは、**そのブランチに対する Pull Request** が作成された際に適用されます。また、ブランチポリシーが設定されたブランチへ直接Pushすることは禁止されます。

ブランチポリシーを設定するには、各ブランチの設定(三点リーダ)から「Branch policies」に遷移します。既にポリシー設定済の場合はバッジを押下することでも遷移できます。

![](https://storage.googleapis.com/zenn-user-upload/roajuzt6xuys8meorun1fovekabk)

## 必須レビュアー数

「最低N人のコードレビューを受ける」という制約を設けることができます。

![](https://storage.googleapis.com/zenn-user-upload/lid7t5rai2i6odelvf5szmf6219k)

これは実際に私のプロダクトで設定しているものです。

この場合、最低2人がコードレビューをして承認をしないとマージができません。ただし、「**Allow requestors to approve their own changes**」をONにしているため、Pull Request を提出した本人が自分で承認ボタンを押してもカウントされます。これは Pull Request を作成した本人にも再度コードをチェックしてもらうために設定しています。そのため、実質的に本人以外に最低1人承認してもらえばOKです。

また、「**When new changes are pushed: Reset all approval votes (does not reset votes to reject or wait)**」を選択することで、コードレビュー承認後に追加でコミットした場合に承認ステータスをリセットして再度レビューしてもらうことができます。

## 固定レビュアー

決まった方(リーダーなど)を必ずレビュアーに含めたい場合はブランチポリシーで設定しておくことができます。

![](https://storage.googleapis.com/zenn-user-upload/6f7cxjzcugzwmr6j6o9efrlz4nia)

**Required** に設定するとそのレビュアーの承認が必須、 **Optional** に設定するとそのレビュアーの承認ステータスに依らずマージできます。

## レビューコメントへの応答のチェック

レビュアーのコメントに対してPR作成者が応答しているかどうかをチェック項目にすることができます。

![](https://storage.googleapis.com/zenn-user-upload/ba6yajrpnofgtk8shbtdz3bu0az6)

**Required** に設定すると、PR作成者はコメントを確認して最終的にコメントのステータスを「Resolved」「Won't fix」「Closed」のいずれかに変更する必要があります。**Optional** に設定した場合はその必要はありませんが、警告が表示されるようになるためコメントを見逃すリスクが減ります。

## PR作成時にパイプラインを実行する

「PR作成と同時に単体テストを実行し、失敗した場合はマージできないようにする」といった制約を設けることができます。

![](https://storage.googleapis.com/zenn-user-upload/heeyedcdbinwd1c8wwaet6glmird)

**Required** に設定すると、パイプラインの正常終了がマージの条件になります。**Optional** の場合はパイプラインは実行されますが、終了ステータスに依らずマージできます。

また、「Build expiration」では、パイプライン実行後にブランチに新しいコミットが増えた場合の挙動を設定できます。

## 関連する Work Item の完了チェック

PRに紐付けられた Work Item が全て Closed になっているかのチェックができます。

![](https://storage.googleapis.com/zenn-user-upload/io3117win4aagyxg0blidzodg10q)

**Required** に設定すると、全て Closed になるまでマージができなくなります。

# 設定結果

無事、ブランチポリシーが設定されているブランチに対するPRを作成すると、下記のように自動でチェックが実施されるようになりました🎉

![](https://storage.googleapis.com/zenn-user-upload/0btwvsb6akjnlictelmwv28dsc4r)