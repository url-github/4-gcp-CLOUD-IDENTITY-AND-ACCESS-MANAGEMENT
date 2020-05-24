# 4-gcp-CLOUD-IDENTITY-AND-ACCESS-MANAGEMENT

( notatki z Chmurowisko )

---

#### Dodanie użytkownika za pomocą CLI 

> gcloud projects add-iam-policy-binding gcp-cloud-276415 --member user:example@gmail.com --role roles/viewer

#### Usunięcie użytkownika za pomocą CLI 

> gcloud projects remove-iam-policy-binding gcp-cloud-276415 --member user:example@gmail.com --role roles/viewer

#### Eksport polityk do JSON

> gcloud projects get-iam-policy gcp-cloud-276415 --format json > ~/policy.json

> ls

> nano policy.json

> ctrl + x ( wyjście z nano )

#### Tworzenie kont serwisowych za pomocą CLI ( kont usługi )

> gcloud iam service-accounts create pm-v1-cli --description "opis" --display-name "pm-v1-cli"

> gcloud iam service-accounts list

#### Stworzenie klucza oraz przypisanie go do wcześniej utworzonego konta serwisowego "pm-v1-cli"

> gcloud iam service-accounts keys create ~/key.json --iam-account pm-v1-gcp@gcp-cloud-276415.iam.gserviceaccount.com

#### Wylistowanie kont serwisowych

> gcloud iam service-accounts list

#### Tworzenie ról za pomocą CLI

1. Wprowadzenie parametrów do pliku YML

> nano roles.yml

title: "Role Viewer"
description: "My custom role description."
stage: "ALPHA"
includedPermissions:
- iam.roles.get
- iam.roles.list

2. Załadowanie roli

> gcloud iam roles create pmviewer --project gcp-cloud-276415 --file roles.yml

3. Sprawdzenie roli

> gcloud iam roles describe pmviewer --project gcp-cloud-276415

#### Organization Policy Service and Constraints ( Zasady organizacji )

To działa kiedy mam założoną organizację. W GCP pod produkcję najlepiej wyłączyć tworzenie domyślnej sieci VPC.

## Dodatkowo mogę ograniczyć ilość obrazów maszyn.

1. Polityka blokowania obrazów podczas tworzenia VM.

> gcloud beta resource-manager org-policies describe compute.trustedImageProjects --effective  --project gcp-cloud-276415

2. Zrzucenie polityki do pliku oraz zrobienie kopii.

> gcloud beta resource-manager org-policies describe compute.trustedImageProjects --effective  --project gcp-cloud-276415 > client2policy.yaml

> gcloud beta resource-manager org-policies describe compute.trustedImageProjects --effective  --project gcp-cloud-276415 > client2restore.yaml

3. Wylistuje listę obrazów.

> gcloud compute images list

4. W pliku "client2policy.yaml" edytuję zawartość.

constraint: constraints/compute.trustedImageProjects
listPolicy:
  allValues: ALLOW
  
---

constraint: constraints/compute.trustedImageProjects
listPolicy:
deniedValues:
- projects/debian-cloud
  
5. Ładuje nową politykę.

> gcloud beta resource-manager org-policies set-policy --project gcp-cloud-276415 client2policy.yaml

6. Przywracam starą politykę.

> gcloud beta resource-manager org-policies set-policy --project gcp-cloud-276415 client2restore.yaml

#### Szyfrowanie danych (za pomocą KMS url: https://cloud.google.com/kms/ ) zapisanych w Google Cloud Bucket.

1. Utworzenie Bucket. 

> gsutil mb gs://gcp-kms-bucket

