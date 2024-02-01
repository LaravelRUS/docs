---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Eloquent: Фабрики (Factory)

<a name="introduction"></a>
## Введение

При тестировании вашего приложения вам может потребоваться вставить несколько записей в вашу базу данных. Вместо того чтобы вручную указывать значение каждого столбца, Laravel позволяет вам определять набор атрибутов по умолчанию для каждой из ваших [моделей Eloquent](/docs/{{version}}/eloquent), используя фабрики моделей.

Чтобы увидеть пример написания фабрики, взгляните на файл `database/factories/UserFactory.php` в вашем приложении. Эта фабрика включена во все новые приложения Laravel и содержит следующее определение фабрики:

    namespace Database\Factories;

    use Illuminate\Support\Str;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class UserFactory extends Factory
    {
        /**
         * Define the model's default state.
         *
         * @return array<string, mixed>
         */
        public function definition(): array
        {
            return [
                'name' => fake()->name(),
                'email' => fake()->unique()->safeEmail(),
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
                'remember_token' => Str::random(10),
            ];
        }
    }
Как видите, фабрики – это классы, которые расширяют базовый класс фабрики Laravel и определяют метод `definition`. Метод `definition` возвращает набор значений атрибутов по умолчанию, которые должны применяться при создании модели с использованием фабрики.

