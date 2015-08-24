Регионы предоставляют методы для согласованного управления, показа и уничтожения представлений
в ваших приложениях и макетах. Они используют jQuery-селекторы, чтобы показать ваши представления
в правильном месте.

С помощью класса `LayoutView` вы можете создавать вложенные регионы.

## Содержание

* [Определение регионов приложения](#defining-an-application-region)
  * [Типы конфигурирования регионов](#region-configuration-types)
* [Инициализация региона с помощью `el`](#initialize-a-region-with-an-el)
* [Основное применение](#basic-use)
  * [Отображение представления](#showing-a-view)
  * [Проверка отображено ли представление в регионе](#checking-whether-a-region-is-showing-a-view)
  * [Сброс региона](#reset-a-region)
  * [Настройка способа добавления `el` представления в DOM](#set-how-views-el-is-attached)
  * [Добавление существующего представления](#attach-existing-view)
    * [Установка `currentView` при инициализации](#set-currentview-on-initialization)
    * [Вызов `attachView` в регионе](#call-attachview-on-region)
* [События и коллбэки региона](#region-events-and-callbacks)
  * [События, которые вызываются в течение `отображения`](#events-raised-during-show)
* [Собственные классы регионов](#custom-region-classes)
  * [Добавление собственных классов регионов](#attaching-custom-region-classes)
  * [Создание экземпляра вашего собственного региона](#instantiate-your-own-region)

## <a name="defining-an-application-region"></a> Определение регионов приложения

Вы можете добавить регионы в ваших приложениях, вызвав метод `addRegions`
у экземпляра вашего приложения. Этот метод ожидает один параметр в виде хэша.
В хэше заданы имена регионов и либо jQuery-селектор, либо объект `Region`.
Вы можете вызвать этот метод столько раз, сколько вам нужно, и он будет продолжать
добавлять регионы в приложение.

```js
myApp.addRegions({
  mainRegion: "#main-content",
  navigationRegion: "#navigation"
});
```

Вскоре после вызова `addRegions`, ваши регионы станут доступны у объекта вашего приложения.
В приведенном выше примере `myApp.mainRegion` и `myApp.navigationRegion` немедленно доступны для использования.

Если вы укажете одно и то же имя для региона дважды, будет использовано последнее определение.

Вы можете также добавить регионы через `LayoutView`:

```js
var AppLayoutView = Marionette.LayoutView.extend({
  template: "#layout-view-template",

  regions: {
    menu: "#menu",
    content: "#content"
  }
});
var layoutView = new AppLayoutView();
layoutView.render();
layoutView.menu.show(new MenuView());
layoutView.content.show(new MainContentView());
```

### <a name="region-configuration-types"></a> Типы конфигурирования регионов

Marionette поддреживает несколько способов определения регионов в вашем `Application` или `LayoutView`.

#### Селектор в виде строки

Для определения региона вы можете использовать jQuery-селектор в виде строки.

```js
myApp.addRegions({
  mainRegion: '#main'
});
```

#### Класс региона

Если у вас есть собственный класс региона, вы можете использовать его для определения региона.

**Внимание:** Убедитесь, что класс региона имеет свойство `el`, в противном случае регион создать не получится.

```js
var MyRegion = Marionette.Region.extend({
  el: '#main-nav'
});

myApp.addRegions({
  navigationRegion: MyRegion
});
```

#### Литерал объекта

Наконец, вы можете определить регионы в виде литерала объекта.
Определения в виде литерала объекта обычно содержит определенное `selector` или `el` свойство.
Свойство `selector` это селектор в виде строки, а свойство `el` может быть
как селектором в виде строки, так и Query-объектом или HTML-узлом.

Вы можете также задать свойство `regionClass` для собственного класса региона.
Если ваш `regionClass` уже имеет установленное свойство `el`, то вам не нужно
определять `selector` или `el` свойство в литерал объекте.

Любые другие свойства, заданные вами в литерале объекта, будут использоваться в качестве параметров, которые будут переданы в экземпляр региона, включая `allowMissingEl`.

Ordinarily regions enforce the presence of a backing DOM element.
In some instances it may be desirable to allow regions to be
instantiated and used without an element, such as when regions
defined by a parent `LayoutView` class are used by only some of its
subclasses. In these instances, the region can be defined with the
`allowMissingEl` option, suppressing the missing element error and
causing `show` calls to the region to be treated as no-ops.

```js
var MyRegion = Marionette.Region.extend();
var MyOtherRegion = Marionette.Region.extend();
var MyElRegion = Marionette.Region.extend({ el: '#footer' });

myApp.addRegions({
  contentRegion: {
    el: '#content',
    regionClass: MyRegion
  },

  navigationRegion: {
    el: '#navigation',
    regionClass: MyOtherRegion,

    // Параметры, которые будут переданы в экземпляр `MyOtherRegion` для
    // `navigationRegion` в `App`
    navigationOption: 42,
    anotherNavigationOption: 'foo'
  },

  footerRegion: {
    regionClass: MyElRegion
  }
});
```

Обратите внимание, что одним из основных преимуществ использования `regionClass` с уже
установленным `el` является возможность предоставления параметров для экземпляра региона.
Это невозможно, когда используется непосредственно класс региона, как в способах ранее.

```js
var MyRegion = Marionette.Region.extend({
  el: '#content',
});

myApp.addRegions({
  contentRegion: {
    regionClass: MyRegion,
    myRegionOption: 'bar',
    myOtherRegionOption: 'baz'
  }
});
```

#### Смешивание и комбинирование

Конечно вы можете смешивать и комбинировать различные типы конфигурирования регионов.

```js
var MyRegion = Marionette.Region.extend({
  el: '#content'
});

var MyOtherRegion = Marionette.Region.extend();

myApp.addRegions({
  contentRegion: MyRegion,

  navigationRegion: '#navigation',

  footerRegion: {
    el: '#footer',
    regionClass: MyOtherRegion
  }
});
```

## <a name="initialize-a-region-with-an-el"></a> Инициализация региона с помощью `el`

Вы можете указать `el` для региона, чтобы управлять местом, где будет создан экземпляр регион:

```js
var myRegion = new Marionette.Region({
  el: "#someElement"
});
```

Опция `el` может также быть ссылкой на DOM-узел:

```js
var myRegion = new Marionette.Region({
  el: document.querySelector("body")
});
```

Также `el` может быть `jQuery`-оберткой для DOM-узла:

```js
var myRegion = new Marionette.Region({
  el: $("body")
});
```

## <a name="basic-use"></a> Основное применение

### <a name="showing-a-view"></a> Отображение представления

Как только регион объявлен, вы можете вызвать его методы `show` и `empty` для отображения или выключения представления:

```js
var myView = new MyView();

// Рендеринг и отобрабражение представления
myApp.mainRegion.show(myView);

// Удаление текущего представления
myApp.mainRegion.empty();
```

#### опция preventDestroy

Если вы хотите заменить текущее представление на новое представлние, вы можете вызвать метод `show`,
этот метод, по умолчанию, автоматически уничтожит предыдущее представление. Вы можете предотвратить
это поведение передав опцию `{preventDestroy: true}` в параметрах метода; больше информации можно
прочитать в [События и коллбэки региона](#region-events-and-callbacks).

```js
// Отобразим первое представление.
var myView = new MyView();
myApp.mainRegion.show(myView);

// Заменим представление на другое.
// Метод `destroy` будет вызван для вас автоматически
var anotherView = new AnotherView();
myApp.mainRegion.show(anotherView);

// Заменим представление на другое.
// Предотвращием вызов метода `destroy`
var anotherView2 = new AnotherView();
myApp.mainRegion.show(anotherView2, { preventDestroy: true });
```

ПРИМЕЧАНИЕ: При использовании `preventDestroy: true` вы должны быть осторожны, необходимо помнить,
что ваши старые представления должны быть удалены вручную, чтобы предотвратить утечку памяти.

#### опция forceShow

Если вы повторно вызовите метод `show` с тем же представлением, то по умолчанию ничего
не произойдет, потому что представление уже в регионе. Вы можете заставить представление
переотобразиться (re-shown), для этого нужно передать опцию `{forceShow: true}` в параметрах метода.

```js
var myView = new MyView();
myApp.mainRegion.show(myView);

// сейчас повторный вызов метода `show` переотобразит представление
myApp.mainRegion.show(myView, {forceShow: true});
```

#### Emptying a region

You can empty a region of its view and contents by invoking `.empty()` on the region instance.
If you would like to prevent the view currently shown in the region from being `destroyed` you can pass `{preventDestroy: true}` to the empty method to prevent the default destroy behavior.
The empty method returns the region instance from the invocation of the method.

#### onBeforeAttach & onAttach

Regions that are attached to the document when you execute `show` are special in that the
views that they show will also become attached to the document. These regions fire a pair of triggerMethods on *all*
of the views that are about to be attached – even the nested ones. This can cause a performance issue if you're
rendering hundreds or thousands of views at once.

If you think these events might be causing some lag in your app, you can selectively turn them off
with the `triggerBeforeAttach` and `triggerAttach` properties.

```js
// No longer trigger attach
myRegion.triggerAttach = false;
```

You can override this on a per-show basis by passing it in as an option to show.

```js
// This region won't trigger beforeAttach...
myRegion.triggerBeforeAttach = false;

// Unless we tell it to
myRegion.show(myView, {triggerBeforeAttach: true});
```

### <a name="checking-whether-a-region-is-showing-a-view"></a> Проверка отображено ли представление в регионе

Если вы хотите проверить, есть ли представление у региона, то вы можете использовать
функцию `hasView`. Эта функция возвращает соответствующее булевское значение в зависимости от того,
отображено ли представление в регионе или нет.

### <a name="reset-a-region"></a> Сброс региона

Регион может быть сброшен в любое время через метод `reset`. Этот метод уничтожит
существующее представление, отображаемое в регионе, и удалит закэшированное значение из `el`.
Затем, регион отобразит представление, значение свойства `el` региона запрашивается у DOM.

```js
myRegion.reset();
```

Это полезно, когда регионы используются повторно для экземпляров представления,
а также в модульном тестировании.

### <a name="set-how-views-el-is-attached"></a> Настройка способа добавления `el` представления в DOM

Переопределение метода `attachHtml` региона изменяет поведение того, как представление
будет добавлено в DOM. Этот метод принимает один параметр - представление для отображения.

По умолчанию, реализация метода `attachHtml` является следующей:

```js
Marionette.Region.prototype.attachHtml = function(view) {
  this.$el.empty().append(view.el);
}
```

Происходит замещение содержимого региона значением/содержимым `el` представления.
Вы можете переопределить метод `attachHtml` для создания своего эффекта перехода
или чего-то еще.  

```js
Marionette.Region.prototype.attachHtml = function(view) {
  this.$el.hide();
  this.$el.html(view.el);
  this.$el.slideDown("fast");
}
```

Кроме того, можно задать свой собственный метод отрисовки для своего региона,
наследуясь от класса `Region` и добавив свою реализацию метода `attachHtml`.

Этот пример задает появление представления slide-эффектом снизу вверх экрана,
а не просто появление в нужном месте:

```js
var ModalRegion = Marionette.Region.extend({
  attachHtml: function(view) {
    // Некоторый эффект отображения представления:
    this.$el.empty().append(view.el);
    this.$el.hide().slideDown('fast');
  }
})

myApp.addRegions({
  mainRegion: '#main-region',
  modalRegion: {
    regionClass: ModalRegion,
    selector: '#modal-region'
  }
})
```

### <a name="attach-existing-view"></a> Добавление существующего представления

Есть несколько сценариев, как желательно добавлять существующее представление в регион,
без отрисовки или отображения представления и без замещения HTML-содержимого региона.
Например, для SEO и [универсального доступа](http://www.w3.org/WAI/intro/accessibility.php) часто нужен HTML, который должен сгенерироваться на сервере, а также для [прогрессивного улучшения](http://en.wikipedia.org/wiki/Progressive_enhancement) HTML.

Есть два способа сделать это:

* установить `currentView` в конструкторе региона
* вызвать `attachView` у экземпляра региона

#### <a name="set-currentview-on-initialization"></a> Установка `currentView` при инициализации

```js
var myView = new MyView({
  el: $("#existing-view-stuff")
});

var myRegion = new Marionette.Region({
  el: "#content",
  currentView: myView
});
```

#### <a name="call-attachview-on-region"></a> Вызов `attachView` в регионе

```js
myApp.addRegions({
  someRegion: "#content"
});

var myView = new MyView({
  el: $("#existing-view-stuff")
});

myApp.someRegion.attachView(myView);
```

## <a name="region-events-and-callbacks"></a> События и коллбэки региона

### <a name="events-raised-during-show"></a> События, которые вызываются в течение `отображения`:

Регион будет вызывать некоторые события при отображении и
уничтожении представления:

* "before:show" / `onBeforeShow` - Вызывается у экземпляра представления после того, как представление было отрисовано (rendered), но до его отображения (показа).
* "before:show" / `onBeforeShow` - Вызывается у экземпляра региона после того, как представление было отрисовано, но до его отображения.
* "show" / `onShow` - Вызывается у экземпляра представления, когда представление было отрисовано и отображено.
* "show" / `onShow` - Вызывается у экземпляра региона, когда представление было отрисовано и отображено.
* "before:swap" / `onBeforeSwap` - Вызывается у экземпляра региона до того, как новое представление будет отображено. ЗАМЕЧАНИЕ: это событие будет вызвано только тогда, когда представление сменилось (swapped), но не когда регион пустой.
* "before:swapOut" / `onBeforeSwapOut` - Вызывается у экземпляра региона до того, как новое представление начинает сменяться. ЗАМЕЧАНИЕ: это событие будет вызвано только тогда, когда представление сменилось, но не когда регион пустой.
* "swap" / `onSwap` - Вызывается у экземпляра региона тогда, когда новое представление `отображено`. ЗАМЕЧАНИЕ: это событие будет вызвано только тогда, когда представление сменилось, но не когда регион пустой.
* "swapOut" / `onSwapOut` - Вызывается у экземпляра региона тогда, когда новое представление сменилось и собирается быть выполнено замещение отображаемого представления новым представлением. ЗАМЕЧАНИЕ: это событие будет вызвано только тогда, когда представление сменилось, но не когда регион пустой.
* "before:empty" / `onBeforeEmpty` - Вызывается у экземпляра региона до того, как представление будет очищено/уничтожено.
* "empty" / `onEmpty` - Вызывается тогда, когда представление было очищено/уничтожено.

Эти события могут быть использованы для запуска кода, когда ваш регион открывает и/или уничтожает представления.

```js
myApp.mainRegion.on("before:show", function(view, region, options){
  // manipulate the `view` or do something extra
  // with the `region`
  // you also have access to the `options` that were passed to the Region.show call
});

myApp.mainRegion.on("show", function(view, region, options){
  // manipulate the `view` or do something extra
  // with the `region`
  // you also have access to the `options` that were passed to the Region.show call
});

myApp.mainRegion.on("before:swap", function(view, region, options){
  // manipulate the `view` or do something extra
  // with the `region`
  // you also have access to the `options` that were passed to the Region.show call
});

myApp.mainRegion.on("swap", function(view, region, options){
  // manipulate the `view` or do something extra
  // with the `region`
  // you also have access to the `options` that were passed to the Region.show call
});

myApp.mainRegion.on("before:swapOut", function(view, region, options){
  // manipulate the `view` or do something extra
  // with the `region`
  // you also have access to the `options` that were passed to the Region.show call
});

myApp.mainRegion.on("swapOut", function(view, region, options){
  // manipulate the `view` or do something extra
  // with the `region`
  // you also have access to the `options` that were passed to the Region.show call
});

myApp.mainRegion.on("empty", function(view, region, options){
  // manipulate the `view` or do something extra
  // with the `region`
  // you also have access to the `options` that were passed to the Region.show call
});

var MyRegion = Marionette.Region.extend({
  // ...

  onBeforeShow: function(view, region, options) {
    // the `view` has not been shown yet
  },

  onShow: function(view, region, options){
    // the `view` has been shown
  }
});

var MyView = Marionette.ItemView.extend({
  onBeforeShow: function(view, region, options) {
    // called before the `view` has been shown
  },
  onShow: function(view, region, options){
    // called when the `view` has been shown
  }
});

var MyRegion = Marionette.Region.extend({
  // ...

  onBeforeSwap: function(view, region, options) {
    // the `view` has not been swapped yet
  },

  onSwap: function(view, region, options){
    // the `view` has been swapped
  },

  onBeforeSwapOut: function(view, region, options) {
    // the `view` has not been swapped out yet
  },

  onSwapOut: function(view, region, options){
    // the `view` has been swapped out
  }
});
```

## <a name="custom-region-classes"></a> Собственные классы регионов

Вы можете определить свой собственный регион, наследуясь от класса `Region`.
Это позволяет вам создавать новую функциональность и обеспечивает базовый
набор функциональных возможностей для вашего приложения.

### <a name="attaching-custom-region-classes"></a> Добавление собственных классов регионов

После того, как вы определите класс регион, вы можете использовать его для конфигурирования регионов в приложении.
Для этого, вы должны добавить новый класс регион в качестве значения для регион класса.
В случае использования `addRegions`, указывается непосредственно сам конструктор класса регион,
а не экземпляр класса регион.

```js
var FooterRegion = Marionette.Region.extend({
  el: "#footer"
});

myApp.addRegions({
  footerRegion: FooterRegion
});
```

Вы можете также указать селектор для региона, используя литерал объекта для конфигурирования региона.

```js
var FooterRegion = Marionette.Region.extend({
  el: "#footer"
});

myApp.addRegions({
  footerRegion: {
    selector: "#footer",
    regionClass: FooterRegion
  }
});
```

Обратите внимание, что регион должен иметь элемент, с которым он будет связан.
Если вы не указали селектор, когда добавляли экземпляр региона в ваше приложение или `LayoutView`,
то `el` региона должно быть указано либо при его определении, либо в опциях конструктора.

### <a name="instantiate-your-own-region"></a> Создание экземпляра вашего собственного региона

Бывают ситуации, когда вы хотите добавить регион в ваше приложение
после того, как ваше приложение запустилось и работает. Что бы сделать это,
вам нужно наследоваться от класса `Region`, как показано выше, и затем использовать эту
функцию конструктора по вашему усмотрению:

```js
var SomeRegion = Marionette.Region.extend({
  el: "#some-div",

  initialize: function(options){
    // ваш код инициализации должен быть здесь
  }
});

myApp.someRegion = new SomeRegion();

myApp.someRegion.show(someView);
```

При желании, вы можете добавить функцию `initialize` при определении
вашего класса региона, как показано в примере выше. Эта функция принимает
`options`, которые были переданы в конструктор региона, аналогично тому, как это
происходит у `Backbone.View`.
