# Instalação

- [Instalação](#installation)
    - [Requisitos do servidor](#server-requirements)
    - [Instalando o Laravel](#installing-laravel)
    - [Configuração](#configuration)
- [Configuração do Web Server](#web-server-configuration)
    - [Configuração de diretório](#directory-configuration)
    - [URLs Amigáveis](#pretty-urls)

<a name="installation"></a>
## Instalação

<a name="server-requirements"></a>
### Requisitos do Servidor

O framework Laravel possui alguns requisitos de sistema. Todos estes requisitos são satisfeitos pelo [Laravel Homestead](/docs/{{version}}/homestead) virtual machine, portanto é altamente recomendado que você use o Homestead como seu ambiente local de desenvolvimento Laravel.

Contudo, se você não usa o Homestead, você precisará que seu servidor atenda aos seguintes requisitos:

<div class="content-list" markdown="1">
- PHP >= 7.2.0
- BCMath PHP Extension
- Ctype PHP Extension
- JSON PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PDO PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
</div>

<a name="installing-laravel"></a>
### Instalando o Laravel

Laravel utiliza o [Composer](https://getcomposer.org) para gerenciar suas dependências. Portanto, antes de usar o Laravel, certifique-se de que o composer esteja instalado em sua maquina.

#### Via Laravel Installer

Primeiramente, faça o download do Laravel Installer usando o composer:

    composer global require laravel/installer

Certifique-se de que o diretório vendor do composer esteja no `$PATH` para que o executável laravel seja localizado pelo seu sistema. Este diretório existe em diferentes localizações de acordo com o seu sistema operacional; no entanto alguns locais comuns incluem:

<div class="content-list" markdown="1">
- macOS and GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin`
- Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
</div>

Uma vez instalado, o comando `laravel new` irá criar uma instalação limpa do Laravel no diretório especificado. Por exemplo, `laravel new blog` criará um diretorio chamado `blog` contendo uma instalação limpa do Laravel com todas as suas depemndências ja instaladas:

    laravel new blog

#### Via Composer Create-Project

Como alternativa você também pode instalar o Laravel por meio do comando `create-project` do composer em seu terminal:

    composer create-project --prefer-dist laravel/laravel blog

#### Servidor de desenvolvimento local
<!-- Se você tiver o PHP instalado e quiser usar o servidor embutido para rodar sua aplicação -->
Se você possui o PHP instalado localmente e quiser usar o servidor de desenvolvimento interno do PHP para rodar sua aplicação, você pode usar o comando `serve` do Artisan. Este comando vai iniciar o servidor de desenvolviemnto em `http://localhost:8000`:

    php artisan serve

Ambientes de desenvolvimento mais robustos estão disponíveis via [Homestead](/docs/{{version}}/homestead) e [Valet](/docs/{{version}}/valet).

<a name="configuration"></a>
### Configuração

#### Diretório public

Antes de instalar o Laravel, você precisa configurar a pasta raiz / do seu servidor web para ser a pasta `public`. O arquivo `index.php` neste diretório gerencia todas as requisições HTTP que da sua aplicação.

#### Arquivos de Configuração

Todos os arquivos de configurração do Laravel framework estão localizados na pasta `config`. Cada uma dfas opções esta documentada, fique à vontade para explorar os arquivos e se familiarizar com as opções disponíveis.

#### Pertmissões de Diretório

Após instalar o Laravel, pode ser necesspario configurar algumas permissões. Diretórios dentro de `storage` e os diretórios `bootstrap/cache` devem ter permissão de escrita pelo servidor web ou o Laravel não funcionara. Se estiver usando o [Homestead](/docs/{{version}}/homestead) virtual machine, Essas permissões ja devem estar definidas.

#### Chave da Aplicação

A proxima etapa que você deve fazer após instalar o Laravel é definir a chave de sua aplicação com uma sequencia aleatória. Se você instalou o laravel via composer ou Laravel installer, essa chave ja foi definida para você com o comando `php artisan key:generate`.

Normalmente, essa string deve ter 32 caracteres. A chave é definida no arquivo `.env`. Se você não copiou o arquivo `.env.example` file para um novo arquivo chamado `.env`, faça isso agora. **Se a chave da aplicação não for definida, suas user sessions e outros dados criptografados não estarão seguros!**

#### Configuração Adicional

Laravel não necessita de nenhuma outra configuração para seu uso. Você é livre para começar a desenvolver! Contudo, você pode rever o arquivo `config/app.php` e esta documentação. ele contem diversas opções como `timezone` e `locale` que você pode modificar de acordo com sua aplicação.

Você também pode configurar alguns componentes adicionas do Laravel, tais como:

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>
## Configuração do Servidor Web

<a name="directory-configuration"></a>
### Configuração de diretório

O Laravel deve sempre ser disponibilizado fora do "diretório web" configurado pelo seu servidor web. Você não deve disponibilizar uma aplicação Laravel a partir de um sub-diretório do "diretorio web" do servidor. Tentar fazer isso pode expor arquivos sensíveis presentes em sua aplicação.

<a name="pretty-urls"></a>
### URLs Amigáveis

#### Apache

Laravel inclui um arquivo `public/.htaccess` usado para fornecer URLS sem o `index.php` em seu path. Antes de disponibilizar  o Laravel com o apache, certifique-se de habilitar o módulo `mod_rewrite`  então o arquivo `.htaccess` seja aceito pelo servidor.

Se o arquivo `.htaccess` que acompanha o Laravel não funcionar com sua instalação do Apache, Tente esta alternativa:

    Options +FollowSymLinks -Indexes
    RewriteEngine On

    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

Caso esteja usando Nginx, a seguinte diretiva na configuração do seu site vai direcionar todas as requisições para o `index.php` principal:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

Ao usar [Homestead](/docs/{{version}}/homestead) ou [Valet](/docs/{{version}}/valet), as URLs amigaveis serão automaticamente configuradas.