С помощью помощник `fake` фабрики имеют доступ к библиотеке PHP [Faker](https://github.com/FakerPHP/Faker), которая позволяет удобно генерировать различные виды случайных данных для тестирования и заполнения базы данных.

> [!NOTE]  
> Вы можете установить языковой стандарт Faker для своего приложения, добавив опцию `faker_locale` в конфигурационном файле `config/app.php`.

<a name="defining-model-factories"></a>
## Определение фабрик моделей

<a name="generating-factories"></a>
### Генерация фабрик

Чтобы сгенерировать новую фабрику, используйте команду `make:factory` [Artisan](/docs/{{version}}/artisan):

```shell
php artisan make:factory PostFactory
```

Эта команда поместит новый класс фабрики в каталог `database/factories`.

<a name="factory-and-model-discovery-conventions"></a>
#### Соглашение для определения моделей и фабрик

После того как вы определили свои фабрики, вы можете использовать статический метод `factory` предоставляемый вашим моделям с помощью трейта `Illuminate\Database\Eloquent\Factories\HasFactory`, чтобы создать экземпляр фабрики для этой модели.

Метод `factory` трейта `HasFactory` будет использовать соглашения для определения подходящей фабрики для модели. В частности, метод будет искать фабрику в пространстве имен `Database\Factories`, имя класса которой соответствует имени модели и имеет суффикс `Factory`. Если эти соглашения не применимы к вашему конкретному приложению или фабрике, вы можете перезаписать метод `newFactory` вашей модели, чтобы напрямую возвращать экземпляр соответствующей фабрики модели:

    use Database\Factories\Administration\FlightFactory;

    /**
     * Создать новый экземпляр фабрики для модели.
     */
    protected static function newFactory(): Factory
    {
        return FlightFactory::new();
    }

Затем определите свойство `model` в соответствующей фабрике:

    use App\Administration\Flight;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class FlightFactory extends Factory
    {
        /**
         * Название модели, соответствующей фабрике.
         *
         * @var class-string<\Illuminate\Database\Eloquent\Model>
         */
        protected $model = Flight::class;
    }

<a name="factory-states"></a>
### Состояния фабрик

Методы управления состоянием позволяют вам определять дискретные изменения, которые могут быть применены к вашим фабрикам моделей в любой их комбинации. Например, ваша фабрика `Database\Factories\UserFactory` может содержать метод состояния `suspended`, который изменяет одно из значений атрибута по умолчанию.

Методы преобразования состояния обычно вызывают метод `state` базового класса фабрики Laravel. Метод `state` принимает замыкание, которое получит массив изначально определенных для фабрики атрибутов, и должен вернуть массив изменяемых атрибутов:

    use Illuminate\Database\Eloquent\Factories\Factory;

    /**
     * Указать, что аккаунт пользователя временно приостановлен.
     */
    public function suspended(): Factory
    {
        return $this->state(function (array $attributes) {
            return [
                'account_status' => 'suspended',
            ];
        });
    }

<a name="trashed-state"></a>
#### "Trashed" State

Если ваша модель Eloquent поддерживает [программное удаление](/docs/{{version}}/eloquent#soft-deleting), вы можете вызвать встроенный метод состояния `trashed`  чтобы указать, что созданная модель уже должна быть "программно удалена". Вам не нужно ручным образом определять состояние `trashed` так как оно автоматически доступно для всех фабрик:

    use App\Models\User;

    $user = User::factory()->trashed()->create();

<a name="factory-callbacks"></a>
### Хуки фабрик

Хуки фабрик регистрируются с использованием методов `afterMaking` и `afterCreating` и позволяют выполнять дополнительные задачи после инициализации или создания модели. Вы должны зарегистрировать эти хуки, переопределив метод `configure` в вашем классе фабрики. Этот метод будет автоматически вызываться Laravel при создании экземпляра фабрики:

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class UserFactory extends Factory
    {
        /**
         * Конфигурация фабрики модели.
         */
        public function configure(): static
        {
            return $this->afterMaking(function (User $user) {
                // ...
            })->afterCreating(function (User $user) {
                // ...
            });
        }

        // ...
    }


Вы также можете зарегистрировать хуки фабрики внутри методов состояния для выполнения дополнительных задач, специфичных для определенного состояния:

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;

    /**
     * Указать, что аккаунт пользователя временно приостановлен.
     */
    public function suspended(): Factory
    {
        return $this->state(function (array $attributes) {
            return [
                'account_status' => 'suspended',
            ];
        })->afterMaking(function (User $user) {
            // ...
        })->afterCreating(function (User $user) {
            // ...
        });
    }

<a name="creating-models-using-factories"></a>
## Создание моделей с использованием фабрик

<a name="instantiating-models"></a>
### Инициализация экземпляров моделей

После того как вы определили свои фабрики, вы можете использовать статический метод `factory`, предоставляемый вашим моделям с помощью трейта `Illuminate\Database\Eloquent\Factories\HasFactory`, чтобы инициализировать экземпляр фабрики для этой модели. Давайте посмотрим на несколько примеров создания моделей. Во-первых, мы воспользуемся методом `make` для создания моделей без сохранения в базе данных:

    use App\Models\User;

    $user = User::factory()->make();

Вы можете создать коллекцию из множества моделей, используя метод `count`:

    $users = User::factory()->count(3)->make();

<a name="applying-states"></a>
#### Применение состояний

Вы также можете применить к моделям любое из ваших [состояний](#factory-states) .Если вы хотите применить к моделям несколько изменений состояния, то вы можете просто вызвать методы преобразования состояния напрямую:

    $users = User::factory()->count(5)->suspended()->make();

<a name="overriding-attributes"></a>
#### Переопределение атрибутов

Если вы хотите переопределить некоторые значения по умолчанию для ваших моделей, вы можете передать массив значений методу `make`. Будут заменены только указанные атрибуты, в то время как для остальных атрибутов сохранятся значения по умолчанию, указанные в фабрике:

    $user = User::factory()->make([
        'name' => 'Abigail Otwell',
    ]);

В качестве альтернативы, метод `state` может быть вызван непосредственно на экземпляре фабрики для выполнения быстрого преобразования состояния:

    $user = User::factory()->state([
        'name' => 'Abigail Otwell',
    ])->make();

> [!NOTE]  
> [Защита от массового назначения](/docs/{{version}}/eloquent#mass-assignment) автоматически отключается при создании моделей с использованием фабрик. .

<a name="persisting-models"></a>
### Сохранение моделей

Метод `create` инициализирует экземпляры модели и сохраняет их в базе данных с помощью метода `save` модели Eloquent:

    use App\Models\User;

    // Создаем один экземпляр `App\Models\User` ...
    $user = User::factory()->create();

    // Создаем три экземпляра `App\Models\User` .
    $users = User::factory()->count(3)->create();

Вы можете переопределить атрибуты модели по умолчанию, передав массив атрибутов методу `create`:

    $user = User::factory()->create([
        'name' => 'Abigail',
    ]);

<a name="sequences"></a>
### Последовательность состояний (Sequences)

Иногда может возникнуть желание чередовать значение определенного атрибута модели для каждой создаваемой модели. Вы можете добиться этого, определив последовательность преобразований состояния модели. Например, вы можете чередовать значение столбца `admin` между `Y` и `N` для каждого вновь созданного пользователя:

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Sequence;

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        ['admin' => 'Y'],
                        ['admin' => 'N'],
                    ))
                    ->create();

В этом примере пять пользователей будут созданы со значением `admin`, равным `Y`, и пять пользователей – со значением `admin`, равным `N`.

При необходимости вы можете внедрить замыкание в качестве значения последовательности. Замыкание будет вызываться каждый раз, когда последовательности потребуется новое значение:

    use Illuminate\Database\Eloquent\Factories\Sequence;

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        fn (Sequence $sequence) => ['role' => UserRoles::all()->random()],
                    ))
                    ->create();

