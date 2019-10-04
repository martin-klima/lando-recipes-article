# Lando - když recept nestačí
Tento článek je zaměřen na konfigurovatelnost a rozšiřitelnost Lando receptů. Je určen pro Lando verze RC2 a vyšší. Pokud vám v článku bude něco chybět, nebo chcete-li článek obohatit o kus vašeho receptu, či ho aktualizovat, na konci článku je informace, jak se zapojit.

# K čemu je Lando

Lando je multiplatformní open-source nástroj pro vývojáře, který s minimálním úsilím postaví komplexní vývojové prostředí pro lokální vývoj. Toto prostředí je vystavěno na technologii Docker, což má mimo jiné výhodu v tom, že si vývojář nemusí zadělat pracovní počítač velkým množstvím softwaru, přestože na velkém množství softwaru vyvíjí.

Lando umožňuje nastavit pro různé projekty různá prostředí a tato prostředí rychle spustit, snadno ovládat, beze stopy zničit a znovu postavit. K tomu Lando posktytuje jednoduché příkazy a konfigurovatelnost, aby ho mohl používat i začátečník.

Lando je pro ty z nás, kdo chceme:

- vytvářet různá vývojová prostředí pro různé projekty
- přenášet konfiguraci vývojového prostředí v týmu přes Git
- automatizovat opakované, složité instalace a nastavování
- vyhnout se masochizmu z přímého používání dockeru nebo docker-compose.

Používáme Lando s tímto cílem: 
„**Vývojáři stačí naklonovat git repozitář a spustit lando start.** 
**Během pár minut by měl mít vše, co potřebuje pro lokální vývoj.**“

# Nejprve základy - často recept stačí

Začněme tím, že nechceme nic speciálního vymýšlet. Potřebujeme si jen “upéct” prostředí, které pro nás už někdo připravil ve formě receptu. Takových receptů je celá řada, např. 

