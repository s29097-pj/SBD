### Ćwiczenie 7-1: Tworzenie i używanie profilu

1. **Tworzenie użytkownika INVENTORY**

Uruchom skrypt `lab_07_01.sh`:

```text
cd $HOME/labs

./lab_07_01.sh
```

Skrypt tworzy użytkownika `inventory` z hasłem `verysecure` i przypisuje mu przestrzeń tabel `inventory`.

2. **Tworzenie profilu HRPROFILE**

W Enterprise Manager:

- **Server** > **Security** > **Profiles** > **Create**

- **Name**: `HRPROFILE`

- **Idle Time (Minutes)**: `15`

- Pozostałe ustawienia: `DEFAULT`

3. **Ustawienie parametru RESOURCE_LIMIT**

W Enterprise Manager:

- **Server** > **Database Configuration** > **Initialization Parameters**

- Wyszukaj `resource_limit` i ustaw na `TRUE`

### Ćwiczenie 7-2: Tworzenie ról

1. **Tworzenie roli HRCLERK**

```sql

CREATE ROLE HRCLERK;

GRANT SELECT, UPDATE ON HR.EMPLOYEES TO HRCLERK;

```

2. **Tworzenie roli HRMANAGER**

```sql

CREATE ROLE HRMANAGER;

GRANT INSERT, DELETE ON HR.EMPLOYEES TO HRMANAGER;

GRANT HRCLERK TO HRMANAGER;

```

### Ćwiczenie 7-3: Tworzenie i konfiguracja użytkowników

1. **Tworzenie użytkowników**

| Użytkownik | Profil     | Rola        | Hasło początkowe |
|------------|------------|-------------|------------------|
| DHAMBY     | HRPROFILE  | HRCLERK     | newuser          |
| RPANDYA    | HRPROFILE  | HRCLERK     | newuser          |
| JGOODMAN   | HRPROFILE  | HRMANAGER   | newuser          |

Przykład dla użytkownika DHAMBY:

```sql

CREATE USER DHAMBY

IDENTIFIED BY newuser

PROFILE HRPROFILE

PASSWORD EXPIRE;

GRANT CONNECT, HRCLERK TO DHAMBY;

```

2. **Testowanie uprawnień**

- **DHAMBY (HRCLERK)** może odczytywać i aktualizować, ale nie usuwać:

```sql

SELECT salary FROM HR.EMPLOYEES WHERE employee_id=197;  -- Działa

DELETE FROM HR.EMPLOYEES WHERE employee_id=197;         -- Błąd ORA-01031

```

- **JGOODMAN (HRMANAGER)** może usuwać (ale wykonujemy rollback):

```sql

DELETE FROM HR.EMPLOYEES WHERE employee_id=197;  -- Działa

ROLLBACK;                                        -- Cofnięcie zmian

```

3. **Test limitu czasu bezczynności**

Po 15 minutach bezczynności:

```sql

ORA-02396: exceeded maximum idle time, please connect again

```

### Odpowiedzi na pytania

1. **Gdzie przechowywany jest usunięty wiersz?**

W przestrzeni undo (UNDO tablespace).

2. **Jakie przestrzenie tabel używają nowi użytkownicy?**

Domyślna stała (USERS) i tymczasowa (TEMP) przestrzeń tabel.

3. **Dlaczego nowi użytkownicy mogą się łączyć?**

Rola `CONNECT` zawiera przywilej `CREATE SESSION`.

### Całość w bloku kodu Markdown:

```markdown

# Laboratorium 7: Zarządzanie użytkownikami i profilami

## Ćwiczenie 7-1: Tworzenie profilu

### Krok 1: Utworzenie użytkownika INVENTORY

```bash

cd /home/oracle/labs

./lab_07_01.sh

```

### Krok 2: Tworzenie profilu HRPROFILE

```sql

CREATE PROFILE HRPROFILE LIMIT IDLE_TIME 15;

```

### Krok 3: Aktywacja limitów zasobów

```sql

ALTER SYSTEM SET RESOURCE_LIMIT=TRUE;

```

## Ćwiczenie 7-2: Tworzenie ról

### Rola HRCLERK

```sql

CREATE ROLE HRCLERK;

GRANT SELECT, UPDATE ON HR.EMPLOYEES TO HRCLERK;

```

### Rola HRMANAGER

```sql

CREATE ROLE HRMANAGER;

GRANT INSERT, DELETE ON HR.EMPLOYEES TO HRMANAGER;

GRANT HRCLERK TO HRMANAGER;

```

## Ćwiczenie 7-3: Konfiguracja użytkowników

### Tworzenie użytkowników

```sql

-- David Hamby (DHAMBY)

CREATE USER DHAMBY

IDENTIFIED BY newuser

PROFILE HRPROFILE

PASSWORD EXPIRE;

GRANT CONNECT, HRCLERK TO DHAMBY;

-- Rachel Pandya (RPANDYA)

CREATE USER RPANDYA

IDENTIFIED BY newuser

PROFILE HRPROFILE

PASSWORD EXPIRE;

GRANT CONNECT, HRCLERK TO RPANDYA;

-- Jenny Goodman (JGOODMAN)

CREATE USER JGOODMAN

IDENTIFIED BY newuser

PROFILE HRPROFILE

PASSWORD EXPIRE;

GRANT CONNECT, HRMANAGER TO JGOODMAN;

```

### Testowanie

```sql

-- Logowanie jako DHAMBY (po zmianie hasła na 'oracle')

CONNECT DHAMBY/oracle;

SELECT * FROM HR.EMPLOYEES;  -- Powinno działać

DELETE FROM HR.EMPLOYEES WHERE employee_id=197;  -- Błąd: ORA-01031

-- Logowanie jako JGOODMAN (po zmianie hasła na 'oracle')

CONNECT JGOODMAN/oracle;

DELETE FROM HR.EMPLOYEES WHERE employee_id=197;  -- Powinno działać

ROLLBACK;  -- Cofnięcie zmiany

```

### Weryfikacja limitu bezczynności

```sql

-- Po 15 minutach bezczynności:

SELECT * FROM HR.EMPLOYEES;  -- Błąd: ORA-02396

```

## Odpowiedzi na pytania

1. Usunięty wiersz jest przechowywany w przestrzeni undo.

2. Nowi użytkownicy używają domyślnych przestrzeni: USERS (dane) i TEMP (tymczasowa).

3. Użytkownicy mogą się łączyć, ponieważ rola CONNECT zawiera przywilej CREATE SESSION.

## Diagram hierarchii ról
```mermaid
graph TD
    U1[DHAMBY] --> R1[HRCLERK]
    U2[RPANDYA] --> R1
    U3[JGOODMAN] --> R2[HRMANAGER]
    R2 --> R1
    R1 --> T[HR.EMPLOYEES]
    R2 --> I[INSERT]
    R2 --> D[DELETE]
    R1 --> S[SELECT]
    R1 --> U[UPDATE]
````