Внутри замыкания последовательности вы можете получить доступ к свойствам `$index` или `$count` экземпляра последовательности, который вводится в замыкание. Свойство `$index` содержит номер текущей итерации, а свойство `$count` - общее количество итераций:

    $users = User::factory()
                    ->count(10)
                    ->sequence(fn (Sequence $sequence) => ['name' => 'Name '.$sequence->index])
                    ->create();


Для удобства последовательности также можно применять с использованием метода `sequence`, который внутренне просто вызывает метод `state`. Метод `sequence` принимает замыкание или массивы атрибутов для последовательности:
    
    $users = User::factory()
                    ->count(2)
                    ->sequence(
                        ['name' => 'First User'],
                        ['name' => 'Second User'],
                    )
                    ->create();

<a name="factory-relationships"></a>
## Отношения

<a name="has-many-relationships"></a>
### HОтношения Has Many

Теперь давайте рассмотрим построение отношений моделей Eloquent с использованием текучего интерфейса методов фабрик Laravel. Во-первых, предположим, что у нашего приложения есть модель `App\Models\User` и модель `App\Models\Post`. Также предположим, что модель `User` определяет отношения `hasMany` с `Post`. Мы можем создать пользователя с тремя постами, используя метод `has`, предоставляемый фабриками Laravel. Метод `has` принимает экземпляр фабрики:

    use App\Models\Post;
    use App\Models\User;

    $user = User::factory()
                ->has(Post::factory()->count(3))
                ->create();

По соглашению, при передаче модели `Post` методу `has`, Laravel будет предполагать, что модель `User` должна иметь метод `posts`, который определяет отношения. При необходимости вы можете явно указать имя отношения, которым вы хотите управлять:

    $user = User::factory()
                ->has(Post::factory()->count(3), 'posts')
                ->create();

Конечно, вы можете выполнять манипуляции с состоянием связанных моделей. Кроме того, вы можете преобразовать состояние связанной модели с помощью замыкания, предоставив ему доступ к родительской модели:

    $user = User::factory()
                ->has(
                    Post::factory()
                            ->count(3)
                            ->state(function (array $attributes, User $user) {
                                return ['user_type' => $user->type];
                            })
                )
                ->create();

<a name="has-many-relationships-using-magic-methods"></a>
#### Использование магических методов Has Many

