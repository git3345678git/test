---
layout: post
title: "Using PHAR deserialization to deploy a custom gadget chain"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Insecure_Deserialization
author: ""
tags: ['PortSwigger_Lab', 'Insecure_Deserialization']
---





# Using PHAR deserialization to deploy a custom gadget chain

```

This lab does not explicitly use [deserialization](https://portswigger.net/web-security/deserialization). However, if you combine `PHAR` deserialization with other advanced hacking techniques, you can still achieve remote code execution via a custom gadget chain.

To solve the lab, delete the `morale.txt` file from Carlos's home directory.

You can log in to your own account using the following credentials: `wiener:peter`

#### Learning path

If you're following our suggested [learning path](https://portswigger.net/web-security/learning-path), please note that this lab requires some understanding of topics that we haven't covered yet. Don't worry if you get stuck; try coming back later once you've developed your knowledge further.


本實驗室未明確使用 [反序列化](https://portswigger.net/web-security/deserialization)。 但是，如果您將“PHAR”反序列化與其他高級黑客技術結合起來，您仍然可以通過自定義小工具鏈實現遠程代碼執行。

要解決實驗室問題，請從 Carlos 的主目錄中刪除 `morale.txt` 文件。

您可以使用以下憑據登錄到您自己的帳戶：`wiener:peter`

####學習路徑

如果您遵循我們建議的 [學習路徑](https://portswigger.net/web-security/learning-path)，請注意，此實驗需要對我們尚未涵蓋的主題有所了解。 如果你被卡住了，不要擔心； 一旦你進一步發展了你的知識，試著稍後再回來。



```