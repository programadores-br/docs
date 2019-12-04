# Guia de atualização

- [Atualizando do 5.8 para o 6.0](#upgrade-6.0)

<a name="high-impact-changes"></a>
## Mudanças de Alto Impacto

<div class="content-list" markdown="1">
- [Recursos de Autorização & `viewAny`](#authorized-resources)
- [Helpers de Strings & Arrays](#helpers)
</div>

<a name="medium-impact-changes"></a>
## Mudanças de Médio Impacto

<div class="content-list" markdown="1">
- [Perda de Suporte ao Carbon 1.x](#carbon-support)
- [Cliente padrão do Redis](#redis-default-client)
- [Método `Capsule::table` do Database](#capsule-table)
- [Arrayable & `toArray` Eloquent](#eloquent-to-array)
- [Método `BelongsTo::update` do Eloquent](#belongs-to-update)
- [Tipos de Chave Primária do Eloquent](#eloquent-primary-key-type)
- [Métodos `Lang::trans` e `Lang::transChoice` de Localização](#trans-and-trans-choice)
- [Método `Lang::getFromJson` de Localização](#get-from-json)
- [Limite de retentativas de filas](#queue-retry-limit)
- [Rota para reenvio de e-mail de verificação](#email-verification-route)
- [Alteração da rota de verificação de e-maiç](#email-verification-route-change)
- [A Facade `Input`](#the-input-facade)
</div>

<a name="upgrade-6.0"></a>
## Atualizando do 5.8 para o 6.0

#### Tempo estimado para atualização: uma hora

> {note} Nós tentamos documentar toda mudança que possa quebrar a sua aplicação. Considerando que algumas destas mudanças ocorreram em partes profundas do framework, somente uma parte destas mudanças podem efetivamente afetar sua aplicação.

### PHP 7.2 é requerido

**Probabilidade de impacto: Médio**

PHP 7.1 não será mais matido ativamente a partir de Dezembro/2019. No entanto, o Laravel 6.0 requer PHP 7.2 ou mais recente.

<a name="updating-dependencies"></a>
### Atualizando Dependências

Atualize sua dependência `laravel/framework` para `^6.0` no seu arquivo `composer.json`.

Em seguida, analise quaisquer pacotes de terceiros consumidos por sua aplicação e verifique se você está usando a versão adequada que suporte o Laravel 6.

### Autorização

<a name="authorized-resources"></a>
#### Recursos de Autorização & `viewAny`

**Probabilidade de impacto: Alto**

Políticas de autorização acopladas ao controller usando o método `authorizeResource` devem definir a partir de agora um método `viewAny`, que será chamado quando um usuário acessa método `index` do controller. Do contrário, chamadas ao método `index` a partir do controller serão rejeitadas como não autorizadas.

#### Respostas de Autorização

**Probabilidade de impacto: Baixo**

A assinatura do construtor da classe `Illuminate\Auth\Access\Response` mudou. Você deve atualizar seu código You should update your code adequadamente. Se você não está construindo respostas de autorização manualmente e somente usa instâncias dos métodos `allow` e `deny` dentro de suas políticas, nenhuma mudança é necessária:

    /**
     * Cria uma nova resposta.
     *
     * @param  bool  $allowed
     * @param  string  $message
     * @param  mixed  $code
     * @return void
     */
    public function __construct($allowed, $message = '', $code = null)

#### Respostas retornando "Deny" (Negado)

**Probabilidade de impacto: Baixo**

Em versões anteriores do Laravel, você não precisava retornar o valor do método `deny` a partir dos métodos de políticas, desde que uma exceção fosse lançada imediatamente. Porém, de acordo com a documentação do Laravel, você precisa agora retornar o valor do método `deny` a partir de suas políticas:

    public function update(User $user, Post $post)
    {
        if (! $user->role->isEditor()) {
            return $this->deny("Você precisa ser um editor para editar este post.")
        }

        return $user->id === $post->user_id;
    }

<a name="auth-access-gate-contract"></a>
#### O contrato `Illuminate\Contracts\Auth\Access\Gate`

**Probabilidade de impacto: Baixo**

O contrato `Illuminate\Contracts\Auth\Access\Gate` recebeu um novo método `inspect`. Se você está implementando esta interface manualmente, você deve adicionar este método à sua implementação.
    
### Carbon

<a name="carbon-support"></a>
#### Perda de Suporte ao Carbon 1.x

**Probabilidade de impacto: Médio**

Carbon 1.x [não é mais suportado](https://github.com/laravel/framework/pull/28683), considerando que estamos próximos do fim do ciclo de manutenção dele. Por favor, atualize sua aplicação para o Carbon 2.0.

### Configuração

#### A variável de ambiente `AWS_REGION`

**Probabilidade de impacto: Opcional**

Se vcoê planeja usa o[Laravel Vapor](https://vapor.laravel.com), você deve atualizar todas as ocorrências de `AWS_REGION` dentro do diretório `config` para `AWS_DEFAULT_REGION`. Adicionalmente, você deve atualizar o nome deste variável de ambiente update no seu arquivo `.env`.

<a name="redis-default-client"></a>
#### Cliente padrão do Redis

**Probabilidade de impacto: Médio**

O cliente padrão do Redis foi alterado, de `predis` para `phpredis`. Para continuar usando o `predis`, tenha certeza que a opção `redis.client` está ajustada para `predis` no seu arquivo de configuração `config/database.php`.

### Banco de dados

<a name="capsule-table"></a>
#### O método encapsulado `table`

**Probabilidade de impacto: Médio**

> {note} Esta mudança se aplica apenas a aplicações não-Laravel que estão usando `illuminate/database` como dependência.

A assinatura do método `table` da classe `Illuminate\Database\Capsule\Manager` foi atualizado para aceitar um apelido da tabela como seu segundo argumento. Se você está usando `illuminate/database` fora de uma aplicação Laravel, você deve atualizar adequadamente quaisquer chamadas ao método:

    /**
     * Obtem uma instância fluida do construtor de queries.
     *
     * @param  \Closure|\Illuminate\Database\Query\Builder|string  $table
     * @param  string|null  $as
     * @param  string|null  $connection
     * @return \Illuminate\Database\Query\Builder
     */
    public static function table($table, $as = null, $connection = null)

#### O método `cursor`

**Probabilidade de impacto: Baixo**

O método `cursor` agora retorna uma instância de `Illuminate\Support\LazyCollection` ao invés de um `Generator`. O `LazyCollection` pode ser iterado assim como um gerador:

    $users = App\User::cursor();

    foreach ($users as $user) {
        //
    }

<a name="eloquent"></a>
### Eloquent

<a name="belongs-to-update"></a>
#### O método `BelongsTo::update`

**Probabilidade de impacto: Médio**

Por consistência, o método `update` do relacionamento `BelongsTo` agora age como uma query de atualização ad-hoc, o que significa que ele não fornece proteção contra atribuição massiva nem que dispara eventos do Eloquent. Isto torna o relacionamento dos métodos `update` consistente com todos os outros tipos de relacionamentos.

Se você gostaria de atualizar um modelo acoplado através de um relacionamento `BelongsTo` e receber uma proteção de atribuição massiva de atualização e eventos, você deve atualizar a chamada ao método `update` dentro do próprio modelo:

    // Query ad-hoc... Sem proteção para atribução massiva ou eventos...
    $post->user()->update(['foo' => 'bar']);

    // Atualização do modelo... fornece proteção de atribução massiva e eventos...
    $post->user->update(['foo' => 'bar']);

<a name="eloquent-to-array"></a>
#### Arrayable & `toArray`

**Probabilidade de impacto: Médio**

Os métodos `toArray` dos modelos do Eloquent vão transformar em um array qualquer atributo que implemente `Illuminate\Contracts\Support\Arrayable`.

<a name="eloquent-primary-key-type"></a>
#### Declaração de tipos de Chave Primária

**Probabilidade de impacto: Médio**

Laravel 6.0 recebeu [otimizaçãoes de performance](https://github.com/laravel/framework/pull/28153) para para chaves do tipo inteiro. Se você está usando uma string como chave primária do seu modelo, você deve decalrar o tipo da chave usando a propriedade `$keyType` no seu modelo:

    /**
     * O "tipo" do ID da chave primária.
     *
     * @var string
     */
    protected $keyType = 'string';

### Email Verification

<a name="email-verification-route"></a>
#### Resend Verification Route HTTP Method

**Likelihood Of Impact: Medium**

To prevent possible CSRF attacks, the `email/resend` route registered by the router when using Laravel's built-in email verification has been updated from a `GET` route to a `POST` route. Therefore, you will need to update your frontend to send the proper request type to this route. For example, if you are using the built-in email verification template scaffolding:

    {{ __('Before proceeding, please check your email for a verification link.') }}
    {{ __('If you did not receive the email') }},

    <form class="d-inline" method="POST" action="{{ route('verification.resend') }}">
        @csrf

        <button type="submit" class="btn btn-link p-0 m-0 align-baseline">
            {{ __('click here to request another') }}
        </button>.
    </form>

<a name="mustverifyemail-contract"></a>
#### The `MustVerifyEmail` Contract

**Likelihood Of Impact: Low**

A new `getEmailForVerification` method has been added to the `Illuminate\Contracts\Auth\MustVerifyEmail` contract. If you are manually implementing this contract, you should implement this method. This method should return the object's associated email address. If your `App\User` model is using the `Illuminate\Auth\MustVerifyEmail` trait, no changes are required, as this trait implements this method for you.

<a name="email-verification-route-change"></a>
#### Email Verification Route Change

**Likelihood Of Impact: Medium**

The route path for verifying emails has changed from `/email/verify/{id}` to `/email/verify/{id}/{hash}`. Any email verification emails that were sent prior to upgrading to Laravel 6.x will not longer be valid and will display a 404 page. If you wish, you may define a route matching the old verification URL path and display an informative message for your users that asks them to re-verify their email address.

<a name="helpers"></a>
### Helpers

#### String & Array Helpers Package

**Likelihood Of Impact: High**

All `str_` and `array_` helpers have been moved to the new `laravel/helpers` Composer package and removed from the framework. If desired, you may update all calls to these helpers to use the `Illuminate\Support\Str` and `Illuminate\Support\Arr` classes. Alternatively, you can add the new `laravel/helpers` package to your application to continue using these helpers:

    composer require laravel/helpers

### Localization

<a name="trans-and-trans-choice"></a>
#### The `Lang::trans` & `Lang::transChoice` Methods

**Likelihood Of Impact: Medium**

The `Lang::trans` and `Lang::transChoice` methods of the translator have been renamed to `Lang::get` and `Lang::choice`.

In addition, if you are manually implementing the `Illuminate\Contracts\Translation\Translator` contract, you should update your implementation's `trans` and `transChoice` methods to `get` and `choice`.

<a name="get-from-json"></a>
#### The `Lang::getFromJson` Method

**Likelihood Of Impact: Medium**

The `Lang::get` and `Lang::getFromJson` methods have been consolidated. Calls to the `Lang::getFromJson` method should be updated to call `Lang::get`.

> {note} You should run the `php artisan view:clear` Artisan command to avoid Blade errors related to the removal of `Lang::transChoice`, `Lang::trans`, and `Lang::getFromJson`.

### Mail

#### Mandrill & SparkPost Drivers Removed

**Likelihood Of Impact: Low**

The `mandrill` and `sparkpost` mail drivers have been removed. If you would like to continue using either of these drivers, we encourage you to adopt a community maintained package of your choice that provides the driver.

### Notifications

#### Nexmo Routing Removed

**Likelihood Of Impact: Low**

A lingering part of the Nexmo notification channel was removed from the core of the framework. If you're relying on routing Nexmo notifications you should manually implement the `routeNotificationForNexmo` method on your notifiable entity [as described in the documentation](/docs/{{version}}/notifications#routing-sms-notifications).

### Password Reset

#### Password Validation

**Likelihood Of Impact: Low**

The `PasswordBroker` no longer restricts or validates passwords. Password validation was already being handled by the `ResetPasswordController` class, making the broker's validations redundant and impossible to customize. If you are manually using the `PasswordBroker` (or `Password` facade) outside of the built-in `ResetPasswordController`, you should validate all passwords before passing them to the broker.

### Queues

<a name="queue-retry-limit"></a>
#### Queue Retry Limit

**Likelihood Of Impact: Medium**

In previous releases of Laravel, the `php artisan queue:work` command would retry jobs indefinitely. Beginning with Laravel 6.0, this command will now try a job one time by default. If you would like to force jobs to be tried indefinitely, you may pass `0` to the `--tries` option:

    php artisan queue:work --tries=0

In addition, please ensure your application's database contains a `failed_jobs` table. You can generate a migration for this table using the `queue:failed-table` Artisan command:

    php artisan queue:failed-table

### Requests

<a name="the-input-facade"></a>
#### The `Input` Facade

**Likelihood Of Impact: Medium**

The `Input` facade, which was primarily a duplicate of the `Request` facade, has been removed. If you are using the `Input::get` method, you should now call the `Request::input` method. All other calls to the `Input` facade may simply be updated to use the `Request` facade.

### Scheduling

#### The `between` Method

**Likelihood Of Impact: Low**

In previous releases of Laravel, the scheduler's `between` method exhibited confusing behavior across date boundaries. For example:

    $schedule->command('list')->between('23:00', '4:00');

For most users, the expected behavior of this method would be to run the `list` command every minute for all minutes between 23:00 and 4:00. However, in previous releases of Laravel, the scheduler ran the `list` command every minute between 4:00 and 23:00, essentially swapping the time thresholds. In Laravel 6.0, this behavior has been corrected.

### Storage

<a name="rackspace-storage-driver"></a>
#### Rackspace Storage Driver Removed

**Likelihood Of Impact: Low**

The `rackspace` storage driver has been removed. If you would like to continue using Rackspace as a storage provider, we encourage you to adopt a community maintained package of your choice that provides this driver.

### URL Generation

#### Route URL Generation & Extra Parameters

In previous releases of Laravel, passing associative array parameters to the `route` helper or `URL::route` method would occasionally use these parameters as URI values when generating URLs for routes, even if the parameter value had no matching key within the route path. Beginning in Laravel 6.0, these values will be attached to the query string instead. For example, consider the following route:

    Route::get('/profile/{location}', function ($location = null) {
        //
    })->name('profile');

    // Laravel 5.8: http://example.com/profile/active
    echo route('profile', ['status' => 'active']);

    // Laravel 6.0: http://example.com/profile?status=active
    echo route('profile', ['status' => 'active']);    

The `action` helper and `URL::action` method are also affected by this change:

    Route::get('/profile/{id}', 'ProfileController@show');

    // Laravel 5.8: http://example.com/profile/1
    echo action('ProfileController@show', ['profile' => 1]);

    // Laravel 6.0: http://example.com/profile?profile=1
    echo action('ProfileController@show', ['profile' => 1]);   

### Validation

#### FormRequest `validationData` Method

**Likelihood Of Impact: Low**

The form request's `validationData` method was changed from `protected` to `public`. If you are overriding this method in your implementation, you should update the visibility to `public`.

<a name="miscellaneous"></a>
### Miscellaneous

We also encourage you to view the changes in the `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). While many of these changes are not required, you may wish to keep these files in sync with your application. Some of these changes will be covered in this upgrade guide, but others, such as changes to configuration files or comments, will not be. You can easily view the changes with the [GitHub comparison tool](https://github.com/laravel/laravel/compare/5.8...master) and choose which updates are important to you.
