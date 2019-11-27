# Configuração

- [Introdução](#introduction)
- [Configuração do Ambiente](#environment-configuration)
    - [Tipos de Variáveis de Ambiente](#environment-variable-types)
    - [Recuperando a Configuração do Ambiente](#retrieving-environment-configuration)
    - [Definindo o Ambiente Atual](#determining-the-current-environment)
    - [Ocultando Variáveis de ambiente das paginas de Debug](#hiding-environment-variables-from-debug)
- [Acessando valores de Configurações](#accessing-configuration-values)
- [Cache de Configurações](#configuration-caching)
- [Modo de Manutenção](#maintenance-mode)

<a name="introduction"></a>
## Introdução

Todos os arquivos de configuração do Laravel estão armazenados no diretório `config`. Cada opção esta documentada, então sinta-se a vontade para olhar os arquivos e farmiliarizar-se com as opções disponíveis para voçê.

<a name="environment-configuration"></a>
## Configuração do Ambiente

Muitas vezes, é util ter valores diferentes de configuração de acordo com o ambiente onde a aplicação esta rodando. Por exemplo, você pode querer usar um driver de cache local diferente  daquele que esta no servidor de produção.

Para facilitar, Laravel utiliza a bliblioteca PHP [DotEnv](https://github.com/vlucas/phpdotenv) criada por Vance Lucas. Em uma instalação Laravel limpa, o diretório principal da aplicação vai conter um arquivo `.env.example`. Se você instalou o laravel via Composer, Este arquivo será automaticamente renomeado para `.env`. Caso contrário, você deve renomea-lo manualmente.

Seu arquivo `.env` não deve ser adicionado ao controle de versão de sua aplicação, uma vez que cada desenvolvedor / servidor requer uma configuração diferente do ambiente. Alem disso seria um risco de segurança caso um invasor obtenha acesso ao controle de versão do repositório, uma vez que quaisquer credênciais sensíveis seriam expostas.

Se você estiver desenvolvendo com uma equipe, você pode continuar a incluir um arquivo `.env.example` junto com sua aplicação. Ao colocar valores de espáço reservado no arquivo de configuração, outros desenvolvedores da sua equipe podem ver claramente quais variáveis serão necessáriaspara a executar a aplicação. Voxê também pode criar um arquivo `.env.testing`. Este arquivo vai sobrescrever o arquivo `.env` ao executar testes PHPUnit ou executando comandos Artisan com a opção `--env=testing`.

> {tip} Qualquer variável do arquivo `.env` pode ser substituida por variáveis de ambiente externas, tais como variáveis de ambiente a nivel de servidor ou a nível de sistema.

<a name="environment-variable-types"></a>
### Tipos de Variáveis de Ambiente

Todas as variáveis em seu arquivo `.env` são interpretadas como strings, portanto alguns valores reservados foram criados para permitir que você retorne uma variedade maior de tipos da função  `env()`:

`.env` Value  | `env()` Value
------------- | -------------
true | (bool) true
(true) | (bool) true
false | (bool) false
(false) | (bool) false
empty | (string) ''
(empty) | (string) ''
null | (null) null
(null) | (null) null

Se você precisar definir uma variável de ambiente com um valor que contenha espaços, você pode faze-lo colocando o valor entre aspas duplas.

    APP_NAME="Minha Aplicação"

<a name="retrieving-environment-configuration"></a>
### Recuperando a Configuração do Ambiente

Todas as variáveis listadas são armazenadas na variável super-global `$_ENV` do PHP quando sua aplicação recebe uma requisição. No entanto, você pode usar o helper `env` para recuperar valores dessas variáveis em seu arquivo de configuração. De fato, se você revisar os arquivos de configuração do Laravel, você notará que várias opções que utilizam esse helper:

    'debug' => env('APP_DEBUG', false),

O segundo parâmetro passado para a função `env` é o "valor padrão". Este valor será usado se não existir nenhuma variável de ambiente para a chave especificada.

<a name="determining-the-current-environment"></a>
### Defifindo o ambiente Atual

O ambiente atual da aplicação é definido pela variável `APP_ENV` do arquivo `.env`. Você pode acessar o valor dessa variável pelo método `environment` da [facade](/docs/{{version}}/facades) `App` :

    $environment = App::environment();

Você também pode passar argumentos para o método `environment` para verificar se o ambiente conresponde ao valor passado. O valor vai retornar `true` se o ambiente corresponder a qualquer um dos valores informados:

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment(['local', 'staging'])) {
        // The environment is either local OR staging...
    }

> {tip} The current application environment detection can be overridden by a server-level `APP_ENV` environment variable. This can be useful when you need to share the same application for different environment configurations, so you can set up a given host to match a given environment in your server's configurations.

<a name="hiding-environment-variables-from-debug"></a>
### Hiding Environment Variables From Debug Pages

When an exception is uncaught and the `APP_DEBUG` environment variable is `true`, the debug page will show all environment variables and their contents. In some cases you may want to obscure certain variables. You may do this by updating the `debug_blacklist` option in your `config/app.php` configuration file.

Some variables are available in both the environment variables and the server / request data. Therefore, you may need to blacklist them for both `$_ENV` and `$_SERVER`:

    return [

        // ...

        'debug_blacklist' => [
            '_ENV' => [
                'APP_KEY',
                'DB_PASSWORD',
            ],

            '_SERVER' => [
                'APP_KEY',
                'DB_PASSWORD',
            ],

            '_POST' => [
                'password',
            ],
        ],
    ];

<a name="accessing-configuration-values"></a>
## Accessing Configuration Values

You may easily access your configuration values using the global `config` helper function from anywhere in your application. The configuration values may be accessed using "dot" syntax, which includes the name of the file and option you wish to access. A default value may also be specified and will be returned if the configuration option does not exist:

    $value = config('app.timezone');

To set configuration values at runtime, pass an array to the `config` helper:

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## Configuration Caching

To give your application a speed boost, you should cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which will be loaded quickly by the framework.

You should typically run the `php artisan config:cache` command as part of your production deployment routine. The command should not be run during local development as configuration options will frequently need to be changed during the course of your application's development.

> {note} If you execute the `config:cache` command during your deployment process, you should be sure that you are only calling the `env` function from within your configuration files. Once the configuration has been cached, the `.env` file will not be loaded and all calls to the `env` function will return `null`.

<a name="maintenance-mode"></a>
## Maintenance Mode

When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, a `MaintenanceModeException` will be thrown with a status code of 503.

To enable maintenance mode, execute the `down` Artisan command:

    php artisan down

You may also provide `message` and `retry` options to the `down` command. The `message` value may be used to display or log a custom message, while the `retry` value will be set as the `Retry-After` HTTP header's value:

    php artisan down --message="Upgrading Database" --retry=60

Even while in maintenance mode, specific IP addresses or networks may be allowed to access the application using the command's `allow` option:

    php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16

To disable maintenance mode, use the `up` command:

    php artisan up

> {tip} You may customize the default maintenance mode template by defining your own template at `resources/views/errors/503.blade.php`.

#### Maintenance Mode & Queues

While your application is in maintenance mode, no [queued jobs](/docs/{{version}}/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.

#### Alternatives To Maintenance Mode

Since maintenance mode requires your application to have several seconds of downtime, consider alternatives like [Envoyer](https://envoyer.io) to accomplish zero-downtime deployment with Laravel.