2. Repozytorium z plikami do zaszyfrowania ( https://github.com/url-github/example-data )

> git clone https://github.com/url-github/example-data

3. Komendy w bash

> cd example-data

> ls

> cat txt01.txt

4. Uruchamiam API KMS.

> gcloud services enable cloudkms.googleapis.com

5. Aby zaszyfrować dane tworzę: Key ring ( to taki odpowiednik folderów ) oraz Key. 

> gcloud kms keyrings create mykeyring --location global

> gcloud kms keys create mykey --location global --keyring mykeyring --purpose encryption 

6. Done. ID zasobu to: 

> projects/gcp-cloud-276415/locations/global/keyRings/mykeyring/cryptoKeys/mykey

7. Szyfrowanie pliku. Wykonuję formatowanie ( base64 ) i zapis pliku do zmiennej TEXT

> TEXT=$(cat txt01.txt | base64 -w0)

8. Podejrzenie pliku sformatowanego do base64

> echo $TEXT

9. Szyfrowanie.

curl -v "https://cloudkms.googleapis.com/v1/projects/gcp-cloud-276415/locations/global/keyRings/mykeyring/cryptoKeys/mykey:encrypt" \
-d "{\"plaintext\":\"$TEXT\"}" \
-H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \
-H "Content-Type: application/json"

W JSON mam zaszyfrowany tekst:

  "ciphertext": "CiQARqcOATlKPR3fnygDNDyah1yO7TqynSCN5xu2XPzztVRI94ESuwMA8VMfnYc2370Vrd5zLFRVFYfrCetE4rn1zkIUMy9Fi6mE6A5RPAZdNYPHsK/OlTx9BIR/UJORq5+0zzJiVi3HSoYDJuNcHR0TDFQDnlTwC00fIQFvIA8VlYKjb
q1mbYhd3yML7g9aBO2pnjhXmd8wIBR+J20kG2uBkwRAE3W07bP6B/zKife1/AcANgywUEyw9Vevlkxbp0ieQqZROqIInfKUkEpMECxM4+VsNhrDv2qMDNraNrA9hwvBvlZmufQHuh85nB89SciPTWFvyqo82RyT+U41HWL20qj7GviBw6OdqwkKF7nm3N528cG
ZbkThtvLpgDv2OTYjfgr8bC/OUGK1h7iINAa9e1e+ok0TSpAFdp2Dnf6SadnX4VvbR5fCXap/52CgszIqzQ0NmgO+zS5iI3QWX/qKwZMVHb82JtSMLjDuSN1B/7dWLL3gEQQ8euraByyeKyzAah/kWVM7gQzRRj/DFpqGSmS/pH3BmsKiZqOCVhZkiSF6Rs8XG
6zG6p53icoFeV79gyYrbYOPG6zwMc9Cp1QgaedxX169O6ywy1DkdjYNctdvu6LLPlZPEk8RIT+bF6kXMg==",

10. Szyfrowanie wraz z zapisem do pliku ( txt01.encrypted ):

curl -v "https://cloudkms.googleapis.com/v1/projects/gcp-cloud-276415/locations/global/keyRings/mykeyring/cryptoKeys/mykey:encrypt" \
-d "{\"plaintext\":\"$TEXT\"}" \
-H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \
-H "Content-Type: application/json" \
jq ciphertext -r > file01.encrypted

# Zadanie 4.

## 1. Zadanie 1
Klient poprosił cię o przygotowanie maszyny dla swoich pracowników, którzy będą mogli pobierać faktury z przygotowanego repozytorium (w naszym przypadku jest to pojemnik Cloud Storage)

### 1.1 Przygotowanie Cloud Storage
#### Zmienne
bucketName="zad4pmstorage"
bucketLocation="europe-west3"

#### Utworzenie bucketa
gsutil mb -c STANDARD -l europe-west3 gs://zad4pmstorage/

#### Utworzenie plików
echo "Plik 1 - przykładowy tekst 1" > test10.txt
echo "Plik 2 - przykładowy tekst 2" > test11.txt

#### Wysłanie plików
gsutil cp test1*.txt gs://zad4pmstorage/

### 1.2 Utworzenie Service Account

Utworzenie konta serwisowego z dostępnem Read-only do wcześniej utworzonego bucketa

Administracja > Konta usługi

Krok 1. Nazwa konta usługi: Bucket Viewer zad4
Krok 2. Rola Wyświetlający obiekty Cloud Storage
Krok 3. Oraz warunku dostępu tylko do danego bucketa:

resource.name == "projects/_/buckets/zad4pmstorage" ||
resource.name.startsWith("projects/_/buckets/zad4pmstorage/objects/")

### 1.3 Utworzenie VM

# Zmienne
vmName="zad4pmvm"
vmType="f1-micro"
vmZone="europe-west3-b"
serviceAccountEmail="bucket-viewer-zad4@gcp-cloud-276415.iam.gserviceaccount.com" 

> gcloud iam service-accounts list 

# Utworzenie VM

> gcloud compute instances create $vmName --zone=$vmZone --machine-type=$vmType --image-project=debian-cloud --image=debian-9-stretch-v20191210 --service-account=$serviceAccountEmail

> gcloud compute instances create zad4pmvm --zone=europe-west3-b --machine-type=f1-micro --image-project=debian-cloud --image=debian-9-stretch-v20191210 --service-account=bucket-viewer-zad4@gcp-cloud-276415.iam.gserviceaccount.com

### 1.4 Sprawdzenie uprawnień
Połączenie się z VM i sprawdzenie czy ma dostęp Read-only do bucketa.

> bigdata_pw_2020@zad4pmvm-v2:~$ gsutil ls gs://zad4pmstorage

gs://zad4pmstorage/test1.txt
gs://zad4pmstorage/test10.txt
gs://zad4pmstorage/test11.txt
gs://zad4pmstorage/testvm.txt

> bigdata_pw_2020@zad4pmvm-v2:~$ gsutil cat gs://zad4pmstorage/test1.txt

Plik 1 - przykładowy tekst 1

> bigdata_pw_2020@zad4pmvm-v2:~$ echo "test1" > testvm.txt

> bigdata_pw_2020@zad4pmvm-v2:~$ ls

testvm.txt

> bigdata_pw_2020@zad4pmvm-v2:~$ gsutil cp testvm.txt gs://zad4pmstorage

Copying file://testvm.txt [Content-Type=text/plain]...
AccessDeniedException: 403 Insufficient Permission                              

> bigdata_pw_2020@zad4pmvm-v2:~$ gsutil rm gs://zad4pmstorage/test1.txt

Removing gs://zad4pmstorage/test1.txt...
AccessDeniedException: 403 Insufficient Permission

> bigdata_pw_2020@zad4pmvm-v2:~$ gsutil ls gs://
AccessDeniedException: 403 bucket-viewer-zad4-v2@gcp-cloud-276415.iam.gserviceaccount.com does not have storage.buckets.list access to the Google Cloud project.

> bigdata_pw_2020@zad4pmvm-v2:~$ 

### 1.5 Usunięcie zasobów

> gcloud compute instances delete $vmName --zone=$vmZone 
> gcloud compute instances delete zad4pmvm --zone=europe-west3-b	

> gcloud iam service-accounts delete $serviceAccountEmail
> gcloud iam service-accounts list
> gcloud iam service-accounts delete bucket-viewer-zad4@gcp-cloud-276415.iam.gserviceaccount.com

> gsutil -m rm -r gs://${bucketName}/
> gsutil -m rm -r gs://zad4pmstorage/

> rm test*.txt

## 2. Zadanie 2

Dany klient przetrzymuje bardzo ważne dokumenty. Zarząd zdecydował, że wprowadzą szyfrowanie krytycznych dokumentów, które będą mogły zostać odszyfrowane po stronie pracownika, który z danego dokumentu chce skorzystać.

### 2.1 Utworzenie bucketa dla plików
bucketName="secretstoragepm"
bucketLocation="europe-west3"

# Utworzenie bucketa

> gsutil mb -c STANDARD -l $bucketLocation gs://${bucketName}/

> gsutil mb -c STANDARD -l europe-west3 gs://secretstoragepm/

### 2.2 Uruchomieie usługi KMS

> gcloud services enable cloudkms.googleapis.com

### 2.3 Utworzenie klucza asymetrycznego
keyringsName="vmkeyrings"
keyName="vmKeyAsync"
keyPurpose="asymmetric-encryption"
defaultAlgorithm="rsa-decrypt-oaep-3072-sha256"

# Utworzenie Keyrings

> gcloud kms keyrings create $keyringsName --location global

> gcloud kms keyrings create vmkeyrings --location global

#### Utworzenie klucza ( https://cloud.google.com/kms/docs/creating-asymmetric-keys )

> gcloud kms keys create $keyName --location global --keyring $keyringsName --purpose $keyPurpose --default-algorithm $defaultAlgorithm

> gcloud kms keys create vmKeyAsync --location global --keyring vmkeyrings --purpose asymmetric-encryption --default-algorithm rsa-decrypt-oaep-3072-sha256 

### 2.4 PoC w Cloud Shell

### PoC w Cloud Shell

#### 2.4.1 Utworzenie przykładowego pliku

> echo "Plik 1 - przykładowy tekst 1 qwertyążźśćłń" > test1.txt

#### 2.4.2 Pobranie klucza publicznego
keyVersion="1"

#### Pobranie klucza publicznego ( https://cloud.google.com/kms/docs/retrieve-public-key#kms-howto-retrieve-public-key-cli )

> gcloud kms keys versions get-public-key $keyVersion --location global --keyring $keyringsName --key $keyName --output-file public-key.pub

> gcloud kms keys versions get-public-key 1 --location global --keyring vmkeyrings --key vmKeyAsync --output-file public-key.pub

#### 2.4.3 Zaszyfrowanie pliku ( https://cloud.google.com/kms/docs/encrypt-decrypt-rsa#encrypt_data )

> openssl pkeyutl -in $HOME/zadanie4/test1.txt -encrypt -pubin -inkey $HOME/zadanie4/public-key.pub -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256 > $HOME/zadanie4/secret/test1.enc

> openssl pkeyutl -in test1.txt -encrypt -pubin -inkey public-key.pub -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256 > test1.enc

#### 2.4.4 Odszyfrowanie pliku ( https://cloud.google.com/kms/docs/encrypt-decrypt-rsa#decrypt_data )

> gcloud kms asymmetric-decrypt --location global --keyring $keyringsName --key $keyName --version $keyVersion --ciphertext-file $HOME/zadanie4/secret/test1.enc --plaintext-file $HOME/zadanie4/test1-odszyfrowany.txt

> gcloud kms asymmetric-decrypt --location global --keyring vmkeyrings --key vmKeyAsync --version 1 --ciphertext-file test1.enc --plaintext-file test1-odszyfrowany.txt

#### 2.4.5 Porównanie pliku po odszyfrowaniu

> cat test1.txt

Plik 1 - przykładowy tekst 1 qwertyążźśćłń

> cat test1-odszyfrowany.txt

Plik 1 - przykładowy tekst 1 qwertyążźśćłń

### 2.5 Utworzenie kont serwisowych

#### 2.5.1 Konto serwisowe do szyfrowania dokumentów

1. Administracja.
2. Konta usługi.
3. Utwórz konto usługi. Nazwa konta usługi: document-encryptor

Dodanie ról (uprawnienia konta usługi):

- Storage Object Creator oraz
- Cloud KMS CryptoKey Public Key Viewer

Oraz warunków:

zapisu tylko do danego bucketa:
Name Starts with projects/_/buckets/secretstoragebp/objects/

oraz pobieranie kluczy publicznych z danego keyringa:
Name Starts with projects/resonant-idea-261413/locations/global/keyRings/vmkeyrings/cryptoKeys/

#### 2.5.2 Konto serwisowe do odszyfrowania dokumentów

Dodanie ról (uprawnienia konta usługi):

- Storage Object Viewer oraz
- Cloud KMS CryptoKey Decrypter

Oraz warunków:

odczytu danych tylko z danego bucketa:
Name is projects/_/buckets/secretstoragepm
or Name Starts with projects/_/buckets/secretstoragepm/objects/

oraz deszyfrowania danych za pomocą kluczy z danego keyringa:
Name Starts with projects/gcp-cloud-276415/locations/global/keyRings/vmkeyrings/cryptoKeys/

### 2.6 Utworzenie VM
vmNameEncrypt="zad4encr"
vmNameDecrypt="zad4decr"
vmType="f1-micro"
vmZone="europe-west3-b"

> gcloud iam service-accounts list 

serviceAccountEmailEncrypt="document-decryptor@gcp-cloud-276415.iam.gserviceaccount.com"
serviceAccountEmailDecrypt="document-encryptor@gcp-cloud-276415.iam.gserviceaccount.com"

# Encryptor VM

> gcloud compute instances create $vmNameEncrypt --zone=$vmZone --machine-type=$vmType --image-project=debian-cloud --image=debian-9-stretch-v20191210 --service-account=$serviceAccountEmailEncrypt --scopes=https://www.googleapis.com/auth/cloud-platform

> gcloud compute instances create zad4encr --zone=europe-west3-b --machine-type=f1-micro --image-project=debian-cloud --image=debian-9-stretch-v20200521 --service-account=document-encryptor@gcp-cloud-276415.iam.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform

# Decryptor VM

> gcloud compute instances create $vmNameDecrypt --zone=$vmZone --machine-type=$vmType --image-project=debian-cloud --image=debian-9-stretch-v20191210 --service-account=$serviceAccountEmailDecrypt --scopes=https://www.googleapis.com/auth/cloud-platform

> gcloud compute instances create zad4decr --zone=europe-west3-b --machine-type=f1-micro --image-project=debian-cloud --image=debian-9-stretch-v20191210 --service-account=document-decryptor@gcp-cloud-276415.iam.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform

### 2.7 Zaszyfrowanie plików

bucketName="secretstoragepm"
keyringsName="vmkeyrings"
keyName="vmKeyAsync"
keyVersion="1"

# Utworzenie przykładowych plików

> echo "Plik 1 - przykładowy tekst 1 ąźćżółęż" > test1.txt

> echo "Plik 2 - przykładowy tekst 2 ąźćżółęż" > test2.txt

# Pobranie klucza publicznego

> gcloud kms keys versions get-public-key $keyVersion --location global --keyring $keyringsName --key $keyName --output-file public-key.pub

> gcloud kms keys versions get-public-key 1 --location global --keyring vmkeyrings --key vmKeyAsync --output-file public-key.pub

# Zaszyfrowanie plików

> mkdir secret

Plik 1.

> openssl pkeyutl -in $HOME/test1.txt -encrypt -pubin -inkey $HOME/public-key.pub -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256 > $HOME/secret/test1.enc

> openssl pkeyutl -in test1.txt -encrypt -pubin -inkey public-key.pub -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256 > secret/test1.enc

Plik 2.

> openssl pkeyutl -in $HOME/test2.txt -encrypt -pubin -inkey $HOME/public-key.pub -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256 > $HOME/secret/test2.enc

> openssl pkeyutl -in test2.txt -encrypt -pubin -inkey public-key.pub -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256 > secret/test2.enc

# Wysłanie plików do Cloud Storage

> gsutil cp $HOME/secret/test1.enc gs://$bucketName/

> gsutil cp secret/test1.enc gs://secretstoragepm/

> gsutil cp $HOME/secret/test2.enc gs://$bucketName/

> gsutil cp secret/test2.enc gs://secretstoragepm/

# Próby wykonania niedozwolonych operacji

> gsutil ls gs://$bucketName

> gsutil ls gs://secretstoragepm/

> gsutil rm gs://$bucketName/test1.enc

> gsutil rm gs://secretstoragepm/test1.enc

> gsutil cat gs://$bucketName/test1.enc # powodzenie - jak można zauważyć może odczytywać pliki które utworzył

> gsutil cat gs://secretstoragepm/test1.enc # powodzenie - jak można zauważyć może odczytywać pliki które utworzył

> gcloud kms asymmetric-decrypt --location global --keyring $keyringsName --key $keyName --version $keyVersion --ciphertext-file $HOME/secret/test1.enc --plaintext-file $HOME/test1-odszyfrowany.txt

> gcloud kms asymmetric-decrypt --location global --keyring vmkeyrings --key vmKeyAsync --version 1 --ciphertext-file secret/test1.enc --plaintext-file test1-odszyfrowany.txt

### 2.8 Odszyfrowanie plików

bucketName="secretstoragebp"
keyringsName="vmkeyrings"
keyName="vmKeyAsync"
keyVersion="1"

# Pobranie plików
gsutil cp gs://$bucketName/* .

# Odszyfrowanie plików
gcloud kms asymmetric-decrypt --location global --keyring $keyringsName --key $keyName --version $keyVersion --ciphertext-file $HOME/test1.enc --plaintext-file $HOME/test1-odszyfrowany.txt
gcloud kms asymmetric-decrypt --location global --keyring $keyringsName --key $keyName --version $keyVersion --ciphertext-file $HOME/test2.enc --plaintext-file $HOME/test2-odszyfrowany.txt

# Wyświetlenie zawartości odszyfrowanych plików
cat test1-odszyfrowany.txt
cat test2-odszyfrowany.txt

# Próby wykonania niedozwolonych operacji
gsutil rm gs://$bucketName/test1.enc
gcloud kms keys versions get-public-key $keyVersion --location global --keyring $keyringsName --key $keyName --output-file public-key.pub
gsutil cp $HOME/test1-odszyfrowany.txt gs://$bucketName/






























