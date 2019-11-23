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

O framework Laravel possui alguns requisitos de sistema. Todos estes requisitos são satisfeitos pela máquina virtual [Laravel Homestead](/docs/{{version}}/homestead), portanto é altamente recomendado que você use o Homestead como seu ambiente local de desenvolvimento Laravel.

Contudo, se você não usa o Homestead, você precisará que seu servidor atenda aos seguintes requisitos:

<div class="content-list" markdown="1">
- PHP >= 7.2.0
- Extensão BCMath PHP
- Extensão Ctype PHP 
- Extensão JSON PHP
- Extensão Mbstring PHP
- Extensão OpenSSL PHP
- Extensão PDO PHP
- Extensão Tokenizer PHP
- Extensão XML PHP
</div>

<a name="installing-laravel"></a>
### Instalando o Laravel

O Laravel utiliza o [Composer](https://getcomposer.org) para gerenciar suas dependências. Portanto, antes de usar o Laravel, certifique-se de que o Composer esteja instalado em sua máquina.

#### Via Laravel Installer

Primeiramente, faça o download do Laravel Installer usando o Composer:

    composer global require laravel/installer

Certifique-se de que o diretório `vendor bin` do Composer esteja no `$PATH` do seu Sistema Operacional, para que o executável laravel seja localizado pelo seu sistema. Este diretório existe em diferentes localizações de acordo com o seu sistema operacional; no entanto, alguns locais comuns incluem:

<div class="content-list" markdown="1">
- macOS and GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin`
- Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
</div>

Uma vez instalado, o comando `laravel new` irá criar uma instalação limpa do Laravel no diretório especificado. Por exemplo, `laravel new blog` criará um diretorio chamado `blog` contendo uma instalação limpa do Laravel com todas as suas depemndências ja instaladas:

    laravel new blog

#### Via Composer Create-Project

Como alternativa, você também pode instalar o Laravel por meio do comando `create-project` do Composer em seu terminal:

    composer create-project --prefer-dist laravel/laravel blog

#### Servidor de desenvolvimento local

Se você possui o PHP instalado localmente e quiser usar o servidor de desenvolvimento embutido do PHP para rodar sua aplicação, você pode usar o comando `serve` do Artisan. Este comando vai iniciar o servidor de desenvolvimento em `http://localhost:8000`:

    php artisan serve

Ambientes de desenvolvimento mais robustos estão disponíveis via [Homestead](/docs/{{version}}/homestead) e [Valet](/docs/{{version}}/valet).

<a name="configuration"></a>
### Configuração

#### Diretório public

Antes de instalar o Laravel, você precisa configurar a pasta raiz `/` do seu servidor web para ser a pasta `public`. O arquivo `index.php` neste diretório gerencia todas as requisições HTTP que entram na sua aplicação.

#### Arquivos de Configuração

Todos os arquivos de configuração do framework Laravel estão localizados na pasta `config`. Cada uma das opções está documentada, fique à vontade para explorar os arquivos e se familiarizar com as opções disponíveis.

#### Permissões de Diretório

Após instalar o Laravel, pode ser necesspario configurar algumas permissões. Os diretórios dentro de `storage` e os diretórios `bootstrap/cache` devem ter permissão de escrita pelo servidor web; senão o Laravel não funcionará. Se estiver usando a máquina virtual [Homestead](/docs/{{version}}/homestead), essas permissões ja devem estar definidas.

#### Chave da Aplicação

A proxima etapa que você deve fazer após instalar o Laravel é definir a chave de sua aplicação com uma sequencia aleatória. Se você instalou o Laravel via Composer ou Laravel installer, essa chave ja foi definida para você com o comando `php artisan key:generate`.

Normalmente, essa string deve ter um tamanho de 32 caracteres. A chave é definida no arquivo `.env`. Se você não copiou o arquivo `.env.example` para um novo arquivo chamado `.env`, faça isso agora. **Se a chave da aplicação não for definida, as sessões dos usuários e outros dados criptografados não estarão seguros!**

#### Configuração Adicional

Laravel não necessita de nenhuma outra configuração para seu uso. Você é livre para começar a desenvolver! Contudo, você pode querer revisar o arquivo `config/app.php` e esta documentação. Ele contém diversas opções, como `timezone` e `locale`, que você pode desejar modificar de acordo com sua aplicação.

Você também pode querer configurar alguns componentes adicionais do Laravel, tais como:

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Banco de dados](/docs/{{version}}/database#configuration)
- [Sessão](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>
## Configuração do Servidor Web

<a name="directory-configuration"></a>
### Configuração de diretório

O Laravel deve sempre ser disponibilizado fora da raiz do "diretório web" padrão configurado para seu servidor web. Você não deve tentar disponibilizar uma aplicação Laravel a partir de um sub-diretório do "diretório web" do servidor. Tentar fazer isso pode expor arquivos sensíveis presentes em sua aplicação.

<a name="pretty-urls"></a>
### URLs Amigáveis

#### Apache

O Laravel inclui um arquivo `public/.htaccess`, usado para fornecer URLS sem usar `index.php` em seu caminho. Antes de disponibilizar o Laravel com o Apache, certifique-se de habilitar o módulo `mod_rewrite`, para que então o arquivo `.htaccess` seja aceito pelo servidor.

Se o arquivo `.htaccess` que acompanha o Laravel não funcionar com sua instalação do Apache, tente esta alternativa:

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

Ao usar [Homestead](/docs/{{version}}/homestead) ou [Valet](/docs/{{version}}/valet), as URLs amigáveis serão automaticamente configuradas.
