# Popis projektu

CRM Manager je bezplatná aplikace - **návrhář databáze** s otevřeným zdrojovým kódem, určená pro instalaci na váš vlastní lokální server nebo internetový server s podporou PHP/MySQL. Zakladatelem projektu je Sergej Kharchishin.

Projekt je rozdělen na dvě části: hlavní (zdarma) a [rozšíření](https://www.rukovoditel.net.ru/extension.php). V hlavní části naleznete sadu nástrojů pro návrh a přizpůsobení vaší aplikace.

[Rozšíření](https://www.rukovoditel.net.ru/extension.php) obsahuje sadu přehledů a nástrojů pro efektivnější plánování a správu.

[Rozšíření](https://www.rukovoditel.net.ru/extension.php) je placené rozšíření projektu a není součástí tohoto obrázku.

# Zdroje aplikace

* **Oficiální webové stránky:** [Rukovoditel.net.ru](https://www.rukovoditel.net.ru/)
* **Dokumentace:** [Oficiální dokumentace CRM Manager](https://docs.rukovoditel.net.ru/)
* **Odpovědi na otázky:** [Oficiální fórum](https://forum.rukovoditel.net.ru)
* **Telegramová skupina:** [Neoficiální komunitní správce CRM](https://t.me/crm_rukovoditel)
* **Úložiště GitHub** [Neoficiální správce CRM v Dockeru](https://github.com/registriren/Rukovoditel)

# Podporované verze aplikací v Dockerfile

* [3.5.1 nejnovější](https://github.com/opravmito/Rukovoditel/blob/master/3.5.1/Dockerfile)

# Jak používat image

1. Vytvoříme společnou síť pro databázi a aplikaci Manager.
2. Vytvořte databázový kontejner.
3. Vytvořte správce aplikačního kontejneru (webového serveru).
4. Provedeme prvotní nastavení Správce web serveru, vytvoříme spojení s databází.
5. Odeberte instalační program (adresář `install`) ze svazku, na kterém je nasazena aplikace Manager.
6. Organizujeme zálohování dat.

## Vytvoření sítě:

Než vytvoříte kontejnery, budete potřebovat společnou síť, se kterou bude webový server a databázové kontejnery komunikovat.

Vytvořte místní síť Docker:

```
docker network vytvořit RukovoditelNET
```

## Vytvořte databázový kontejner:

Ke správě dat potřebujete databázi a k ​​ukládání dat potřebujete trvalé úložiště. V tomto případě jde o svazek MariaDB a Docker. Můžete si vybrat jinou databázi, například MySQL, nebo uložit data do složky na hostitelském počítači.

Vytvořte databázový kontejner MariaDB a svazek Docker RukovoditelDB pro uložení dat:

```
docker run -d -p 3306:3306 --rm --name mariadb --network RukovoditelNET -v RukovoditelDB:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_USER=ruser -e MYSQL_PASSWORD=secret_ATADASEitel -e mariadb
```

Hodnotu proměnných `MYSQL_ROOT_PASSWORD=secret, MYSQL_USER=user, MYSQL_PASSWORD=secret, MYSQL_DATABASE=rukovoditel` lze nastavit nezávisle. V budoucnu je budete potřebovat při prvním nastavování webového serveru.

## Vytvoření kontejneru webového serveru Leader:

Nakonec, když jsou síť a databáze připraveny, můžete spustit kontejner aplikace Manager.

Vytvořte kontejner s názvem RukovoditelWEB, svazek Docker pro ukládání dat a připojte jej k síti:

```
docker run -dit --rm --name rukovoditel --network RukovoditelNET -v RukovoditelWEB:/var/www/html -p 8008:80 registriren/rukovoditel
```

Hodnota portu `8008` musí být nastavena nezávisle, s ohledem na fungující služby na vašem serveru.

## Spuštění a počáteční konfigurace správce webového serveru:

Zadejte do prohlížeče název domény nebo IP adresu vašeho serveru, na kterém je aplikace Leader nasazena, například `http://localhost:8080`

Vyplňte pole pro připojení k databázi pomocí informací uvedených v proměnných `MYSQL_USER=uživatel, MYSQL_PASSWORD=secret, MYSQL_DATABASE=rukovoditel`

Zadejte podrobnosti o svém administrátorovi. Připraveno! Můžete začít budovat svou databázovou aplikaci.

## Odebrání instalačního programu:

Po instalaci a počáteční konfiguraci Správce webového serveru odstraňte adresář `install`:

```
docker exec rukovoditel /bin/rm -r /var/www/html/install
```

## Zálohování a obnova dat:

### Vytvořte výpis databáze

Většina obvyklých nástrojů bude fungovat, i když v některých případech může být jejich použití trochu matoucí, aby bylo zajištěno, že budou mít přístup k serveru mysqld. Snadný způsob, jak to ověřit, je použít docker exec a spustit nástroj ze stejného kontejneru:

```
docker exec mariadb sh -c 'exec mysqldump -user USERNAME --password --lock-tables --databases DATABASENAME' > /vaše/cesta/na/váš/hostitel/rukovoditel.sql
```

### Obnova dat z výpisu databáze

K obnovení dat můžete použít příkaz docker exec s parametrem -i

```
docker exec -i mariadb sh -c 'exec mysql -user USERNAME --password' < /vaše/cesta/na/váš/hostitel/rukovoditel.sql
```

# Použití docker-compose:

Webovou aplikaci lze nainstalovat také pomocí docker-compose:

```
version: '3'
services:
        rukovoditel:
                image: registriren/rukovoditel:latest
                volumes:
                        - RukovoditelWEB:/var/www/html
                networks:
                        - RukovoditelNET
                ports:
                        - "8008:80"
        mariadb:
                image: mariadb
                volumes:
                        - RukovoditelDB:/var/lib/mysql
                networks:
                        - RukovoditelNET
                ports:
                        - "3306:3306"
                environment:
                        - MYSQL_ROOT_PASSWORD=secret
                        - MYSQL_USER=ruser
                        - MYSQL_PASSWORD=secret
                        - MYSQL_DATABASE=rukovoditel
volumes:
        RukovoditelWEB:
        RukovoditelDB:
networks:
        RukovoditelNET:
```

# License

[Rukovoditel](https://www.rukovoditel.net/download.php) is open source and released under the terms of the [GNU General Public License v2 (GPL).](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html) [Quick GPL-2.0 Summary.](https://tldrlegal.com/license/gnu-general-public-license-v2)

Main version of [Rukovoditel](https://www.rukovoditel.net/download.php) is free and fully functional software without any restrictions. [Extension](https://www.rukovoditel.net/extension.php) is commercial product and it is not included in this image.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
