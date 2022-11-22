# h4 Omat komennot

## a) Hei komento! Tee järjestelmään uusi "hei maailma" -komento ja asenna se orjille Saltilla. Liitä raporttiisi orjan 'ls -l /usr/local/bin/' tulosteesta ainakin se rivi, jolla näkyy uuden komentotiedostosi oikeudet.

Aloitin tekemällä kotihakemistooni kansion `scripts`, ja sinne tiedoston `heimaailma.sh`:

		~$ mkdir scripts
		~$ cd scripts/
		$ micro heimaailma.sh

Tämän jälkeen tallensin skriptitiedoston, tarkistin sen sisällön katenoimalla, tarkistin sen käyttäjäoikeudet ja lisäsin jokaiselle käyttäjälle suoritusoikeudet (+x). Tämän jälkeen vielä ajoin skriptin kerran:

![screenshot of testing of simple helloworld script](1.png)

Skripti vaikutti toimivalta, joten siirryin tekemään sille salt-tilaa. Tämän tein luomalla `/srv/salt/`-hakemistoon uuden hakemiston `simple_scripts`, ja luomalla sinne `init.sls`-staten. Statea varten tarvitsin myös tiedoston oikeuksien muokkauksen, tämän löysin saltin state_doc:ista etsimällä file.managed-kohdan esimerkkejä. Tiesin linuxin perusteista, että käyttäjäoikeuksia voidaan määritellä numeroin, joten yhden esimerkin mode-kohdan numerosarja '0644' vaikutti relevantilta. Sitten selvitin tarvittavat oikeudet aiemmin tekemästäni `heimaailma.sh`-tiedostosta:

		$ stat ~/scripts/heimaailma.sh

Tulosteen kohdasta access näki tiedoston oikeudet -` 0755/-rwxr-xr-x`

![execute priviliges in stat](2.png)

		~$ sudo mkdir /srv/salt/simple_scripts
		~$ cd /srv/salt/simple_scripts
		$ sudo micro init.sls

		/usr/local/bin/simple_scripts:
		  file.managed:
		    - source: salt://simple_scripts/heimaailma.sh
		    - mode: '0755'

Tallensin tiedoston ja kopioin `heimaailma.sh`-skriptin saltin kansioon (source://salt-viittaa /srv/saltin-hakemistoon, eli tiedoston tulee olla siellä):

		$ sudo cp ~/scripts/heimaailma.sh /srv/salt/simple_scripts/
		$ ls /srv/salt/simple_scripts/
		heimaailma.sh  init.sls

Sitten kokeilin ajaa staten:

		$ sudo salt '*' state.apply simple_scripts

![salt state applied to minions](3.png)

Minionilla scriptiä testatessani huomasin tehneeni pienen virheen staten luonnissa. `heimaailma.sh` sijaan olin luonut tiedoston `simple_scripts`, jonka sisältö oli sama kuin `heimaailma.sh`:n

![incorrect state file](4.png)

Korjasin tilanteen muokkaamalla `/srv/salt/simple_scripts/init.sls`-tiedostoa siten, että ensimmäiselle riville vaihdoin `simple_scripts` tilalle `heimaailma.sh`. Sitten ajoin tilan uudestaan, ja skripti toimi orjilla halutulla tavalla:

![working script file in a minion](5.png)


## b) whatsup.sh. Tee järjestelmään uusi komento, joka kertoo ajankohtaisia tietoja; asenna se orjille. Vinkkejä: Voit näyttää valintasi mukaan esimerkiksi päivämäärää, säätä, tietoja koneesta, verkon tilanteesta...

Tein komennon `~/scripts/`-hakemistoon `micro whatsup.sh`-komennolla.

		~$ cd /scripts
		$ micro whatsup.sh

		#!/usr/bin/bash
		
		echo -n "Hei "
		hostname -s
		echo "Tämänhetkinen päivämäärä ja aika on: "
		date

Tallensin tiedoston, muutin oikeudet `chmod ugo+x whatsup.sh` ja ajoin skriptin testiksi paikallisesti:

![whatsup.sh](6.png)

## c) hello.py. Tee järjestelmään uusi komento Pythonilla ja asenna se orjille. Vinkkejä: Hei maailma riittää, mutta propellihatut saavat toki koodaillakin. Shebang on "#!/usr/bin/python3". Helpoin Python-komento on: print("Hei Tero!")

Kuten aiemmin, loin skriptitiedoston `~/scripts`-hakemistoon. Vaihdoin oikeudet ja ajoin skriptin paikallisesti:

![hello.py](7.png)

## d) Laiskaa skriptailua. Tee kansio, josta jokainen skripti kopioituu orjille.

Kopioin aiemmin luodut skriptit `/srv/salt/simple_scripts/scripts`-hakemistoon, jonka loin skriptien säilöntää varten:

		$ sudo mkdir /srv/salt/simple_scripts/scripts
		$ sudo cp -r ~/scripts/ /srv/salt/simple_scripts/
		$ ls -l /srv/salt/simple_scripts/scripts/
		total 12
		-rwxr-xr-x 1 root root  37 marras 22 18:18 heimaailma.sh
		-rwxr-xr-x 1 root root  49 marras 22 18:18 hello.py
		-rwxr-xr-x 1 root root 101 marras 22 18:18 whatsup.sh

Tiedostot olivat nyt hakemistossa. Sitten muutin `/srv/salt/init.sls`-tiedostoa siten, että se vie koko kansion sisällön orjille:
		             
		/usr/local/bin/:
		  file.recurse:
		    - source: salt://simple_scripts/scripts/    
		    - file_mode: '0755'

file_mode on tärkeä muutos - file.recursen kanssa pelkkä mode ei riitä. Sitten ajoin salt-tilan uudestaan kuten a-kohdassa:

		$ sudo salt '*' state.apply simple_scripts
		good_debian:
		----------
		          ID: /usr/local/bin/
		    Function: file.recurse
		      Result: True
		     Comment: Recursively updated /usr/local/bin/
		     Started: 18:25:34.881313
		    Duration: 128.053 ms
		     Changes:   
		              ----------
		              /usr/local/bin/hello.py:
		                  ----------
		                  diff:
		                      New file
		                  mode:
		                      0755
		              /usr/local/bin/whatsup.sh:
		                  ----------
		                  diff:
		                      New file
		                  mode:
		                      0755
		
Salt lupasi tehneensä muutokset, tarkastin etukäteen yhdellä orjalla skriptin toimimattomuuden ja uusien skriptien toimivuuden salt-tilan ajon jälkeen:

![multiple scripts to minions with salt](8.png)

Kaikki kolme skriptiä toimi! (Toki heimaailma.sh oli säädetty toimimaan jo aiemmin, mutta se toimi jatkossakin)
