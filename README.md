# ⚠️ Fehlerbehandlung / Exceptions in PL/SQL

In PL/SQL können – wie in anderen Programmiersprachen – Fehler gezielt behandelt werden.  
Dieses Kapitel zeigt die **5 wichtigsten Techniken der Fehlerbehandlung**, basierend auf dem Unterricht und dem Handout.

---

## 📌 Inhalt

1. ✅ Named System Exceptions  
2. ❗ Unnamed Programmer-defined Exceptions  
3. 🏷️ Named Programmer-defined Exceptions  
4. 🔗 PRAGMA EXCEPTION_INIT (Verbindung mit Fehlercode)  
5. 📦 Fehlerbehandlung über Packages  

---

## ✅ 1. Named System Exceptions

**Definition:**  
Oracle liefert viele vordefinierte Fehlercodes, die automatisch erkannt werden.  
Z. B. bei `SELECT INTO`, wenn **kein Datensatz** oder **mehrere Datensätze** zurückgegeben werden.

**Typische Fehler:**
- `NO_DATA_FOUND` – Kein Datensatz gefunden  
- `TOO_MANY_ROWS` – Mehrere Zeilen zurückgegeben  

**🧪 Beispiel:**
```sql
BEGIN
  SELECT ename INTO testvar FROM emp WHERE deptno = 50;
EXCEPTION
  WHEN NO_DATA_FOUND OR TOO_MANY_ROWS THEN
    DBMS_OUTPUT.PUT_LINE('❗ SELECT INTO Fehler – keine oder zu viele Zeilen!');
END;
```

---

## ❗ 2. Unnamed Programmer-defined Exceptions

**Definition:**  
Mit `RAISE_APPLICATION_ERROR` kann man selbst Fehler mit einem eigenen Code und Text erzeugen.  
Diese Fehler sind **sichtbar für den Aufrufer** (z. B. in einem Tool oder im Clientcode).

**🧪 Beispiel:**
```sql
BEGIN
  IF sal > 5000 THEN
    RAISE_APPLICATION_ERROR(-20001, '❗ Gehalt ist zu hoch!');
  END IF;
END;
```

🔸 **Fehlercodes müssen im Bereich -20000 bis -20999 liegen.**  
🔸 **Nur `RAISE_APPLICATION_ERROR` erzeugt tatsächlich eine sichtbare Fehlermeldung.**

---

## 🏷️ 3. Named Programmer-defined Exceptions

**Definition:**  
Selbst deklarierte Exceptions im `DECLARE`-Block – sie können mit `RAISE` ausgelöst und im `EXCEPTION`-Block behandelt werden.

**🧪 Beispiel:**
```sql
DECLARE
  EMP_SAL_TOO_HIGH EXCEPTION;
BEGIN
  IF sal > 5000 THEN
    RAISE EMP_SAL_TOO_HIGH;
  END IF;
EXCEPTION
  WHEN EMP_SAL_TOO_HIGH THEN
    DBMS_OUTPUT.PUT_LINE('❗ Benutzerdefinierter Fehler: Gehalt zu hoch!');
END;
```

---

## 🔗 4. PRAGMA EXCEPTION_INIT

**Definition:**  
Wenn du einen Oracle-Fehlercode (z. B. -2292 bei Fremdschlüsselverletzung) mit einem **selbst gewählten Namen** verbinden möchtest, geht das mit `PRAGMA EXCEPTION_INIT`.

**🧪 Beispiel:**
```sql
DECLARE
  fk_violation EXCEPTION;
  PRAGMA EXCEPTION_INIT(fk_violation, -2292); -- ORA-02292: FK-Verletzung
BEGIN
  DELETE FROM dept WHERE deptno = 10;
EXCEPTION
  WHEN fk_violation THEN
    DBMS_OUTPUT.PUT_LINE('❗ Fremdschlüsselverletzung: Löschung nicht möglich!');
END;
```

---

