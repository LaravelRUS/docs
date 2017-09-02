git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Помощь в разработке фреймворка

- [Сообщения о багах](#bug-reports)
- [Обсуждение разработки](#core-development-discussion)
- [Какая ветка?](#which-branch)
- [Уязвимости безопасности](#security-vulnerabilities)
- [Стиль оформления кода](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Сообщения о багах

Чтобы стимулировать активное сотрудничество, Laravel настоятельно рекомендует отправлять запросы на исправления и улучшения, а не только отчёты об ошибках. "Отчёт о баге" также может быть отправлен в виде запроса на улучшение, содержащего проваленный тест.

А если вы отправляете отчёт об ошибке, то ваша заявка должна содержать заголовок и понятное описание проблемы. Вам следует прикрепить как можно больше сопутствующей информации и пример кода, демонстрирующий проблему. Цель отчёта — упростить вам (и остальным) возможность воспроизвести ошибку и разработать исправление.

Помните, отчёт об ошибке создаётся с целью объединения людей, столкнувшихся с той же проблемой, для её решения. Не ждите, что отчёт автоматически приведёт к скорейшему решению, и остальные разработчики кинуться решать проблему. Создание отчёта служит началом для вас и остальных в решении проблемы.

Исходный код Laravel расположен на Github. Вот репозитории каждого из проектов Laravel:

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Website](https://github.com/laravel/laravel.com)
</div>

<a name="core-development-discussion"></a>
## Обсуждение разработки

Вы можете предложить новую функцию или исправление существующего поведения Laravel на внутренней [доске проблем](https://github.com/laravel/internals/issues) Laravel Internals. Пожалуйста, когда предлагаете новую функцию, напишите хотя бы часть необходимого для её реализации кода.

Обсуждение относительно багов, новых функций и реализации существующих функций ведётся на канале `#internals` в чате [LaraChat](https://larachat.co) команды Slack. Тэйлор Отвелл, создатель Laravel, обычно присутствует на канале по будням с 8 утра до 5 вечера (по чикагскому времени UTC-06:00), а иногда и в другое время.

<a name="which-branch"></a>
## Какая ветка?

**Все** исправления багов должны отправляться в последнюю стабильную ветку. Исправления багов **никогда** не должны отправляться в ветку `master`, только если они относятся к функциям, которые есть только в следующем релизе.

**Небольшие** функции, которые **полностью обратно совместимы** с текущим релизом Laravel, могут быть отправлены в последнюю стабильную ветку.

**Крупные**  новые функции должны всегда отправляться в ветку `master`, которая содержит следующий релиз Laravel.

Если вы не уверены к каким функциям относится ваша, мажорным или минорным, спросите об этом Тэйлора Отвелла на канале `#internals` в [LaraChat](https://larachat.co) команды Slack.

<a name="security-vulnerabilities"></a>
## Уязвимости безопасности

Если вы обнаружите уязвимость в безопасности Laravel, пожалуйста, напишите об этом Тэйлору Отвеллу по адресу <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Все уязвимости будут оперативно устранены.

<a name="coding-style"></a>
## Стиль оформления кода

Laravel следует стандарту написания кода [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) и стандарту автозагрузки [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md).

<a name="phpdoc"></a>
### PHPDoc

Ниже приведён пример правильного блока документации Laravel. Обратите внимание, что после атрибута `@param` стоят два пробела, тип аргумента, ещё два пробела и в конце имя переменной:

    /**
     * Зарегистрировать связку с контейнером.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

Если ваш стиль написания кода не идеален, не волнуйтесь! [StyleCI](https://styleci.io/) автоматически поместит в репозиторий Laravel все исправления стиля после размещения запроса на включение. Это позволяет нам сконцентрироваться на содержании, а не форме.
