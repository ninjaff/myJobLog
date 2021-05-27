---
title: Highlight Test
draft: true
tags:
  - tag1
  - tag2
  - tag3
categories:
  - - Diary
    - PlayStation
  - - Diary
    - Games
  - - Tech
archives: '2021'
abbrlink: 39796
date: 2021-05-07 07:37:23
updated: 2021-05-21 09:50:23
---

### highlight
#### yaml
```yaml
prismjs:
  enable: false
  preprocess: true
  line_number: false
  tab_replace: ''
```
#### bash
```bash
cd themes
git clone https://github.com/tanksuzuki/angels-ladder
hugo server -t angels-ladder -D -w
```
#### go
```go
func main(){
    var a int
    a = 1
    b := 2
    var str string
    a = "Hello"
    fmt.Println(str, a, b, a+b)
}
```
#### js
```js
function test(){
    var a=1,b=2;
    var str='Hello';
    console.log(str, a, b, a+b);
}
```
#### css
```css
body {
  background-color: #fff;
  font-family: "Hiragino Kaku Gothic ProN", Meiryo, sans-serif;
  font-size: 14px;
  color: #666;
  line-height: 1.6em;
  letter-spacing: 0.5px;
}
```