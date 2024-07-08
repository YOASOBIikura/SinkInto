---
title: WebUtilities And WordList
date: 2024-07-03 20:53:53
categories:
  - ethers
tags:
  - WebUtilities And WordList
---

`ethers.utils.fetchJson(urlOrConnectionInfo, json, processFun) => any`
通过请求指定的url或是Connection获取json内容并进行解析

`ethers.utils.poll(pollFunc, option) => any`
反复调用指定的函数，知道返回一个非undifined的值为止

`wordlist.local => string`
返回词表的local设置

`wordlist.getWord(index) => string`
根据索引返回词表中的单词

`wordlist.getWordIndex(word) => string`
查找指定单词的词表索引

`wordlist.split(mnemonic) => array`
返回拆分为单个单词的助记词列表

`wordlist.join(words) => string`
将每个单词拼接在一起返回助记词

`wordlist.check(wordlist) => string`
检查所有单词的双向映射是否正确，并返回列表的哈希值

`wordlist.register(wordlist, name) => void`
通过词表的列表注册一个wordlist

