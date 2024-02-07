---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Laravel Mix


<a name="introduction"></a>
## Введение

[Laravel Mix](https://github.com/laravel-mix/laravel-mix) – это пакет, разработанный создателем [Laracasts](https://laracasts.com) Джеффри Уэй, предлагает гибкий API для определения шагов сборки [Webpack](https://webpack.js.org) для вашего приложения с использованием нескольких распространенных препроцессоров CSS и JavaScript.

Другими словами, Mix упрощает компиляцию и минимизацию файлов CSS и JavaScript вашего приложения. Посредством простой цепочки методов вы можете гибко определять свой сценарий по сборки исходников. Например:

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

> [!NOTE]
> В новых установках Laravel, Vite заменил Laravel Mix. Для документации по Mix, пожалуйста, посетите [официальный сайт Laravel Mix](https://laravel-mix.com/). Если вы хотите перейти на Vite, пожалуйста, ознакомьтесь с нашим [руководством по миграции на Vite](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite).