## 📦 5. Fehlerbehandlung über Packages

**Definition:**  
Eine Exception wird im Package-Header deklariert, im Body ausgelöst und im **aufrufenden Block** behandelt.  
Das ist nützlich für **saubere Fehlerkommunikation über Prozedurgrenzen hinweg**.

**📦 Beispielstruktur:**
```sql
-- Package-Header
CREATE OR REPLACE PACKAGE exception_pkg IS
  EMP_SAL_TOO_HIGH EXCEPTION;
  PROCEDURE check_salary(p_ename VARCHAR2);
END;
```

```sql
-- Package-Body
CREATE OR REPLACE PACKAGE BODY exception_pkg IS
  PROCEDURE check_salary(p_ename VARCHAR2) IS
    v_sal emp.sal%TYPE;
  BEGIN
    SELECT sal INTO v_sal FROM emp WHERE ename = p_ename;
    IF v_sal > 5000 THEN
      RAISE EMP_SAL_TOO_HIGH;
    END IF;
  END;
END;
```

```sql
-- Aufrufender Block
BEGIN
  exception_pkg.check_salary('KING');
EXCEPTION
  WHEN exception_pkg.EMP_SAL_TOO_HIGH THEN
    DBMS_OUTPUT.PUT_LINE('❗ KING verdient zu viel – Exception aus Package!');
END;
```

---

## ✅ Gesamtbeispiel aus dem Unterricht

```sql
DECLARE
    testvar VARCHAR(32);
    highest_sal NUMBER;

    -- Benutzerdefinierte Ausnahme
    EMP_SAL_TOO_HIGH EXCEPTION;

    -- Verbindung mit Oracle-Fehlercode (optional sichtbar)
    PRAGMA EXCEPTION_INIT(EMP_SAL_TOO_HIGH, -20012);
BEGIN
    SELECT sal INTO highest_sal FROM emp WHERE ename = 'KING';

    IF sal > 5000 THEN
        RAISE EMP_SAL_TOO_HIGH;
    END IF;

    -- Fehler, der vom Aufrufer gesehen werden kann
    RAISE_APPLICATION_ERROR(-20101, '❗ Something went wrong');

EXCEPTION
    WHEN NO_DATA_FOUND OR TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('❗ SELECT INTO Fehler!');
    WHEN EMP_SAL_TOO_HIGH THEN
        DBMS_OUTPUT.PUT_LINE('❗ Gehalt von KING zu hoch!');
    WHEN OTHERS THEN
        RAISE_APPLICATION_ERROR(-20102, '❗ Unerwarteter Fehler aufgetreten …');
END;
```

---

## 📌 Vergleichstabelle

| Typ                          | Sichtbar für Aufrufer? | Deklariert im Code? | Eigenes Label möglich? | Beispiel                      |
|-----------------------------|------------------------|----------------------|--------------------------|-------------------------------|
| `NO_DATA_FOUND`             | 🔹 Ja                 | ❌ Nein              | ❌ Nein                 | System Exception              |
| `RAISE_APPLICATION_ERROR`   | ✅ Ja                 | ✅ Nein              | ✅ Ja                   | `RAISE_APPLICATION_ERROR(...)`|
| Named Programmer Exception  | ❌ Nein               | ✅ Ja                | ✅ Ja                   | `RAISE my_exception`          |
| PRAGMA + Exception          | ✅ Ja                 | ✅ Ja                | ✅ Ja                   | `PRAGMA EXCEPTION_INIT(...)`  |

---

## 👨‍🏫 Fazit

Fehler in PL/SQL lassen sich **gezielt behandeln** – du kannst:
- Systemfehler abfangen,
- eigene Fehler auslösen,
- Fehler benennen und weiterreichen,
- Exceptions zwischen Packages und Aufrufern kommunizieren.

Nutze diese Möglichkeiten, um **robuste und verständliche Fehlerbehandlung** zu implementieren 💡

---
# 📛 Fehlerbehandlung in PL/SQL – Quiz & Komplettbeispiel

> Ergänzung zum Hauptkapitel: Exceptions in PL/SQL (Stand: 06.04.2025)

---

## 🧠 Multiple-Choice-Test – Fehlerbehandlung

Wähle jeweils **eine oder mehrere** richtige Antworten!

### ❓ 1. Was ist eine benannte (named) Exception in PL/SQL?

- [ ] a) Eine Exception, die Oracle automatisch behandelt  
- [x] b) Eine selbst definierte Exception mit `EXCEPTION`  
- [x] c) Eine Exception, die im `DECLARE`-Block benannt wird  
- [ ] d) Eine Exception, die mit `RAISE_APPLICATION_ERROR` erzeugt wird  

---

### ❓ 2. Welche Exception wird ausgelöst, wenn `SELECT ... INTO` keine Zeile findet?

- [ ] a) TOO_MANY_ROWS  
- [ ] b) INVALID_NUMBER  
- [x] c) NO_DATA_FOUND  
- [ ] d) VALUE_ERROR  

---

### ❓ 3. Wie verknüpft man eine benannte Exception mit einem Fehlercode?

- [ ] a) WITH ERRORCODE …  
- [ ] b) USING -CODE  
- [x] c) `PRAGMA EXCEPTION_INIT(...)`  
- [ ] d) `EXCEPTION_MAP(...)`

---

### ❓ 4. Welche Aussage zu `RAISE_APPLICATION_ERROR` ist korrekt?

- [x] a) Damit lassen sich benutzerdefinierte Fehler auslösen  
- [ ] b) Es muss im `EXCEPTION`-Block stehen  
- [ ] c) Es ist nur in Prozeduren erlaubt  
- [x] d) Der Fehlercode muss zwischen -20000 und -20999 liegen  

---

### ❓ 5. Was passiert bei `WHEN OTHERS THEN ...`?

- [ ] a) Nur NO_DATA_FOUND wird behandelt  
- [x] b) Es fängt **alle nicht explizit behandelten** Fehler ab  
- [ ] c) Es ignoriert alle Systemfehler  
- [ ] d) Es beendet das Programm sofort  

---

## 🧩 Komplettbeispiel – Alle Exception-Arten

### 📦 1. Exception im Package

```sql
CREATE OR REPLACE PACKAGE my_exceptions_pkg IS
  ex_sal_too_high EXCEPTION;
  PRAGMA EXCEPTION_INIT(ex_sal_too_high, -20012);
END;
/
```

---

### 💥 2. Hauptblock mit allen Exception-Typen

```sql
DECLARE
  v_sal emp.sal%TYPE;
  v_bonus emp.comm%TYPE;

  bonus_too_high EXCEPTION; -- Named Programmer-defined
BEGIN
  SELECT sal, comm INTO v_sal, v_bonus FROM emp WHERE ename = 'CHRISTOPH';

  IF v_bonus IS NOT NULL AND v_bonus > 1000 THEN
    RAISE bonus_too_high;
  END IF;

  IF v_sal > 5000 THEN
    RAISE my_exceptions_pkg.ex_sal_too_high;
  END IF;

  DBMS_OUTPUT.PUT_LINE('Bonus & Gehalt sind okay!');

EXCEPTION
  WHEN bonus_too_high THEN
    DBMS_OUTPUT.PUT_LINE('❗ Bonus ist zu hoch!');

  WHEN my_exceptions_pkg.ex_sal_too_high THEN
    DBMS_OUTPUT.PUT_LINE('⚠️ Fehlercode -20012: Gehalt ist zu hoch!');

  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('❌ Kein Mitarbeiter CHRISTOPH gefunden!');

  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('❗ Unbekannter Fehler: ' || SQLERRM);
END;
/
```

---

✅ Dieses Beispiel kombiniert **alle Exception-Techniken**, die im Unterricht und für den Test relevant sind.

