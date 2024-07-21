# Web Scraping With PHP 

-[Devops do Samba 2024]


- [Installing Prerequisites](#installing-prerequisites)
- [Making an HTTP GET request](#making-an-http-get-request)
- [Web scraping in PHP with Goutte](#web-scraping-in-php-with-goutte)
- [Web scraping with Symfony Panther](#web-scraping-with-symfony-panther)

PHP é uma linguagem de script de uso geral e uma das opções mais populares para desenvolvimento web. Por exemplo, WordPress, 
o sistema de gerenciamento de conteúdo mais comum para criar sites, é construído em PHP.

PHP oferece vários blocos de construção necessários para construir um web scraper, embora possa rapidamente se tornar uma tarefa cada vez mais complicada. 
Convenientemente, existem muitas bibliotecas de código aberto que podem tornar o web scraping com PHP mais acessível.

Este artigo irá guiá-lo através do processo passo a passo de escrever várias rotinas de web scraping em PHP que podem extrair dados públicos 
de páginas da web estáticas e dinâmicas.

Para uma explicação detalhada, consulte nosso [blog post](https://devopsdosamba.com.br).

## Instalando pré-requisitos

```sh
# Windows
choco install php
choco install composer
```

ou 

```sh
# macOS
brew install php
brew install composer
```

## Fazendo uma solicitação HTTP GET de exemplo.

```php
<?php
$html = file_get_contents('https://books.toscrape.com/');
echo $html;
```

## Web scraping no PHP com uma biblioteca chamada Goutte

```sh
composer init --no-interaction --require="php >=7.1"
composer require fabpot/goutte
composer update
```

```php
<?php
require 'vendor/autoload.php';
use Goutte\Client;
$client = new Client();
$crawler = $client->request('GET', 'https://books.toscrape.com');
echo $crawler->html();
```

### Localizando elementos HTML por meio de seletores CSS

```php
echo $crawler->filter('title')->text(); //CSS
echo $crawler->filterXPath('//title')->text(); //XPath
```

### Extraindo os elementos

```php
function scrapePage($url, $client){
    $crawler = $client->request('GET', $url);
    $crawler->filter('.product_pod')->each(function ($node) {
            $title = $node->filter('.image_container img')->attr('alt');
            $price = $node->filter('.price_color')->text();
            echo $title . "-" . $price . PHP_EOL;
        });
    }
```



### Manipulando a paginação

```php
function scrapePage($url, $client, $file)
{
   //...
  // Handling Pagination
    try {
        $next_page = $crawler->filter('.next > a')->attr('href');
    } catch (InvalidArgumentException) { //Next page not found
        return null;
    }
    return "https://books.toscrape.com/catalogue/" . $next_page;
}
```

### Gravando dados em CSV

```php
function scrapePage($url, $client, $file)
{
    $crawler = $client->request('GET', $url);
    $crawler->filter('.product_pod')->each(function ($node) use ($file) {
        $title = $node->filter('.image_container img')->attr('alt');
        $price = $node->filter('.price_color')->text();
        fputcsv($file, [$title, $price]);
    });
    try {
        $next_page = $crawler->filter('.next > a')->attr('href');
    } catch (InvalidArgumentException) { //Next page not found
        return null;
    }
    return "https://books.toscrape.com/catalogue/" . $next_page;
}
$client = new Client();
$file = fopen("books.csv", "a");
$nextUrl = "https://books.toscrape.com/catalogue/page-1.html";
while ($nextUrl) {
    echo "<h2>" . $nextUrl . "</h2>" . PHP_EOL;
    $nextUrl = scrapePage($nextUrl, $client, $file);
}
fclose($file);
```



## Web scraping com Symfony Panther

```sh
composer init --no-interaction --require="php >=7.1" 
composer require symfony/panther
composer update
brew install chromedriver
```

### Enviando solicitações HTTP com Panther

```php
<?php
require 'vendor/autoload.php';
use \Symfony\Component\Panther\Client;
$client = Client::createChromeClient();
$client->get('https://quotes.toscrape.com/js/');
```

### Localizando elementos HTML por meio de seletores CSS

```php
    $crawler = $client->waitFor('.quote');
    $crawler->filter('.quote')->each(function ($node) {
        $author = $node->filter('.author')->text();
        $quote = $node->filter('.text')->text();
       echo $autor." - ".$quote
    });
```

### Manipulando paginação

```php
while (true) {
    $crawler = $client->waitFor('.quote');
…
    try {
        $client->clickLink('Next');
    } catch (Exception) {
        break;
    }
}
```

### Gravando dados em um arquivo CSV

```php
$file = fopen("quotes.csv", "a");
while (true) {
    $crawler = $client->waitFor('.quote');
    $crawler->filter('.quote')->each(function ($node) use ($file) {
        $author = $node->filter('.author')->text();
        $quote = $node->filter('.text')->text();
        fputcsv($file, [$author, $quote]);
    });
    try {
        $client->clickLink('Next');
    } catch (Exception) {
        break;
    }
}
fclose($file);
```



Se você deseja saber mais sobre web scraping com PHP, consulte nosso [blog post](https://devopsdosamba.com.br).