Для удобства вы можете использовать магические методы отношений фабрики Laravel для построения отношений. Например, в следующем примере будет использоваться соглашение, определяющее, что связанные модели должны быть созданы с помощью метода отношений `posts` модели `User`:

    $user = User::factory()
                ->hasPosts(3)
                ->create();

При использовании магических методов для создания отношений фабрики вы можете передать массив атрибутов для их переопределения в связанных моделях:

    $user = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

Вы можете преобразовать состояние связанной модели с помощью замыкания, предоставив ему доступ к родительской модели:

    $user = User::factory()
                ->hasPosts(3, function (array $attributes, User $user) {
                    return ['user_type' => $user->type];
                })
                ->create();

<a name="belongs-to-relationships"></a>
### Отношения Belongs To

Теперь, когда мы изучили, как построить отношения Has Many с помощью фабрик, давайте рассмотрим обратное отношение. Метод `for` используется для определения родительской модели, к которой принадлежат модели, созданные фабрикой. Например, мы можем создать три экземпляра модели `App\Models\Post`, которые принадлежат одному пользователю:

    use App\Models\Post;
    use App\Models\User;

    $posts = Post::factory()
                ->count(3)
                ->for(User::factory()->state([
                    'name' => 'Jessica Archer',
                ]))
                ->create();

Если у вас уже есть экземпляр родительской модели, который должен быть связан с создаваемыми вами моделями, вы можете передать экземпляр модели методу `for`:

    $user = User::factory()->create();

    $posts = Post::factory()
                ->count(3)
                ->for($user)
                ->create();

<a name="belongs-to-relationships-using-magic-methods"></a>
#### Использование магических методов Belongs To

Для удобства вы можете использовать магические методы отношений фабрики Laravel для построения отношений Belongs To. Например, в следующем примере будет использоваться соглашение, чтобы определить, что три поста должны принадлежать отношениям `user` в модели `Post`:

    $posts = Post::factory()
                ->count(3)
                ->forUser([
                    'name' => 'Jessica Archer',
                ])
                ->create();

<a name="many-to-many-relationships"></a>
### Отношения Many To Many

