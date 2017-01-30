---
layout: 'post'
title: 'Галерея с сортировкой на Ruby on Rails'
date: '2015-10-15'
---

Идея состоит в том, чтобы загружать картинки, а после менять их порядок отображения перетаскиванием.
Предположим, наше приложение уже создано командой `rails new`. Что понадобится дальше? Естественно, модель для наших картинок, и контроллер, который обеспечит связь между пользователем и нашей системой. Однако это чуть позже.

Я не буду показывать подробные логи создания контролера, модели, миграций. Хочется просто отразить суть происходящего.

Создадим модель командой `rails g model Picture`. Сделаем `rake db:migrate`, чтобы создать таблицу pictures, после добавим еще одно изменение к базе.
Для этого прогоним следующую миграцию:
`rails g migration add_pritority_to_pictures priority:integer`

После ее применения к нашей таблице добавится еще одна колонка — priority. Она нам понадобится при сортировке.
Следующим шагом позаботимся о корректной работе `paperclip`. Это все написано в документации, но кратко обрисую. Генерируем еще одну миграцию со следующим содержанием и применяем.

{% highlight ruby %}
class AddFileToPicture < ActiveRecord::Migration
  def change
    add_attachment :pictures, :file
  end
end
{% endhighlight %}

Она внесет необходимые изменения для корректной работы гема. Можно зайти в schema.rb файл и посмотреть, как изменилась таблица.

С базой, по большому счету, мы покончили. Теперь перейдем к изменению файла модели. Все, что нам нужно сделать, это сказать, что у нашей картинки есть прикрепленный файл. Этим тоже займется paperclip.

{% highlight ruby %}
class Picture < ActiveRecord::Base
  has_attached_file :file, styles: { medium: "300x300>" }
  validates :file, :title, presence: true
end
{% endhighlight %}

На этом всё. Перед тем, как перейти к контроллеру, не забудем про наш роутинг.

{% highlight ruby %}
Rails.application.routes.draw do
  resources :pictures, only: [:index, :create, :new] do
    put "sort", on: :collection
  end

  root 'pictures#index'
end
{% endhighlight %}

Мы здесь сообщаем: Rails, хочу, чтобы ты мне сделал 3 базовых роута, а также роут, который будет обрабатывать PUT запрос по адресу `pictures/sort`.
Итак, наш контроллер.

{% highlight ruby %}
class PicturesController < ApplicationController
  def index
    @pictures = Picture.order(:priority)
  end

  def new
    @picture = Picture.new
  end

  def create
    @picture = Picture.new(picture_params)
    if @picture.save
      redirect_to root_path
    else
      render :new
    end
  end

  def sort
    JSON.parse(params[:data]).each do |el|
      Picture.find(el["id"]).update_attributes(priority: el["priority"])
    end
    render nothing: true
  end

  private

  def picture_params
    params.require(:picture).permit(:file, :title)
  end
end
{% endhighlight %}

На что обращаем внимание.
Первое, это на то, что в нашем index экшне мы получаем картинки, отсортированные по приоритету. Все просто.
Второе — на наш экшн sort. Здесь немного сложнее.
Мы предполагаем, что придут какие-то данные в формате JSON, содержащие информацию в виде ключ-значение.
Можно заметить, что мы получаем информацию об id нашей картинки, и ее порядковый номер на странице.
Пробегаемся по всем значениям и обновляем порядковые номера найденных по id картинок.
В конце используем render nothing:true, т.к. мы хотим просто обрабатывать ajax запрос и ничего не возвращать на клиентскую сторону.
Теперь перейдем к нашей клиентской части.
index.html.erb файл.

{% highlight erb %}
<div class="container">
  <h3><%= link_to "Add picture", new_picture_path %></h3>
  <% @pictures.each do |picture| %>
    <div class = "picture" data-id = "<%= picture.id %>">
      <%= image_tag(picture.file.url(:medium)) %>
    </div>
  <% end %>
</div>
{% endhighlight %}

Здесь все прозрачно, кроме одного момента: мы добавляем data-id атрибут к нашей картинке. Сейчас мы увидим, для чего это нужно.
Создадим файл sortable.js со следующим содержанием.

{% highlight javascript %}
$(function(){

  var updated_order = function(){
    var data_arr = [];
    $.each($(".picture"), function(index){
      $(this).attr("priority", index);
      data_arr.push({id: $(this).attr("data-id"), priority: index })
    });
    return data_arr;
  }

  $(".container").sortable({
    cursor: "move",
    stop: function(event, ui){
      var data = updated_order();
      $.ajax({
        url: "pictures/sort",
        type: "PUT",
        data: {"data": JSON.stringify(data)},
        success: function(msg){
          console.log("Sorted!");
        }
      });
    }
  });
});
{% endhighlight %}

Код не самый красивый, но свою функцию он сейчас выполняет. Как говорится: сначала сделай, чтобы
работало, а потом, чтобы работало красиво.

Используем jQuery-ui для реализации перетаскивания. Для этого вызываем .sortable() на нашем контейнере с картинками. После того, как какая-нибудь картинка будет перемещена, будет вызван колбэк, который отправит ajax-запрос на наш сервер, передав ему нужные данные.

Эти данные собирает функция updated_order. Мы пробегаемся по всем элементам и добавляем в массив хэшики с id, прочитанным из data-id атрибута, а также с порядковым номером элемента.
Вот, собственно, и всё.
