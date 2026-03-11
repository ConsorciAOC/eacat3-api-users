# eacat3-api-users
Guia d'integració per a desenvolupadors de les APIs d'autenticació, entitats i rols d'usuaris d'EACAT 3.0

## ⚠️ Avís important

Els entorns **DEV** i **PRE** són entorns de prova.

❗ **NO UTILITZAR DADES REALS EN ENTORNS DEV NI PRE**

Només es poden utilitzar:

- dades fictícies
- dades anonimitzades
- dades de prova

Aquest requisit s'estableix per garantir el compliment de:

- Esquema Nacional de Seguretat (ENS)
- normativa de protecció de dades

  ---

# Índex

- [1. Introducció](#1-introducció)
- [1.1 Propòsit](#11-propòsit)
- [1.2 Abast](#12-abast)
- [1.3 Document dirigit a](#13-document-dirigit-a)
- [2. Definicions](#2-definicions-abreviacions-i-acrònims)
- [3. Conceptes bàsics](#3-conceptes-bàsics)
- [3.1 Protocol](#31-protocol)
- [3.2 Política de seguretat de contrasenyes](#32-política-de-seguretat-de-contrasenyes)
- [3.3 Autenticació / Autorització](#33-autenticació--autorització)
- [3.4 Validació identificadors](#34-normalització-i-validació-didentificadors-nif--nie)
- [4. APIs exposades](#4-apis-exposades)
- [4.1 Autenticació d'usuaris](#41-autenticació-dusuaris)
- [4.2 Entitats d'usuaris](#42-entitats-dusuaris)
- [4.3 Rols d’un usuari](#43-rols-dun-usuari-en-una-entitat-i-una-aplicació)
- [4.4 Creació / modificació de credencials](#44-creació--modificació-de-les-credencials-del-usuari)

---

# 1. Introducció

## 1.1 Propòsit

Aquest document serveix per a donar a conèixer els diferents serveis exposats a **EACAT 3.0**.

Permet als integradors interactuar amb el sistema mitjançant API.

---

## 1.2 Abast

El document es centra en la definició de totes les possibles interaccions que es puguin portar a terme mitjançant **API REST**.

---

## 1.3 Document dirigit a

Aquest document està dirigit a persones amb **perfil tècnic** que necessiten implementar una integració amb **EACAT 3.0**.

---

# 2. Definicions, abreviacions i acrònims

| Concepte | Descripció |
|--------|------------|
| API | Interfície de programació |
| JWT | JSON Web Token |
| REST | Arquitectura de serveis web |
| HTTPS | Protocol segur de comunicació |

---

# 3. Conceptes bàsics

---

# 3.1 Protocol

El protocol de comunicació es fa mitjançant:
HTTPS + REST + JSON

Les peticions es realitzen a través de serveis REST utilitzant dades en format **JSON**.

---

# 3.2 Política de seguretat de contrasenyes

Després de **6 intents fallits de login**, el sistema bloqueja temporalment l’usuari.

Cada nou intent requereix esperar **90 segons**.

### Expressió regular de validació
^(?=.[0-9])(?=.[a-z])(?=.[A-Z])(?=.[!@#&()–[{}]:;',?/*~$^+=<>]).{8,20}$

Requisits:

- almenys un dígit
- almenys una lletra minúscula
- almenys una lletra majúscula
- almenys un caràcter especial
- longitud entre **8 i 20 caràcters**

---

# 3.3 Autenticació / Autorització

Per utilitzar l’API cal enviar un **token JWT** a la capçalera HTTP.

### Exemple
Authorization: Bearer [TOKEN]

Aquest token s’obté a partir de credencials proporcionades per l’administrador de la plataforma.

---

# 3.4 Normalització i validació d’identificadors (NIF / NIE)

Els serveis que reben el paràmetre `identifier` assumeixen que aquest correspon a un **NIF o NIE vàlid**.

Validacions aplicades:

- eliminació d'espais
- validació del format
- normalització a majúscules

### Formats acceptats

NIF
7 o 8 digits + lletra

NIE
X,Y,Z + 7 digits + lletra

### Expressió regular

^([0-9]{7,8}|[XYZ][0-9]{7})[A-Za-z]$


Aquesta validació és comuna a totes les APIs que utilitzen el camp **identifier**.

---

# 4. APIs exposades

---

# 4.1 Autenticació d'usuaris

## Endpoint
POST /eacat/rest/api/auth/user/authenticate

### Headers
Content-Type: application/json

Authorization: Bearer [JWT_TOKEN]

### Body

```json
{
  "identifier": "[USER_DNI]",
  "password": "[USER_PASSWORD]"
}
```

### Resposta correcta

```json
{
  "ok": true,
  "messageCode": null,
  "messageDesc": null,
  "entity": {
    "userIdentifier": "90184217K",
    "userName": "John Bonham"
  }
}
```

### Errors possibles

| Code | Descripció |
|--------|------------|
| FAIL | Authentication fails |
| RENEW_PASSWORD | Renew password required |
| DISABLED_USER | Disabled user |
| DISABLED_TEMPORARILY | User temporarily disabled |
| UNKNOWN_USER | User not known |


## 4.2 Entitats d'usuaris

Retorna totes les entitats associades a l'usuari.

### Endpoint
POST /eacat/rest/api/auth/user/load-entities

### Headers
Content-Type: application/json

Authorization: Bearer [JWT_TOKEN]

### Body

```json
{
  "identifier": "[USER_DNI]"
}
```

### Exemple resposta

```json
{
  "ok": true,
  "entity": [
    {
      "ine10": "9612050006",
      "name": "Departament de Drets Socials",
      "cif": "Q0801175A",
      "defaultEntity": true,
      "userEmail": "email@email.cat"
    }
  ]
}
```

### Errors possibles

| Code | Descripció |
|--------|------------|
| UNKNOWN_USER | user not known |
| DISABLED_USER | Disabled user |
| NOT_ALLOWED_WHITESPACE | Input contains whitespace |


## 4.3 Rols d’un usuari en una entitat i una aplicació

Permet consultar els rols d’un usuari en una entitat i una aplicació.

### Endpoint

POST /eacat/rest/api/auth/user/load-entity-profiles

### Headers
Content-Type: application/json

Authorization: Bearer [JWT_TOKEN]

### Possibles combinacions

<img width="668" height="400" alt="image" src="https://github.com/user-attachments/assets/584b75f4-0c93-4669-8808-8537c3b9db88" />

### Body

```json
{
  "identifier": "[identifier]",
  "applicationCode": "[APP_CODE]",
  "ine10": "[ENTITY_INE10]",
  "profileName": "[Profile Name]"
}
```

### Exemple resposta
```json
{
  "ok": true,
  "entity": [
    {
      "ine10": "21",
      "identifier": "90184217K",
      "profiles": [
        {
          "profileName": "ADM_SERV"
        }
      ]
    }
  ]
}
```

### Errors possibles

| Code | Descripció |
|--------|------------|
| UNKNOWN_USER | user not known |
| DISABLED_USER | Disabled user |
| UNKNOWN_ENTITY | Unknown entity |
| IDENTIFIER_NO_BELONGS_TO_ENTITY | User not belongs to the entity |
| UNKNOWN_APPLICATION | Unknown application |
| MISSING_IDENTIFIER_OR_ENTITY | Validate that at least one of the two comes (identifier or ine10) |
| NOT_ALLOWED_WHITESPACE | Input contains whitespace characters that are not allowed |
| NO_PROFILES_FOUND | No active profiles found for user |


## 4.4 Creació / modificació de les credencials del usuari

Només disponible per aplicacions de confiança (ex: ValidAPP).

### Endpoint
POST /eacat/rest/api/auth/user/create-credentials

### Headers
Content-Type: application/json

Authorization: Bearer [JWT_TOKEN]

### Body

```json
{
  "name": "[USER_NAME]",
  "identifier": "[USER_IDENTIFIER]",
  "password": "[USER_PASSWORD]",
  "recoverEmail": "[RECOVER_EMAIL]"
}
```

### Exemple resposta

```json
{
  "ok": true,
  "entity": {
    "name": "Jimmy Page",
    "identifier": "90184217K",
    "password": "LesPaul243$!",
    "recoverEmail": "Jimmy.Page@correu.cat"
  }
}
```

### Errors possibles

| Code | Descripció |
|--------|------------|
| UNKNOWN_ERROR | user not known |
| PASSWORD_NOT_VALID | Password policy not valid |
