# Python 技巧

* `s.find(sub, start, end)`
* `s.strip(chars)`
* `s.split(' ')` // separator	Optional. Specifies the separator to use when splitting the string. By default any whitespace is a separator
`'  '.split(' ')` => `['', '', '']` 两个空格，分出3个空串
* `''.join(list)`

# Term

* sub string
* sub sequence
* prefix
* subfix
* palindrome / palindromic string

# string matching
## KMP

# string hashing

A simple implementation:
``` C
hash('abc') = ((a * 233) + b) * 233 + c
```

## Hash Application
### string matching

求出模式串的哈希值后，求出文本串每个长度为模式串长度的子串的哈希值，分别与模式串的哈希值比较即可。
> 为什么要这么搞？？？我不会用这样的方法。

### 允许 k 次失配的字符串匹配

### 最长公共子字符串

一般是问 2 个字符串的最长公共子字符串，求解一般用DP。但如果问 m 个字符串的最长公共子字符串，就要用hash暴力解了。

指定一个数字 k 。利用`check(k)`来判断，是否有长度为 k 的 common substring。`check(k)` 就是对一个string所有长度为 k 的字串求 hash 。最后 m 个 hash 集合求交集。

而 k 可以二分查找。





 