Как и [отношения Has Many](#has-many-relationships), отношения Many To Many могут быть созданы с использованием метода `has`:

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->has(Role::factory()->count(3))
                ->create();

<a name="pivot-table-attributes"></a>
#### Атрибуты сводной таблицы

Если вам нужно определить атрибуты, которые должны быть установлены в сводной / промежуточной таблице, связывающей модели, вы можете использовать метод `hasAttached`. Этот метод принимает в качестве второго аргумента массив имен и значений атрибутов сводной таблицы:

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->hasAttached(
                    Role::factory()->count(3),
                    ['active' => true]
                )
                ->create();

Вы можете преобразовать состояние связанной модели с помощью замыкания, предоставив ему доступ к родительской модели:

    $user = User::factory()
                ->hasAttached(
                    Role::factory()
                        ->count(3)
                        ->state(function (array $attributes, User $user) {
                            return ['name' => $user->name.' Role'];
                        }),
                    ['active' => true]
                )
                ->create();

Если у вас уже есть экземпляры модели, которые вы хотите прикрепить к создаваемым моделям, вы можете передать экземпляры модели методу `hasAttached`. В этом примере всем трем пользователям будут назначены одни и те же три роли:

    $roles = Role::factory()->count(3)->create();

    $user = User::factory()
                ->count(3)
                ->hasAttached($roles, ['active' => true])
                ->create();

<a name="many-to-many-relationships-using-magic-methods"></a>
#### Использование магических методов Many To Many

Для удобства вы можете использовать магические методы отношений фабрики Laravel для построения отношений Many To Many. Например, в следующем примере будет использоваться соглашение, чтобы определить, что связанные модели должны быть созданы с помощью метода отношений `roles` модели `User`:

    $user = User::factory()
                ->hasRoles(1, [
                    'name' => 'Editor'
                ])
                ->create();

<a name="polymorphic-relationships"></a>
### Полиморфные отношения

[Полиморфные отношения](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) также могут быть созданы с использованием фабрик. Полиморфные отношения Morph Many создаются так же, как типичные отношения Has Many. Например, если модель `App\Models\Post` имеет отношение `morphMany` с моделью `App\Models\Comment`:

    use App\Models\Post;

    $post = Post::factory()->hasComments(3)->create();

<a name="morph-to-relationships"></a>
#### Отношения Morph To

Магические методы нельзя использовать для создания отношений `morphTo`. Вместо этого метод `for` должен использоваться напрямую, а имя отношения должно быть явно указано. Например, представьте, что модель `Comment` имеет метод `commentable`, который определяет отношение `morphTo`. В этой ситуации мы можем создать три комментария, относящиеся к одному посту, используя напрямую метод `for`:

    $comments = Comment::factory()->count(3)->for(
        Post::factory(), 'commentable'
    )->create();

<a name="polymorphic-many-to-many-relationships"></a>
#### Полиморфные отношения Many To Many

Полиморфные отношения Many To Many (`morphToMany` / `morphedByMany`) могут быть созданы точно так же, как не полиморфные отношения Many To Many:

    use App\Models\Tag;
    use App\Models\Video;

    $videos = Video::factory()
                ->hasAttached(
                    Tag::factory()->count(3),
                    ['public' => true]
                )
                ->create();

Конечно, магический метод `has` также используется для создания полиморфных отношений Many To Many:

    $videos = Video::factory()
                ->hasTags(3, ['public' => true])
                ->create();

<a name="defining-relationships-within-factories"></a>
### Определение отношений внутри фабрик

Чтобы определить отношение в рамках вашей фабрики модели, вы обычно назначаете новый экземпляр фабрики внешнему ключу отношения. Обычно это делается для «обратных» отношений, таких как `belongsTo` и `morphTo`. Например, если вы хотите создать нового пользователя при создании публикации, вы можете сделать следующее:

    use App\Models\User;

    /**
     * Определить состояние модели по умолчанию.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'user_id' => User::factory(),
            'title' => fake()->title(),
            'content' => fake()->paragraph(),
        ];
    }

Если свойства отношения зависят от фабрики, которая его определяет, вы можете назначить замыкание атрибуту. Замыкание получит массив проанализированных атрибутов фабрики:

    /**
     * Определить состояние модели по умолчанию.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'user_id' => User::factory(),
            'user_type' => function (array $attributes) {
                return User::find($attributes['user_id'])->type;
            },
            'title' => fake()->title(),
            'content' => fake()->paragraph(),
        ];
    }

<a name="recycling-an-existing-model-for-relationships"></a>
### Повторное использование существующей модели для отношений

Если у вас есть модели, которые имеют общее отношение с другой моделью, вы можете использовать метод `recycle`, чтобы обеспечить повторное использование одного экземпляра связанной модели для всех отношений, созданных фабрикой.

Например, представьте, что у вас есть модели `Airline`, `Flight`, и `Ticket`, где билет принадлежит авиакомпании и рейсу, а рейс также принадлежит авиакомпании. При создании билетов вы, вероятно, захотите использовать одну и ту же авиакомпанию как для билета, так и для рейса. Для этого вы можете передать экземпляр авиакомпании методу `recycle`:

    Ticket::factory()
        ->recycle(Airline::factory()->create())
        ->create();

Метод recycle может быть особенно полезен, если у вас есть модели, принадлежащие одному пользователю или команде.

Метод `recycle` также принимает коллекцию существующих моделей. Если методу`recycle` предоставляется коллекция, то, когда фабрике понадобится модель данного типа, из коллекции будет выбрана случайная модель:

    Ticket::factory()
        ->recycle($airlines)
        ->create();
