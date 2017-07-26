---
layout: 'post'
title: 'Чистота должна быть не только в коде, но и в названиях коммитов'
date: '2016-07-18'
---

![VCS](https://proglib.io/wp-content/uploads/2017/05/c5131475jw1esypv5j0h2g20ng0b40tz.gif)

Пользуясь всего лишь несколькими правилами, можно кардинально изменить воспрятие истории репозитория.

### Названия коммитов с большой буквы в повелительном наклонении

Серьезно, не понимаю, почему люди об этом забывают.

{% highlight text %}
e215f4sd2b49 Consolidated classes;
esdfaga24b49 added new markdown for the…;
3r5f4ab49ag9 fixed specs;
{% endhighlight %}

Многие, может быть, и привыкли к такому стилю, но читабельность такой истории стремится к нулю.

{% highlight text %}
e215f4sd2b49 Consolidate classes;
esdfaga24b49 Add new markdown;
3r5f4ab49ag9 Fix specs;
{% endhighlight %}

Это уже лучше, правда?

### Лаконичность

Если что-то не умещается в 60 символов, может, стоит вынести это в описание, но не в название коммита? Все просто.
Так что если перед вашими глазами длинная колбаса из текста, просто стоит задуматься.

Хочется закончить картинкой, которую я позаимствовал из твиттера [CTO GitLab](https://twitter.com/dzaporozhets).

![Commit messages](https://pbs.twimg.com/media/DBPQbTrXkAA4v-H.jpg:large)
