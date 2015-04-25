git e9d876cf2b530347fdd44bad2b00a7ffe1b75c86

---

# Постраничный вывод данных (пагинация)

- [Настройка](#configuration)
- [Использование](#usage)
- [Параметры в ссылках](#appending-to-pagination-links)
- [Конвертация в JSON](#converting-to-json)
- [Изменение отображения](#custom-presenters)

<a name="configuration"></a>
## Настройка

В других фрейморках пагинация (постраничный вывод данных) может быть большой проблемой. Laravel же делает этот процесс безболезненным. В файле настроек `app/config/view.php` есть единственный параметр `pagination`, который указывает, какой шаблон (views) нужно использовать при создании навигации по страницам. Изначально Laravel включает в себя два таких шаблона.

Шаблон `pagination::slider` выведет "умный" список страниц в зависимости от текущего положения, а шаблон `pagination::simple` просто создаст ссылки "Назад" и "Вперёд" для простой навигации. **Оба шаблона совместимы с [Twitter Bootstrap](http://getbootstrap.com).**

<a name="usage"></a>
## Использование

Есть несколько способов разделения данных на страницы. Самый простой - используя метод `paginate` объекта-[построителя запросов](/docs/4.2/queries) или на модели [Eloquent](/docs/4.2/eloquent).

#### Постраничный вывод выборки из БД

	$users = DB::table('users')->paginate(15);

> **Примечание:** Если вы используете `groupBy` в запросе, то встроенная пагинация Laravel будет работать неэффективно. В этом случае вам нужно сделать пагинацию вручную при помощи `Paginator::make`.

#### Постраничный вывод запроса Eloquent

	$allUsers = User::paginate(15);

	$users = User::where('votes', '>', 100)->paginate(15);

Аргумент, передаваемый методу `paginate` - число строк, которые вы хотите видеть на одной странице. Блок пагинации в шаблоне отображаются методом `links`:

	<div class="container">
		<?php foreach ($users as $user): ?>
			<?php echo $user->name; ?>
		<?php endforeach; ?>
	</div>

	<?php echo $users->links(); ?>

Это всё, что нужно для создания страничного вывода! Заметьте, что нам не понадобилось уведомлять фреймворк о номере текущей страницы - Laravel определит его сам. Номер страницы добавляется к урлу в виде строки запроса с параметром page: `?page=N`.

Вы можете получить информацию о текущем положении с помощью этих методов:

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`

#### Простая пагинация

If you are only showing "Next" and "Previous" links in your pagination view, you have the option of using the `simplePaginate` method to perform a more efficient query. This is useful for larger datasets when you do not require the display of exact page numbers on your view:

Если вам нужна простая навигация, состоящая только из кнопок "Вперед" и "Назад" (полезно на больших объемах данных, где нужны только последние данные), то воспользуйтесь методом `simplePaginate`. Это уменьшит количество запросов к БД.

	$someUsers = User::where('votes', '>', 100)->simplePaginate(15);

#### Создание пагинации вручную

	Иногда вам может потребоваться создать объект пагинации вручную. Вы можете сделать это методом `Paginator::make`:

	$paginator = Paginator::make($items, $totalItems, $perPage);

#### Настройка URI для вывода ссылок

	$users = User::paginate();

	$users->setBaseUrl('custom/url');

Пример выше создаст ссылки наподобие: http://example.com/custom/url?page=2

<a name="appending-to-pagination-links"></a>
## Параметры в ссылках

Вы можете добавить параметры запросов к ссылкам страниц с помощью метода `appends` страничного объекта:

	<?php echo $users->appends(array('sort' => 'votes'))->links(); ?>

Код выше создаст ссылки наподобие http://example.com/something?page=2&sort=votes

Чтобы добавить к урлу хэш-последовательность ("#xyz" в конце урла), используйте метод `fragment`:

	<?php echo $users->fragment('foo')->links(); ?>

Код выше создаст ссылки типа http://example.com/something?page=2#foo

<a name="converting-to-json"></a>
## Конвертация To JSON

Класс `Paginator` реализует (implements) `Illuminate\Support\Contracts\JsonableInterface`, следовательно, у него есть метод `toJson`, который используется для вывода пагинируемой информации в формате json. Помимо пагинируемых данных, которые располагаются в `data`, этот метод добавляет мета-информацию, а именно: `total`, `current_page`, `last_page`, `from`, `to`. 

<a name="custom-presenters"></a>
## Изменение отображения

По умолчанию пагинация в Laravel совместима с Twitter Bootstrap. Если вы хотите изменить html-код ссылок пагинации, вам нужно использовать свой презентер.

### Расширение абстрактного презентера

Допустим, наш проект построен на css-фреймворке Zurb Foundation. Расширим (extends) `Illuminate\Pagination\Presenter` и реализуем его абстрактные методы:

	class ZurbPresenter extends Illuminate\Pagination\Presenter {

        public function getActivePageWrapper($text)
        {
            return '<li class="current"><a href="">'.$text.'</a></li>';
        }

        public function getDisabledTextWrapper($text)
        {
            return '<li class="unavailable"><a href="">'.$text.'</a></li>';
        }

        public function getPageLinkWrapper($url, $page, $rel = null)
        {
            return '<li><a href="'.$url.'">'.$page.'</a></li>';
        }

    }

### Использование своего презентера

1. Создаем шаблон в `app/views`, который должен отображать ссылки пагинации, с таким, например, содержимым:

	<ul class="pagination">
        <?php echo with(new ZurbPresenter($paginator))->render(); ?>
    </ul>

2. В конфиге `app/config/view.php` указываем имя этого шаблона (параметр `'pagination'`).