- [Drupal 6](https://docs.devwithlando.io/tutorials/drupal6.html)
- [**Drupal 7**](https://docs.devwithlando.io/tutorials/drupal7.html)
- [**Drupal 8**](https://docs.devwithlando.io/tutorials/drupal8.html)
- [Joomla](https://docs.devwithlando.io/tutorials/joomla.html)
- [Laravel](https://docs.devwithlando.io/tutorials/laravel.html)
- [MEAN -](https://docs.devwithlando.io/tutorials/mean.html) JavaScript software stack for building dynamic web apps
- [LAMP -](https://docs.devwithlando.io/tutorials/lamp.html) vývoj PHP aplikací, Apache + MySQL
- [LEMP -](https://docs.devwithlando.io/tutorials/lemp.html) vývoj PHP aplikací, Nginx + MySQL
- [Pantheon](https://docs.devwithlando.io/tutorials/pantheon.html)
- [WordPress](https://docs.devwithlando.io/tutorials/wordpress.html)

Dejme tomu, že máme chuť vyzkoušet si Drupal 8. Je to jednoduché. Založte si adresář (v mé ukázce je to my_drupal8). Nemusíme podle receptu sami vařit, stačí Landu ukázat, podle jakého výše uvedeného receptu má vařit a dodat pár údajů. 


    marty@ProBook ~ $ mkdir my_drupal8
    marty@ProBook ~ $ cd my_drupal8/
    marty@ProBook ~/my_drupal8 $ lando init
    ? From where should we get your app's codebase? current working directory
    ? What recipe do you want to use? drupal8
    ? Where is your webroot relative to the init destination? web
    ? What do you want to call this app? my-drupal8-web
    
    NOW WE'RE COOKING WITH FIRE!!!
    Your app has been initialized!
    
    Go to the directory where your app was initialized and run `lando start` to get rolling.
    Check the LOCATION printed below if you are unsure where to go.
    
    Oh... and here are some vitals:
    
     NAME      my-drupal8-web                                      
     LOCATION  /home/marty/my_drupal8                              
     RECIPE    drupal8                                             
     DOCS      https://docs.devwithlando.io/tutorials/drupal8.html

Recept zajistil, že máme kromě prostředí i užitečné příkazy. Ty z Drupal8 receptu jsou označeny tučně:

    $ lando
    Usage: lando <command> \[args\] [options]
    
    Commands:
      lando composer          Runs composer commands
      lando config            Displays the lando configuration
      lando db-export [file]  Exports database from a database service to a file
      lando db-import <file>  Imports a dump file into a database service
      lando destroy           Destroys your app
      lando drupal            Runs drupal console commands
      lando drush             Runs drush commands
      lando info              Prints info about your app
      lando init              Initializes code for use with lando
      lando list              Lists all running lando apps and containers
      lando logs              Displays logs for your app
      lando mysql             Drops into a MySQL shell on a database service
      lando php               Runs php commands
      lando poweroff          Spins down all lando related containers
      lando rebuild           Rebuilds your app from scratch, preserving data
      lando restart           Restarts your app
      lando share             Shares your local site publicly
      
      lando ssh               Drops into a shell on a service, runs commands
      lando start             Starts your app
      lando stop              Stops your app
      lando version           Displays the lando version
    
    Options:
      --clear        Clears the lando tasks cache                       [boolean]
      --lando        Show help for lando-based options                  [boolean]
      --verbose, -v  Runs with extra verbosity                            [count]
      --version      Show version number                                [boolean]
    

Aby se upečené prostředí také spustilo, nesmíme zapomenout na 

    lando start

a je hotovo. Máme Composer, Drush, Drupal consoli.

# Když chceme víc - tuníme recept

Dejme tomu, že potřebujeme upéct něco jinak a nebo něco navíc. Máme tyto možnosti:

- přizpůsobit si poskytnutý recept - config
- přidávat a konfigurovat další služby (něco jako přílohy a mrkvový salát) - services
- definovat příkazy a zjednodušující nástroje (jako brčko nebo vidličku) - tooling
- definovat akce pro jednotlivé fáze - events
- automatizovat opakované příkazy a nastavení do instalačních kroků - build
- definovat vlastní služby na nižší úrovni docker-compose (něco jako vypěstovat si vlastní mrkev)

V následujících příkladech vyjdeme z existujícího konfiguračního souboru .lando.yml, který ve své základní podobě vypadá takto:

    name: my-drupal8-web
    recipe: drupal8
    config:
      webroot: web

Důležité je pochopit, že vše, co se vztahuje ke konfiguraci receptu, a tudíž to musí recept podporovat, se nachází pod klíčem “config”.

## Případ 1: Potřebujeme specifickou verzi PHP

V základní verzi máme v tuto chvíli nainstalováno PHP v 7.2.14, MySQL 5,7 a webserver Apache/2.4.25 (Debian). Dejme tomu, že na produkci máme verzi 7.0, nebo 7.1, tak ji chceme mít i lokálně. Na to, jaké verze (resp. docker image) jsou podporované, najdete vždy v [dokumentaci](https://docs.devwithlando.io/tutorials/php.html).


    name: my-drupal8-web
    recipe: drupal8
    config:
      webroot: web
      php: "7.0"

 Verze 7.0 je v uvozovkách, protože YAML by to bez uvozovek přeložil jako 7 a Lando by se nespustilo. Pro verzi 7.1 to můžete nechat bez uvozovek. Aktualizace: V novějších verzích by to už mělo být opraveno a uvozovky nejsou nutné.
 
 Stejně tak je možné konfigurovat i další věci, např. verzi MySQL,

      database: mysql:8.0

nebo typ webserveru, tedy Nginx místo Apache.

      php: "7.2"
      via: nginx  


## Případ 2: Potřebujeme Drush 9 a Xdebug

Nyní si nastavíme, že chceme místo Drush 8 novější Drush 9 a zapnout podporu pro Xdebug.

    name: my-drupal8-web
    recipe: drupal8
    config:
      webroot: web
      php: "7.0"
      drush: 9
      xdebug: true

  
Pozor! Po každé změně konfigurace musíte prostředí znovu postavit (nebo upéct) příkazem

      lando rebuild

  což je novinka od verze RC2.
  
  Kontrola verze Drushe:

      lando drush version
        Drush version : 9.5.2

  
Pokud chcete mít Drush součástí vašeho projektu, máte ho součástí composer.json. V takovém případě bude Lando používat Drush definovaný ve vašem projektu a verzi v .lando.yml bude ignorovat.
  

## Případ 3: Preferujeme MariaDB nebo Postgres místo MySQL
    name: my-drupal8-web
    recipe: drupal8
    config:
      webroot: web
      php: "7.0"
      drush: 9
      xdebug: true
      database: mariadb

Kontrola:

    lando mysql -v
      Server version: 10.1.38-MariaDB Source distribution

Můžete požadovat i Postgres, např.

    config:
      database: postgres:9.6


## Případ 4: Potřebujeme extra konfiguraci pro PHP nebo MySQL

Někdy si potřebujeme nastavit něco speciálního v php.ini nebo v my.cnf. Tu vychytávku, která se má nastavit, dáme do samostatného souboru a řekneme Landu, kde ho najde. V ukázce jsou soubory v adresáři config (adresář config je ve stejném adresáři jako .lando.yml)


    name: my-drupal8-web
    recipe: drupal8
    config:
      webroot: web
      php: "7.0"
      drush: 9
      xdebug: true
      database: mariadb
      config:
        php: config/my-custom-php.ini
        database: config/my-custom.cnf

Taková malá, ne zcela praktická ukázka, co by mohlo být v my-custom-php.ini

    max_execution_time = 180
    date.timezone = Europe/Prague
    memory_limit = 256M
    post_max_size = 99M

Nezapomeneme na 

    lando rebuild -y


## Případ 5: Potřebujeme zachytávat maily

Při vývoji je užitečné zachycovat e-maily v lokálním prostředí, neriskovat že nám nějaký mail odeslaný zákazníkovi cronem unikne do internetu. K tomu slouží různé služby a Lando nabízí Maihog a my si tuto službu přidáme k ostatním.
Zatím jsme konfigurovali recept. Nyní přidáváme další službu, resp. další Docker container. Služby jsou definovány pod klíčem “services”.  Pojmenovat si je můžete libovolně, takže místo  “mailhog:” můžete mít třeba “mailserver:”.

    services:
      mailhog:
        type: mailhog
        hogfrom:
          - appserver      

Nyní všechny maily z PHP skončí ve falešné mailové schránce Mailhogu, kterou navštívíme přes webové rozhraní. Můžete snadno otestovat.

     lando php -r "mail('test@example.com', 'subject', 'message');"

Adresa na webové rozhraní Mailhogu se objevila po dokončení příkazu:

    lando start

nebo ji najdeme přes:

    lando info


## Případ 6: Je libo PhpMyAdmin?
    services:
      pma:
        type: phpmyadmin
        hosts:
          - database


## Případ 7: Node+SASS atd.
    services:
      node:
        type: node:10
        globals:
          gulp-cli: "latest"
          gulp: "latest"


## Případ 8: Potřebujeme kompilovat Compass SASS na CSS.

Někdy máme projekt, který nepoužívá Grunt ani Gulp a cílem je konvertovat SASS na CSS. Pak se může hodit tato odlehčená verze. Posilujeme naši odvahu a použijeme “build”, kam uložíme všechny kroky, které se mají vykonat při vytváření této služby.

    services:
      compass:
        type: ruby
        build: gem update --system && gem install sass compass bootstrap-sass breakpoint
           


## Případ 9: Potřebujeme Solr server
    services:
      solr:
        type: solr:6.6
        core: my_core
        portforward: 8983
        overrides:
          environment:
            HTTPS_METHOD: redirect
        config:
          dir: config/solr
## Případ 10: Chceme mít kód podle standardů - PHP Code sniffer - phpcs

V tompo případě si ukážeme dvě nové věci. Za prvé vytvoříme vlastní novou službu z vybraného existujícího image, který se nachází na Docker hubu (doposud jsme používali jen ty doporučené v Landu).  Službu pojmenujeme “phpcs” a dáme ji typ “compose”. Command “tail -f /dev/null” nikdy neskončí a udržuje tím container v chodu a naši službu aktivní.

    services:
      phpcs:
        type: compose
        services:
          image: willhallonline/drupal-phpcs:alpine
          command: tail -f /dev/null

Za druhé si ukážeme tooling, což je způsob, jak definovat vlastní příkazy. Zde definujeme příkaz “phpcs”, předáváme ho i s jeho parametry složbě phpcs a spustíme ho tam jako uživatel root.

    tooling:
      phpcs:
        service: phpcs
        user: root    

Pro kontrolu PHP kódu pak stačí jen zavolat:

    lando phpcs cesta_k_souboru_či_adresáři
    

## Případ 11: Potřebujeme programátorský editor (IDE) bez instalace

Není to příliš běžné, že by vývojář neměl nainstalovaný editor, ale např. při školení se to stát může. V takovém případě stačí přidat následující službu, která promění internetový prohlížeč na portu 8081 v profesionální editor pro programátory. Port si můžeme samozřejmě změnit.

    services:
      ide:
        type: compose
        services:
          image: martinklima/cloud9-with-codeintel:1.0
          ports:
           - '8081:80'
          command: "node /cloud9/server.js -p 80 -l 0.0.0.0 -w /app -a :"


## Případ 12: Chci mít zachovanou historii příkazů v kontejneru

Po restartu Landa, resp. po novém startu docker kontejneru zmizí historie příkazů, které po připojení přes `lando ssh` do kontejneru zapíšete. Následující trik namapuje historii do souboru `.appserver_bash_history.txt` v adresáři projektu.

    services:
      appserver:
        overrides:
          volumes:
            - .appserver_bash_history.txt:/var/www/.bash_history
          environment:
            PROMPT_COMMAND: "history -a"


## Případ 13: Chci rychle zapínat a vypínat Xdebug

Mít trvale zapnutý Xdebug znamená mít trvale pomalejší PHP. Proto je dobré si ho zapnout, jen když je to potřeba. Zdroj tohoto nápadu je zde: https://github.com/lando/lando/issues/1668#issuecomment-507191275

Pro nginx:

```yaml
tooling:
  xdebug-on:
    service: appserver
    description: Enable xdebug for nginx.
    cmd: docker-php-ext-enable xdebug && pkill -o -USR2 php-fpm
    user: root
  xdebug-off:
    service: appserver
    description: Disable xdebug for nginx.
    cmd: rm /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && pkill -o -USR2 php-fpm
    user: root
```

Pro Apache:
```yaml
tooling:
  xdebug-on:
    service: appserver
    description: Enable xdebug for apache.
    cmd: "docker-php-ext-enable xdebug && /etc/init.d/apache2 reload"
    user: root
  xdebug-off:
    service: appserver
    description: Disable xdebug for apache.
    cmd: "rm /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && /etc/init.d/apache2 reload"
    user: root

```

# Případ 14: Potřebuji z PHP komunikovat s Microsoft SQL serverem

PHP standardně neobsahuje rozšíření pro MS SQL server. Zde je postup, jak nainstalovat PHP extensions `pdo_sqlsrv` a `sqlsrv` do PHP 7.2.

```yaml
services:
  appserver:
    type: php:7.2
    overrides:
      environment:
        ACCEPT_EULA: "Y"
    build_as_root:
      - apt-get update -y
      - apt-get install apt-transport-https -y
      - curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
      - curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
      - apt-get update -y
      - apt-get install msodbcsql17 -y
      - apt-get install unixodbc-dev -y
      - pecl install sqlsrv
      - pecl install pdo_sqlsrv
      - docker-php-ext-enable sqlsrv
      - docker-php-ext-enable pdo_sqlsrv
``` 

# Předzávěr

A kdyby toto vše, co je přednastaveno v receptech a v základních konfiguracích někomu nestačilo, vždy je možné jít o úroveň níže, vytvořit vlastní docker-compose.yml a předat ho Landu. Zájemce o více informací odkazuji na [dokumentaci.](https://docs.devwithlando.io/)


# Závěr

Tento článek obsahuje moji přednášku z Drupal setkání navazujícího na Drupal trénink v únoru roku 2019. Je volným pokračováním o rok starší přednášky a článku [Lando - vývojové protředí s Dockerem pro normální lidi](https://www.drupal.cz/clanky/lando-vyvojove-protredi-s-dockerem-pro-normalni-lidi). Reflektuje rozsáhlé změny mezi verzemi RC1 a RC2 a tedy oproti staršímu článku odpovídá realitě prvního čtvrtletí roku 2019.

