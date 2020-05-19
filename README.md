# 4-gcp-CLOUD-IDENTITY-AND-ACCESS-MANAGEMENT

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







