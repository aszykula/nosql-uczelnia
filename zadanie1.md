Sprz�t, na kt�rym pracuj�: Pentium B970 @ 2.3 GhZ, 6 GB RAM, Linux Mint Xfce 17

### Zadanie 1a

Najpierw przetworzy�em baz� za pomoc� skryptu z repozytorium prowadz�cego:

```sh
if [ -z "$1" ] ; then
echo "First replaces all the \\n with spaces then replaces all the \\r with \\n"
echo "Replace header line with: _id,title,body,tags (for mongoDB)"
echo "usage: $0 input.csv output.csv"
exit 1;
fi

tr '\n' ' ' < "$1" | tr '\r' '\n' > "$2"
sed -i '$ d' "$2"

sed -i '1 c "_id","title","body","tags"' "$2" 

```

P�niej zaimportowa�em plik do bazy za pomoc� polecenia:

```sh
time mongoimport -c Topics --type csv --file Train2.csv --headerline
```

W wypadku Postgresa wygl�da to nieco inaczej - najpierw trzeba stworzy� odpowiedni� tabel�:

```sh
CREATE TABLE Topics(
ID INT PRIMARY KEY NOT NULL,
TITLE CHAR(128) NOT NULL,
BODY CHAR(128) NOT NULL,
TAGS CHAR(128) NOT NULL,
);
```

Dopiero potem mo�emy wype�ni� tabel� danymi z pliku:

```sh
COPY Topics FROM '/home/pc/nosql/Train2.csv' DELIMITER ',' CSV;
```

Czasy dla importu wynios�y: MongoDB 7m53s, PostgreSQL 9m01s.

### Zadanie 1b

Do obliczenia ilo�ci zaimportowanych rekord�w u�y�em polecenia:

```sh
db.Topics.count()
```

kt�re da�o wynik 6034195.

Zadanie 1c

```sh
baza = db.Train.find();

var tagsUnique = {};
var tagsNumber = 0;

baza.forEach(function(train){
    var tagsArray = [];
    if(typeof train.tags === "string") {
        tagsArray = train.tags.split(" ");
        db.Train.update({_id: train._id}, {$set: {tags: tagsArray}});
    } else if(typeof train.tags === "number") {
        tagsArray.push(train.tags.toString());
        db.Train.update({_id: train._id}, {$set: {tags: tagsArray}});
    } else {
        tagsArray = train.tags;
    }
    tagsNumber += tagsArray.length;
    tagsArray.forEach(function(tag) {
        if(typeof tagsUnique[tag] === "undefined")
            tagsUnique[tag] = 1;
    });
});

print("Wszystkie: " + tagsNumber);
print("Unikalne: " + Object.keys(tagsUnique).length);

```

### Zadanie 1d

Do rozwi�zania zadania u�y�em danych ze strony poipoint.pl. Dane dotycz� lokalizacji szk� podstawowych w Polsce. 

Na pocz�tek zaimportowa�em dane do mongo

```sh
mongoimport --db zadanie -d geo -c schools < /home/pc/nosql/szkoly.json
```

Do bazy wykona�em nast�puj�ce zapytania:


```sh
db.schools.ensureIndex({"loc" : "2dsphere"})
```
 
by ca�o�� mia�a prawo dzia�a�.

Szko�y podstawowe w odleg�o�ci 50 km od Gda�ska


```sh
 db.schools.find( { loc : { $near :
                         { $geometry :
                             { type : "Point" ,
                               coordinates: [ 18.639, 54.360 ] } },
                           $maxDistance : 50000
              } }, { _id: 0 } )
```

[JSON](master/data/zapytanie nr 1.json), [GeoJSON](master/data/zapytanie nr 1.geojson)

Szko�y podstawowe w odleg�o�ci 50 km od Warszawy

```sh
 db.schools.find( { loc : { $near :
                         { $geometry :
                             { type : "Point" ,
                               coordinates: [ 21.020, 52.259 ] } },
                           $maxDistance : 50000
              } }, { _id: 0 } )
```

[JSON](master/data/zapytanie nr 2.json), [GeoJSON](master/data/zapytanie nr 2.geojson)

sdfd