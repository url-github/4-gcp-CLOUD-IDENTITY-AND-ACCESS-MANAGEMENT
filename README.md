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































