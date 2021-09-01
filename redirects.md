git 2a0615c0b2833c9a6c80052c6896071fa525d0fc

---

# HTTP переадресация

- [Создание переадресации](#creating-redirects)
- [Переадресация на именованные маршруты](#redirecting-named-routes)
- [Переадресация на методы контроллера](#redirecting-controller-actions)
- [Переадресация с данными сеанса](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## Создание переадресации

Переадресация является экземпляром класса `Illuminate\Http\RedirectResponse` и содержит в ответе правильные заголовки, необходимые для перенаправления пользователя на другой URL. Есть несколько способов сгенерировать экземпляр `RedirectResponse`. Самый простой способ — использовать глобальный помощник `redirect`:

    Route::get('/dashboard', function () {
        return redirect('/home/dashboard');
    });

Иногда необходимо перенаправить пользователя в его предыдущее местоположение, например, когда отправленная форма недействительна. Это можно сделать с помощью глобальной вспомогательной функции `back`. Поскольку эта функция использует [сессии](/docs/{{version}}/session), убедитесь, что маршрут, вызывающий функцию `back`, использует группу посредников` web` или применяет все необходимые посредники сессии:

    Route::post('/user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## Переадресация на именованные маршруты

Когда вы вызываете помощник `redirect` без параметров, возвращается экземпляр `Illuminate\Routing\Redirector`, что позволяет вам вызывать любой метод в экземпляре `Redirector`. Например, чтобы сгенерировать `RedirectResponse` на именованный маршрут, вы можете использовать метод route:

    return redirect()->route('login');

Если ваш маршрут имеет параметры, вы можете передать их в качестве второго аргумента методу `route`:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### Передача параметров через модели Eloquent

Если вы перенаправляете на маршрут с параметром "ID", который заполняется из модели Eloquent, вы можете передать саму модель. ID будет извлечен автоматически:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Если вы хотите настроить значение, которое помещается в параметр маршрута, вы должны переопределить метод `getRouteKey` в вашей модели Eloquent:

    /**
     * Получить значение ключа маршрута модели.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
## Перенаправление на методы контроллера

Вы также можете обеспечить перенаправления на [методы контроллера](/docs/{{version}}/controllers). Для этого передайте имя контроллера и его метода методу `action` экземпляра `redirect`:

    use App\Http\Controllers\HomeController;

    return redirect()->action([HomeController::class, 'index']);

Если метод контроллера требует параметров, вы можете передать их в качестве второго аргумента методу `action`:

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
## Перенаправление с данными сеанса

Перенаправление на новый URL-адрес и [передача данных в сеанс](/docs/{{version}}/session#flash-data) обычно выполняются одновременно. Обычно это делается после успешного выполнения действия, когда вы отправляете сообщение об успешном завершении сеанса. Для удобства вы можете создать экземпляр `RedirectResponse` и передать данные в сеанс в цепочке методов:

    Route::post('/user/profile', function () {
        // Update the user's profile...

        return redirect('/dashboard')->with('status', 'Profile updated!');
    });

Вы можете использовать метод `withInput`, предоставляемый экземпляром `RedirectResponse`, для передачи входных данных текущего запроса в сеанс перед перенаправлением пользователя на новый URL. После того как данные были переданы в сеанс, вы можете легко [получить их](/docs/{{version}}/requests#retrieving-old-input) во время следующего запроса:

    return back()->withInput();

После перенаправления вы можете отобразить всплывающее сообщение [сеанса](/docs/{{version}}/session). Например, используя [синтаксис Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif
