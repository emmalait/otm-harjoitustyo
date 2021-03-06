# Arkkitehtuurikuvaus

## Rakenne
Ohjelma on jaettu kolmeen pakkaukseen:


<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/pakkaukset.png?raw=true" width="200">

Pakkaus **filmlogger.ui** pitää sisällään JavaFX:llä toteutetun käyttöliittymän ja siihen liittyvät FXML Controller -luokat. Pakkaus **filmlogger.domain** sisältää sovelluslogiikasta vastaavat luokat ja pakkaus **filmlogger.dao** puolestaan sisältää tiedon tallentamiseen ja hakemiseen liittyvät luokat.

## Käyttöliittymä
Käyttöliittymässä on neljä näkymää:
- sisäänkirjautuminen (*LoginScene*)
- rekisteröityminen (*RegisterScene*)
- logattujen elokuvien tarkastelu -näkymä (*LoggerScene*)
- arvioinnin lisäys -näkymä (*ReviewScene*)

Näkymät ovat omiia Scene-olioitaan, jotka ovat yksikerrallaan näkyvissä ohjelmassa eli asetettuna ohjelman stageen. Käyttöliittymä on rakennettu FXML-toteutuksena. Näkymiä vastaavat FXML-tiedostot on tallennettu ohjelman resources/fxml-kansioon (*/src/main/resources/fxml*). Näkymien toiminnallisuudesta vastaavat Controller-luokat ovat ohjelman filmlogger.ui -pakkauksessa.

Ohjelman sovelluslogiikka on pyritty erottamaan käyttöliittymästä ja käyttöliittymän toiminnot toteuttavat Controller-luokat kutsuvat ensisijaisesti sovelluslogiikasta vastaavan Logger-olion metodeja. Kirjautumisen yhteydessä LoggerSceneen päivitetään kirjautunutta käyttäjää vastaavat elokuvien tiedot, jotka ladataan tietokannasta uudelleen myös aina lisäysten ja muokkausten yhteydessä. 

## Sovelluslogiikka
Sovelluksen loogisen datamallin muodostavat luokat User, Film, Tag ja Review, jotka kuvaavat käyttäjien ja elokuvien välisiä suhteita:

<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/luokat1.png?raw=true" width ="600">

Ohjelman toiminnallisuudesta vastaa keskeisesti *Logger*-olion ohjelmakohtainen instanssi, joka tarjoaa käyttöliittymän toiminnoille metodit kuten esim.:
- boolean login(String username)
- User getCurrentUser()
- void logout()
- String createUser(String user, String username)
- List<Review> getWatchlist()
- String addToWatchlist(String filmName, String filmYear)

Loggerilla on pääsy käyttäjiin, elokuviin, arvioihin ja tunnisteisiin filmlogger.dao-pakkauksessa DAO-rajapinnat toteuttavien luokkien kautta. Sovelluslogiikka eli Logger-instanssi saa tiedon luokista konstruktorinsa kautta. Loggerin suhdetta tallennuksesta vastaaviin luokkiin kuvataan alla olevassa kuvassa.

<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/pakkauskaavio.png?raw=true" width="650">

## Tietojen pysyväistallennus

Tietojen tallennuksesta tietokantaan ja hakemisesta tietokannasta vastaavat filmlogger.dao-pakkauksen luokat DbUserDAO, DbFilmDAO, DbTagDAO ja DbReviewDAO. Luokat noudattavat Data Access Object -suunnittelumallia ja kaikki luokat toteuttavat niitä vastaavan rajapinnan. Sovelluslogiikka ei näin ollen ole suoraan yhteydessä tallennuksesta vastaaviin luokkiin ja tallennuksen toteutus voidaan tarvittaessa muuttaa jos tietoa halutaankin tallentaa toisessa muodossa. 

### Tietokanta

Ohjelma käyttää tietojen tallentamiseen SQL-tietokantaa. Tietokanta tauluineen luodaan ohjelman käynnistyksen yhteydessä jos niitä ei ennalta ole olemassa. Tietokanta nimetään automaattisesti nimellä *filmlogger.db* ja se sisältää taulut User, Film, Tag ja Review.

## Päätoiminnallisuudet

Seuraavassa kuvataan ohjelman keskeisimpiä toiminnallisuuksia.

### Uuden käyttäjän luominen
Kun rekisteröitymisnäkymässä käyttäjä on syöttänyt käyttäjätunnuksen ja nimen ja klikkaa painiketta registerInputButton, toimii ohjelma seuraavasti:

<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/DiagramRegister.png?raw=true">

Käyttöliittymän näkymästä vastaava Controller-luokka (RegisterSceneController) kutsuu Logger-instanssin createUser-metodia. Logger-instanssi tarkistaa UserDAO:n kautta tietokannasta onko käyttäjätunnusta olemassa ja palauttaa null kun käyttäjää ei löydy. Tämän jälkeen Logger luo UserDAO:n kautta tietokantaan uuden käyttäjän ja Logger palauttaa käyttöliittymään viestin, että käyttäjän luonti onnistui.

### Kirjautuminen
Kun kirjautumisnäkymässä käyttäjä syöttää käyttäjätunnuksensa ja klikkaa loginButton-painiketta, toimii ohjelma seuraavasti:

<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/DiagramLogin.png?raw=true">

Käyttöliittymän näkymästä vastaava Controller-luokka (LoginSceneController) kutsuu Logger-instanssin login-metodia. Logger tarkistaa UserDAO:n kautta löytyykö käyttäjä tietokannasta. UserDAO palauttaa Loggerille käyttäjän ja Logger palauttaa Controllerille true. Tämän jälkeen käyttöliittymään vaihdetaan seuraava näkymä.

### Elokuvan lisääminen watchlistille
Kun käyttäjä syöttää logger-näkymässä elokuvan nimen ja vuoden ja klikkaa addToWatchlistButton-painiketta, toimii ohjelma seuraavasti:

<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/DiagramAddToWatchlist.png?raw=true">

Käyttöliittymän näkymästä vastaava Controller-luokka (LoggerSceneController) kutsuu Logger-instanssin metodia findByName, joka tarkistaa FilmDAO:n avulla tietokannasta löytyykö elokuva sieltä jo. Kun elokuvaa ei löydy, palauttaa se Loggerille arvon null. Tämän jälkeen Logger kutsuu FilmDAO:n create-metodia, joka lisää elokuvan tietokantaan. Seuraavaksi Logger kutsuu TagDAO:n metodia getToWatch(), joka palauttaa "to watch"-tagin. Tämän jälkeen Logger kutsuu ReviewDAO:n create-metodia, joka lisää uuden Reviewn tietokantaan. Koska kyseessä on elokuva, jota käyttäjä ei ole vielä nähnyt, jätetään katselupäivämäärä, numeerinen arvio ja sanallinen arvio toistaiseksi tyhjiksi.

Lisäysoperaation jälkeen käyttöliittymän watchlist-näkymä päivitetään heijastamaan muuttunutta tilannetta.

### Elokuvan merkitseminen nähdyksi
Kun käyttäjä klikkaa logger-näkymän watchlist-välilehdessä olevan elokuvan vieressä olevaa markAsSeen-painiketta, toimii ohjelma seuraavasti:

<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/DiagramAddToSeen.png?raw=true">

Käyttöliittymän näkymästä vastaava Controller-luokka (LoggerSceneController) kutsuu Logger-instanssin metodia markAsSeen. Logger kutsuu ensin TagDAO:n findByName-metodia, joka palauttaa "seen"-tagin. Tämän jälkeen Logger kutsuu käsiteltävän Review'n setTag-metodia ja asettaa sille "seen"-tagin. Seuraavaksi Logger-instanssi kutsuu ReviewDAO:n update-metodia, jonka avulla tieto päivitetään tietokantaan. 

Muutosperaation jälkeen käyttöliittymän watchlist- ja seen-näkymät päivitetään heijastamaan muuttunutta tilannetta.

### Elokuvalle arvion lisääminen
Kun käyttäjä klikkaa logger-näkymän seen-välilehdessä elokuvan nimen vieressä olevaa reviewButton-painiketta, toimii ohjelma seuraavasti: 

<img src="https://github.com/emmalait/FilmLogger/blob/master/dokumentaatio/images/DiagramAddReview.png?raw=true">

Ensin käyttöliittymään vaihdetaan arvionäkymä. Arvionäkymässä käyttäjä syöttää katselupäivämäärän, numeerisen arvion (1-5) ja sanallisen arvion. Kun käyttäjä klikkaa addReviewButton-painiketta, käyttöliittymän Controller-luokka (ReviewSceneController) kutsuu käsiteltävän review-olion metodeja setDate, setRating ja setReview, jotka lisäävät oliolle käyttäjän syöttämät tiedot. Tämän jälkeen Controller-luokka kutsuu Loggerin updateReview-metodia ja Logger päivittää muuttuneet tiedot ReviewDAO:n avulla tietokantaan. Lopuksi käyttöliittymään palautetaan logger-näkymä.
