git e73c40f0dea4db1205c83584d6c5b544b5ff1683

---

# HTTP редиректы

- [Создание редиректов](#creating-redirects)
- [Редиректы на именованные роуты](#redirecting-named-routes)
- [Редиректы на методы контроллера](#redirecting-controller-actions)
- [Редиректы с данными сессии](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## Создание редиректов

Отклики для редиректов — это экземпляры класса `Illuminate\Http\RedirectResponse`; они содержат соответствующие заголовки, необходимые для переадресации пользователя на другой URL. Есть несколько способов создания экземпляров `RedirectResponse`. Самый простой способ - использовать глобальный хелпер `redirect`:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

Иногда необходимо перенаправить пользователя в предыдущее место, например, когда в отправленной форме обнаружены ошибки. Вы можете сделать это с помощью глобального хелпера `back`. Так как для этой функции используются [сессии](/docs/{{version}}/session), убедитесь в том, что вызывающий функцию `back` роут использует группу посредников `web` или использует всех посредников сессий:

    Route::post('user/profile', function () {
        // Проверка запроса...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## Редиректы на именованные роуты

При вызове хелпера `redirect` без параметров возвращается экземпляр `Illuminate\Routing\Redirector`, позволяя вам вызывать любой метод на экземпляре `Redirector`. Например, вы можете использовать метод `route`, чтобы сгенерировать `RedirectResponse` на именованный роут:

    return redirect()->route('login');

Если у вашего роута есть параметры, то вы можете передать их в качестве второго аргумента метода `route`:

    // Для роута со следующим URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### Получение параметров из моделей Eloquent

Если вы делаете переадресацию на роут с параметром "ID", который был получен из модели Eloquent, то вы можете передать саму модель. ID будет извлечён автоматически:

    // Для роута со следующим URI: profile/{id}

    return redirect()->route('profile', [$user]);

Если вы хотите изменить значение, которое помещается в параметр роута, вам надо переопределить метод `getRouteKey` в вашей модели Eloquent:

    /**
     * Получить значение ключа роута модели.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
## Редиректы на методы контроллера

Также вы можете создать редиректы на [методы контроллера](/docs/{{version}}/controllers). Для этого передайте контроллер и имя метода контроллера в метод `action`. Помните, что вам не надо указывать полное пространство имён контроллера, потому что`RouteServiceProvider` Laravel автоматически задаст его сам:

    return redirect()->action('HomeController@index');

Если роуту вашего контроллера нужны параметры, то вы можете передать их вторым аргументом метода `action`:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
## Редирект с данными сессии

Редирект на новый URL и [передача данных в сессию](/docs/{{version}}/session#flash-data) обычно происходят одновременно. Обычно это делается после успешного выполнения действия, когда вы передаёте в сессию сообщение об успехе. Для удобства вы можете создать экземпляр `RedirectResponse` и передать данные в сессию в одной цепочке вызовов:

    Route::post('user/profile', function () {
        // Обновление профайла юзера...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Когда пользователь переадресован, то вы можете вывести сообщение из [сессии](/docs/{{version}}/session). Например, используя [синтаксис Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif
