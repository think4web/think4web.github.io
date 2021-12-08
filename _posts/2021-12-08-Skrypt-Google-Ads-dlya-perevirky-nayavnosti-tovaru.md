---
published: true
layout: post
title: Скрипт Google Ads для перевірки наявності товару
tags: google-ads script 
discription: Скрипт Google Ads зупиняє або відновлює роботу оголошень в залежності від наявності товару на сторінці.
---

Скрипт для перевірки наявності товару в магазині. Шукає ключову фразу на цільовій сторінці оголошення і якщо знаходить, призупиняє оголошення. Або поновлює його роботу якщо не знаходить. Необхідно використовувати скрипт обережно бо він відновить роботу усіх призупинених оголошень якщо вони ведуть на сторінки з товаром.

```js
/************************************
* Quick Url In Stock/Out Of Stock Checker
* Version 1.0
* Created By: Russ Savage
* FreeAdWordsScripts.com
***********************************/
var STRIP_QUERY_STRING = true;
var WRAPPED_URLS = true;
var OUT_OF_STOCK_TEXT = 'The Text That Identifies An Out Of Stock Item Goes Here';
var URL_TO_TEST = 'Your Url To Check Goes Here';

function main() {
  var urlToTest = URL_TO_TEST;
  urlToTest = cleanUrl(urlToTest);
  var htmlCode = UrlFetchApp.fetch(urlToTest).getContentText();
  if(htmlCode.indexOf(OUT_OF_STOCK_TEXT) >= 0) {
    Logger.log('The item is out of stock.');
  } else {
    Logger.log('The item is in stock.');
  }
}

function cleanUrl(url) {
  if(WRAPPED_URLS) {
    url = url.substr(url.lastIndexOf('http'));
    if(decodeURIComponent(url) !== url) {
      url = decodeURIComponent(url);
    }
  }
  if(STRIP_QUERY_STRING) {
    if(url.indexOf('?')>=0) {
      url = url.split('?')[0];
    }
  }
  if(url.indexOf('{') >= 0) {
    //Let's remove the value track parameters
    url = url.replace(/\{[0-9a-zA-Z]+\}/g,'');
  }
  return url;
}
```

Скрипт брав [звідси](http://www.freeadwordsscripts.com/2013/10/disable-ads-and-keywords-for-out-of.html)
