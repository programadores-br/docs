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

Laravel 6 (LTS) continua as melhorias feitas no Laravel 5.8, introduzindo versionamento semântico, compatibilidade com o [Laravel Vapor](https://vapor.laravel.com), melhorias em respostas de autorizações, middleware de jobs, coleções de Lazy, melhorias de sub-query, retirada dos códigos prontos de frontend para o pacote `laravel/ui` do Composer, e uma variedade de outras correções de segurança e melhorias de usabilidade.

### Versionamento Semântico

O pacote do framework Laravel (`laravel/framework`) agora segue o padrão de [versionamento semântico](https://semver.org/lang/pt-BR/). Isto torna o framework consistente com os outros pacotes primários Laravel que já seguem este padrão de versionamento. O ciclo de lançamento do Laravel permanece inalterado.

### Compatibilidade com Laravel Vapor

_Laravel Vapor foi feito por [Taylor Otwell](https://github.com/taylorotwell)_.

Laravel 6 é compatível com [Laravel Vapor](https://vapor.laravel.com), uma plataforma de deploy com auto-scale serverless para o Laravel. Vapor abstrai a complexidade de gerenciar aplicações Laravel no AWS Lambda, bem como interface das aplicações com filas SQS, bancos de dados, clusters de Redis, redes, CDN CloudFront, e outros.

### Exceções melhoradas via Ignition

Laravel 6 vem com o [Ignition](https://github.com/facade/ignition), uma nova página com detalhes de exceções de código aberto, criado por Freek Van der Herten e Marcel Pociot. Ignition oderece muitos benefícios em relação às versões anteriores, como uma melhoria no gerenciamento dos erros em arquivos do Blade e número da linha, soluções executáveis para problemas comuns, edição de código, compartilhamento das exceções e melhorias de UX.

### Melhorias em Respostas de Autorizações

_Respostas de autorizações melhoradas foram implementadas por [Gary Green](https://github.com/garygreen)_.

Em versões anteriores do Laravel, era difícil recuperar e expor mensagens customizadas de autorização para os usuários finais. Isto tornava difícil explicar para o usuário o porquê exatamente uma requisição particular foi negada. No Laravel 6, agora é muito mais fácil de usar mensagens de resposta de autorização e o novo método `Gate::inspect`. Por exemplo, dada a seguinte política no método:

    /**
     * Determina se o usuário pode visualizar o voo fornecido.
     *
     * @param  \App\User  $user
     * @param  \App\Flight  $flight
     * @return mixed
     */
    public function view(User $user, Flight $flight)
    {
        return $this->deny('Explicação da negativa.');
    }

A resposta da política de autorização e a sua mensagem podem ser facilmente recuperada usando o método `Gate::inspect`:

    $response = Gate::inspect('view', $flight);

    if ($response->allowed()) {
        // Usuário está autorizado a visualizar o voo...
    }

    if ($response->denied()) {
        echo $response->message();
    }

Adicionalmente, estas mensagens customizadas serão automaticamente retornadas para seu frontend ao utilizar os métodos auxiliares como `$this->authorize` ou `Gate::authorize` a partir de suas rotas ou controladores.

### Middleware de Jobs

_Middleware de Jobs foram implementadas por [Taylor Otwell](https://github.com/taylorotwell)_.

O middleware de jobs permite que você possa esconder uma lógica customizada em torno da execução de filas de jobs, reduzindo o boilerplate dentro dos próprios jobs. Por exemplo, nas versões anteriores do Laravel, você poderia esconder a lógica do método `handle` de um job com uma função callback de rate-limited:

    /**
     * Executa o job.
     *
     * @return void
     */
    public function handle()
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Bloqueio obtido...');

            // Lida com o job...
        }, function () {
            //  Não pode obter o bloqueio...

            return $this->release(5);
        });
    }

No Laravel 6, esta lógica pode ser extraída para dentro de middleware de job, permitindo que você matenha o método `handle` do seu job livre de qualquer responsabilidade de fazer rate limiting:


    <?php

    namespace App\Jobs\Middleware;

    use Illuminate\Support\Facades\Redis;

    class RateLimited
    {
        /**
         * Processa o job colocado na fila.
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
                        // Bloqueio obtido...

                        $next($job);
                    }, function () use ($job) {
                        // Não pode obter o bloqueio...

                        $job->release(5);
                    });
        }
    }

Depois de criar o middleware, eles podem ser anexados a um job, retornando de um método `middleware` do job:

    use App\Jobs\Middleware\RateLimited;

    /**
     * Pega o middleware pelo qual o job deve passar.
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited];
    }

### Coleções Lazy

_Coleções Lazy foram implementadas por [Joseph Silber](https://github.com/JosephSilber)_.

Muitos desenvolvedores já aproveitam todo o poder dos  [métodos Collection](https://laravel.com/docs/collections) do Laravel. Para complementar a já poderosa classe `Collection`, o Laravel 6 apresenta um `LazyCollection`, que é To supplement the already powerful  class, Laravel 6 introduces a `LazyCollection`, que se aproveita dos [generators](https://www.php.net/manual/en/language.generators.overview.php) do PHP, para permitir que você trabalhe com conjuntos de dados muito grandes, enquanto mantém o uso de memória baixo.

Por exemplo, imagine que sua aplicação necessita processar um arquivo de log com vários gigabytes, enquanto tira vantagem dos métodos Collection do Laravel para analisar os logs. Ao invés de ler o arquivo inteiro dentro da memória de uma única vez, as Collections Lazy podem ser usadas para manter somente uma pequena parte do arquivo na memória por vez:

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
        // Processa a entrada do log...
    });

Ou então, imagine que você precise iterar sobre 10.000 modelos do Eloquent. Quando usa as tradicionais Collections do Laravel, todos os 10.000 modelos do Eloquent precisam ser carregados para a memória ao mesmo tempo:

    $users = App\User::all()->filter(function ($user) {
        return $user->id > 500;
    });

No entanto, começando no Laravel 6, o método de construtor de queries `cursor` foi atualizado para retornar uma instância de `LazyCollection`. Isto permite a você ainda rodar uma única query no banco de dados, mas também somente manter um modelo do Eloquent carregado na memória por vez. Neste exemplo, o callback `filter` não é executado até nós iterarmos sobre cada usuário invidualmente, permitindo uma drástica redução do uso de memória:

    $users = App\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

### Aprimoramentos de Subquery no Eloquent

_Aprimoramentos de subquery no Eloquent foram implementadas por [Jonathan Reinink](https://github.com/reinink)_.

O Laravel 6 introduziu muitas novas melhorias e aprimoramentos para suporte à subquery no banco de dados. Por exemplo, vamos imaginar que nós temos uma tabela de voos chamadas `destinos` e uma tabela de `voos` para destinos. A tabela `voos` contém uma coluna `chegada_em`, que indica quando o voo chegou no destino.

Usando a nova funcionalidade de select do subquery dentro do Laravel 6, nós podemos selecionar todos os `destinos` e o nome do voo que mais recentemente chegaram ao destino usando uma única consulta:

    return Destination::addSelect(['ultimo_voo' => Flight::select('nome')
        ->whereColumn('id_destino', 'destinos.id')
        ->orderBy('chegada_em', 'desc')
        ->limit(1)
    ])->get();

Adicionalmente, nós podemos usar os novos recursos de subquery em conjunto com a função `orderBy` do construtor de queries para ordenar todos os destinos baseado em quando o último voo chegou em seu destino. De novo, isto pode ser feito executando uma única query para o banco de dados:

    return Destination::orderByDesc(
        Flight::select('chegada_em')
            ->whereColumn('id_destino', 'destinos.id')
            ->orderBy('chegada_em', 'desc')
            ->limit(1)
    )->get();

### Laravel UI

O códigos semi-prontos de frontend, tipicamente fornecidos em versões anteriores do Laravel, foram extraídos para o pacote `laravel/ui` do Composer. Isto permite que o código primário da interface seja desenvolvido e versionado separadamente do framework primário. Como resultado desta mudança, nenhum código Bootstrap ou Vue está presente no código padrão do framework e o comando `make:auth` também foi extraído do framework.

Para restaurar o tradicional código semi-pronto do Vue/Bootstrap como nas versões anteriores do Laravel, basta instalar o pacote `laravel/ui` e usar o comando `ui` do Artisan para instalar o código do frontend:

    composer require laravel/ui --dev

    php artisan ui vue --auth
