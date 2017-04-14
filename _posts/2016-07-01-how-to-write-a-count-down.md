---
layout: post
title:  "How to write a count down"
categories: javascript
tags: issues
author: Roc
---

* content
{:toc}

There are 2 ways for count down in javascript

* setTimeout
* setInterval

But them are not accurate for long time count down and sometimes not works well if the user update their PC time.

Below is the accurate implementation which from author `Gaohaoyang` the guy works at T-mall.

## Code

```js
/**
 * @param  {Number} endTime
 * @param  {Number} deviceTime
 * @param  {Number} serverTime
 * @return {Object}
 */
let getRemainTime = (endTime, deviceTime, serverTime) => {
    let t = endTime - Date.parse(new Date()) - serverTime + deviceTime
    let seconds = Math.floor((t / 1000) % 60)
    let minutes = Math.floor((t / 1000 / 60) % 60)
    let hours = Math.floor((t / (1000 * 60 * 60)) % 24)
    let days = Math.floor(t / (1000 * 60 * 60 * 24))
    return {
        'total': t,
        'days': days,
        'hours': hours,
        'minutes': minutes,
        'seconds': seconds
    }
}
```

```js
getServerTime((serverTime) => {

    let intervalTimer = setInterval(() => {

        let remainTime = getRemainTime(endTime, deviceTime, serverTime)

        if (remainTime.total <= 7200000 && remainTime.total > 0) {
            // do something

        } else if (remainTime.total <= 0) {
            clearInterval(intervalTimer);
            // do something
        }
    }, 1000)
})
```
