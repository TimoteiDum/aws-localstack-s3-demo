# Demo Amazon S3 cu LocalStack

Proiect demonstrativ pentru crearea unui bucket S3 local, încărcarea, citirea, listarea și ștergerea unui obiect. LocalStack emulează endpoint-ul AWS pe calculator; comenzile nu creează resurse AWS reale.

## Cerințe

- Docker Desktop sau Docker Engine cu Docker Compose v2
- Conexiune la internet la prima pornire, pentru descărcarea imaginilor
- Cont LocalStack gratuit și Auth Token pentru activarea LocalStack

Nu este necesar un cont AWS sau AWS CLI instalat local. Comenzile AWS CLI rulează într-un container temporar.

LocalStack solicită acum un Auth Token. Creează un cont gratuit la [LocalStack](https://app.localstack.cloud), apoi copiază un Developer Auth Token din pagina de tokenuri. În Terminal, setează tokenul numai pentru sesiunea curentă (înlocuiește valoarea exemplificativă local, nu o publica):

```sh
export LOCALSTACK_AUTH_TOKEN='TOKENUL_TAU_LOCALSTACK'
```

Nu adăuga tokenul în repository și nu îl trimite altor persoane.

## Pornire

Deschide terminalul în directorul proiectului și pornește LocalStack:

```sh
docker compose up -d localstack
```

Așteaptă inițializarea și verifică starea:

```sh
docker compose ps
curl http://localhost:4566/_localstack/health
```

## Operații S3

În comenzile de mai jos, AWS CLI din container folosește endpoint-ul intern `http://localstack:4566`. Datele de autentificare `test` sunt fictive și sunt folosite numai pentru emulator.

Creează bucket-ul:

```sh
docker compose run --rm aws --endpoint-url=http://localstack:4566 s3api create-bucket --bucket referat-demo
```

Încarcă fișierul de exemplu:

```sh
docker compose run --rm -v "$PWD:/work" -w /work aws --endpoint-url=http://localstack:4566 s3 cp mesaj.txt s3://referat-demo/documente/mesaj.txt
```

Listează bucket-urile și obiectele:

```sh
docker compose run --rm aws --endpoint-url=http://localstack:4566 s3api list-buckets
docker compose run --rm aws --endpoint-url=http://localstack:4566 s3 ls s3://referat-demo/documente/
```

Citește obiectul și salvează copia locală `mesaj.download`:

```sh
docker compose run --rm -v "$PWD:/work" -w /work aws --endpoint-url=http://localstack:4566 s3 cp s3://referat-demo/documente/mesaj.txt mesaj.download
cat mesaj.download
```

Șterge obiectul, apoi bucket-ul:

```sh
docker compose run --rm aws --endpoint-url=http://localstack:4566 s3 rm s3://referat-demo/documente/mesaj.txt
docker compose run --rm aws --endpoint-url=http://localstack:4566 s3api delete-bucket --bucket referat-demo
```

## Oprire și curățare

Oprește și șterge containerul și volumul proiectului:

```sh
docker compose down -v
```

## Rezultatul așteptat

- `create-bucket` răspunde fără eroare.
- Comanda `s3 ls` afișează `mesaj.txt`.
- `cat mesaj.download` afișează textul demonstrativ.
- După comenzile de ștergere, lista bucket-urilor nu mai conține `referat-demo`.

## Observații

LocalStack este un emulator pentru dezvoltare și testare locală. Comportamentul unor servicii și funcții poate diferi de AWS, deci un prototip local nu înlocuiește testarea și configurarea securizată în mediul AWS real. Pentru AWS real nu se folosesc cheile demonstrative din acest proiect.
