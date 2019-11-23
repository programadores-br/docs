# Notas de Lançamento

- [Padrão de Versionamento](#versioning-scheme)
- [Política de Suporte](#support-policy)
- [Laravel 6](#laravel-6)

<a name="versioning-scheme"></a>
## Padrão de Versionamento

Laravel e seus outros pacotes primários seguem o [Versionamento Semântico](https://semver.org/lang/pt-BR/). Lançamentos maiores são feitos a cada seis meses (Fevereiro e Agosto), enquanto versões menores e corretivas são lançadas, geralmente, a cada semana. Versões menores e corretivas **nunca** devem conter mudanças bruscas.

Quando se referenciar ao framework Laravel ou seus componentes a partir da sua aplicação ou pacote, você devem sempre usar uma restrição de versão, tal como `^6.0`, já que os lançamentos principais do Laravel incluem mudanças bruscas. Porém, nós nos esforçamos para sempre garantir que você possa atualizar para uma versão principal em um dia ou menos

<a name="support-policy"></a>
## Política de Suporte

Para versões LTS (Longo Tempo de Suporte - Long-Term Support, em inglês), tal como o Laravel 6, correções de bugs são fornecidos por 2 anos e correções de segurança são fornecidas por 3 anos. Estes lançamentos provêm uma maior janela de suporte e manutenção. Para lançamentos em geral, correções de bugs são fornecidos por 6 meses e correções de segurança são fornecidos por 1 ano. Para todas as bibliotecas adicionais, incluindo o Lumen, somnete a versão mais recente recebem correções de segurança. Para outros casos, consulte as versões de banco de dados [mantido pelo Laravel](/docs/{{version}}/database#introduction).

| Versão | Data de lançamento | Correções de Bugs até | Correções de Segurança até |
| --- | --- | --- | --- |
| 5.5 (LTS) | 30 de agosto de 2017 | 30 de agosto de 2019 | 30 de agosto de 2020 |
| 5.6 | 7 de fevereiro de 2018 | 7 de agosto de 2018 | 7 de fevereiro de 2019 |
| 5.7 | 4 de setembro de 2018 | 4 de março de 2019 | 4 de setembro de 2019 |
| 5.8 | 26 de fevereiro de 2019 | 26 de agosto de 2019 | 26 de fevereiro de 2020 |
| 6 (LTS) | 3 de setembro de 2019 | 3 de setembro de 2021 | 3 de setembro de 2022 |

<a name="laravel-6"></a>
## Laravel 6

Laravel 6 (LTS) continua as melhorias feitas no Laravel 5.8, introduzindo versionamento semântico, compatibilidade com o [Laravel Vapor](https://vapor.laravel.com), melhorias em respostas de autorizações, middlewares de jobs, coleções de lazy-loading, melhorias de sub-query, retirada dos códigos de frontend para o pacote `laravel/ui` do Composer, e uma variedade de outras correções de segurança e melhorias de usabilidade.

### Versionamento Semântico

O pacote do framework Laravel (`laravel/framework`) agora segue o padrão de [versionamento semântico](https://semver.org/lang/pt-BR/). Isto torna o framework consistente com os outros pacotes primários Laravel que já seguem este padrão de versionamento. O ciclo de lançamento do Laravel permanece inalterado.

### Compatibilidade com Laravel Vapor

_Laravel Vapor foi feito por [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel 6 é compatível com [Laravel Vapor](https://vapor.laravel.com), uma plataforma de deploy com auto-scale serverless para o Laravel. Vapor abstrai a complexidade de gerenciar aplicações Laravel no AWS Lambda, bem como interface das aplicações com filas SQS, bancos de dados, clusters de Redis, redes, CDN CloudFront, e outros.

### Exceções melhoradas via Ignition

Laravel 6 vem com o [Ignition](https://github.com/facade/ignition), uma nova página com detalhes de exceções de código aberto, criado por Freek Van der Herten e Marcel Pociot. Ignition oderece muitos benefícios em relação às versões anteriores, como uma melhoria no gerenciamento dos erros em arquivos do Blade e número da linha, soluções executáveis para problemas comuns, edição de código, compartilhamento das exceções e melhorias de UX.

### Improved Authorization Responses

_Improved authorization responses were implemented by [Gary Green](https://github.com/garygreen)_.

In previous releases of Laravel, it was difficult to retrieve and expose custom authorization messages to end users. This made it difficult to explain to end-users exactly why a particular request was denied. In Laravel 6, this is now much easier using authorization response messages and the new `Gate::inspect` method. For example, given the following policy method:

    /**
     * Determine if the user can view the given flight.
     *
     * @param  \App\User  $user
     * @param  \App\Flight  $flight
     * @return mixed
     */
    public function view(User $user, Flight $flight)
    {
        return $this->deny('Explanation of denial.');
    }

The authorization policy's response and message may be easily retrieved using the `Gate::inspect` method:

    $response = Gate::inspect('view', $flight);

    if ($response->allowed()) {
        // User is authorized to view the flight...
    }

    if ($response->denied()) {
        echo $response->message();
    }

In addition, these custom messages will automatically be returned to your frontend when using helper methods such as `$this->authorize` or `Gate::authorize` from your routes or controllers.

### Job Middleware

_Job middleware were implemented by [Taylor Otwell](https://github.com/taylorotwell)_.

Job middleware allow you to wrap custom logic around the execution of queued jobs, reducing boilerplate in the jobs themselves. For example, in previous releases of Laravel, you may have wrapped the logic of a job's `handle` method within a rate-limited callback:

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Lock obtained...');

            // Handle job...
        }, function () {
            // Could not obtain lock...

            return $this->release(5);
        });
    }

In Laravel 6, this logic may be extracted into a job middleware, allowing you to keep your job's `handle` method free of any rate limiting responsibilities:

    <?php

    namespace App\Jobs\Middleware;

    use Illuminate\Support\Facades\Redis;

    class RateLimited
    {
        /**
         * Process the queued job.
         *
         * @param  mixed  $job
         * @param  callable  $next
         * @return mixed
         */
        public function handle($job, $next)
        {
            Redis::throttle('key')
                    ->block(0)->allow(1)->every(5)
                    ->then(function () use ($job, $next) {
                        // Lock obtained...

                        $next($job);
                    }, function () use ($job) {
                        // Could not obtain lock...

                        $job->release(5);
                    });
        }
    }

After creating middleware, they may be attached to a job by returning them from the job's `middleware` method:

    use App\Jobs\Middleware\RateLimited;

    /**
     * Get the middleware the job should pass through.
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited];
    }

### Lazy Collections

_Lazy collections were implemented by [Joseph Silber](https://github.com/JosephSilber)_.

Many developers already enjoy Laravel's powerful [Collection methods](https://laravel.com/docs/collections). To supplement the already powerful `Collection` class, Laravel 6 introduces a `LazyCollection`, which leverages PHP's [generators](https://www.php.net/manual/en/language.generators.overview.php) to allow you to work with very large datasets while keeping memory usage low.

For example, imagine your application needs to process a multi-gigabyte log file while taking advantage of Laravel's collection methods to parse the logs. Instead of reading the entire file into memory at once, lazy collections may be used to keep only a small part of the file in memory at a given time:

    use App\LogEntry;
    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    })
    ->chunk(4)
    ->map(function ($lines) {
        return LogEntry::fromLines($lines);
    })
    ->each(function (LogEntry $logEntry) {
        // Process the log entry...
    });

Or, imagine you need to iterate through 10,000 Eloquent models. When using traditional Laravel collections, all 10,000 Eloquent models must be loaded into memory at the same time:

    $users = App\User::all()->filter(function ($user) {
        return $user->id > 500;
    });

However, beginning in Laravel 6, the query builder's `cursor` method has been updated to return a `LazyCollection` instance. This allows you to still only run a single query against the database but also only keep one Eloquent model loaded in memory at a time. In this example, the `filter` callback is not executed until we actually iterate over each user individually, allowing for a drastic reduction in memory usage:

    $users = App\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

### Eloquent Subquery Enhancements

_Eloquent subquery enhancements were implemented by [Jonathan Reinink](https://github.com/reinink)_.

Laravel 6 introduces several new enhancements and improvements to database subquery support. For example, let's imagine that we have a table of flight `destinations` and a table of `flights` to destinations. The `flights` table contains an `arrived_at` column which indicates when the flight arrived at the destination.

Using the new subquery select functionality in Laravel 6, we can select all of the `destinations` and the name of the flight that most recently arrived at that destination using a single query:

    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();

In addition, we can use new subquery features added to the query builder's `orderBy` function to sort all destinations based on when the last flight arrived at that destination. Again, this may be done while executing a single query against the database:

    return Destination::orderByDesc(
        Flight::select('arrived_at')
            ->whereColumn('destination_id', 'destinations.id')
            ->orderBy('arrived_at', 'desc')
            ->limit(1)
    )->get();

### Laravel UI

The frontend scaffolding typically provided with previous releases of Laravel has been extracted into a `laravel/ui` Composer package. This allows the first-party UI scaffolding to be developed and versioned separately from the primary framework. As a result of this change, no Bootstrap or Vue code is present in default framework scaffolding, and the `make:auth` command has been extracted from the framework as well.

In order to restore the traditional Vue / Bootstrap scaffolding present in previous releases of Laravel, you may install the `laravel/ui` package and use the `ui` Artisan command to install the frontend scaffolding:

    composer require laravel/ui --dev

    php artisan ui vue --auth
