---
layout: page
title: Контактная информация
description: Контакты компании Prefix в Шымкенте
weight : 100
sitemap:
  priority: 0.7
  changefreq: 'monthly'
---


<info@prefix.kz>

+7 701 783-87-45 Александр

+7 702 659-00-54 Станислав

160005, проезд Мамин-Сибиряк, 21 <br>
Шымкент, Казахстан


## Отправить сообщение

<form action="//formspree.io/info@prefix.kz" method="POST" class="form-dark">

    <div class="form-group">
        <label for="InputName">Имя</label>
        <input name="name" type="name" class="form-control" id="InputName" placeholder="Имя">
    </div>

    <div class="form-group">
        <label for="InputEmail">Эл. почта</label>
        <input name="_replyto" type="email" class="form-control" id="InputEmail" placeholder="email@example.com">
    </div>

    <div class="form-group">
        <label for="TextArea">Сообщение</label>
        <textarea name="message" class="form-control" rows="3"></textarea>
    </div>

    <input type="hidden" name="_next" value="{{ site.baseurl }}thanks/" />
    <input type="text" name="_gotcha" class="hidden" />

    <button type="submit" class="button">Отправить</button>

</form>

