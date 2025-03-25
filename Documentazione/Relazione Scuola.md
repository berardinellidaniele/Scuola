
## Create table

```sql
CREATE TABLE Classe
(
 ID_Classe INT PRIMARY KEY IDENTITY NOT NULL,
 numero TINYINT NOT NULL,
 sezione VARCHAR(10) NOT NULL
)

CREATE TABLE Insegnante
(
 ID_Insegnante INT PRIMARY KEY IDENTITY NOT NULL,
 cod_fiscale VARCHAR(255) NOT NULL,
 nome VARCHAR(255) NOT NULL,
 cognome VARCHAR(255) NOT NULL,
 data_nascita DATE NOT NULL
)

CREATE TABLE Materia
(
 ID_Materia INT PRIMARY KEY IDENTITY NOT NULL,
 nome VARCHAR(255) NOT NULL
)

CREATE TABLE Studente
(
 ID_Studente INT PRIMARY KEY IDENTITY NOT NULL,
 nome VARCHAR(255) NOT NULL,
 cognome VARCHAR(255) NOT NULL,
 data_nascita DATE NOT NULL,
 luogo_nascita VARCHAR(255) NOT NULL,
 ID_Classe INT NOT NULL FOREIGN KEY REFERENCES Classe(ID_Classe)
)

CREATE TABLE Insegnamento
(
 ID_Insegnante INT NOT NULL,
 ID_Classe INT NOT NULL,
 ID_Materia INT NOT NULL,
 PRIMARY KEY (ID_Insegnante, ID_Classe, ID_Materia),
 FOREIGN KEY (ID_Insegnante) REFERENCES Insegnante(ID_Insegnante),
 FOREIGN KEY (ID_Classe) REFERENCES Classe(ID_Classe),
 FOREIGN KEY (ID_Materia) REFERENCES Materia(ID_Materia)
)
```

## Popolamento

```sql
-- inserimento per le classi
INSERT INTO Classe (numero, sezione) VALUES (1, 'A');
INSERT INTO Classe (numero, sezione) VALUES (2, 'B');

-- inserimento per gli insegnanti
INSERT INTO Insegnante (cod_fiscale, nome, cognome, data_nascita) VALUES 
('MRCMRC06M26I608Q', 'Marco', 'Mudric', '2006-08-26'),
('BRRDNL06E24I608Y', 'Daniele', 'Berardinelli', '2006-05-24');

-- inserimento di materie 
INSERT INTO Materia (nome) VALUES ('Matematica');
INSERT INTO Materia (nome) VALUES ('Informatica');
INSERT INTO Materia (nome) VALUES ('Italiano');

-- inserimento di studenti
INSERT INTO Studente (nome, cognome, data_nascita, luogo_nascita, ID_Classe) VALUES 
('Giampaolo', 'Magi', '2006-09-15', 'Fano', 1),
('Claudio', 'Lenci', '2006-05-24', 'Senigallia', 1),
('Giacomo', 'Principi', '2006-11-05', 'Montesicuro', 2);

-- la relazione ternaria
INSERT INTO Insegnamento (ID_Insegnante, ID_Classe, ID_Materia) VALUES 
(1, 1, 1), -- Marco Mudric insegna matematica alla classe 1A
(1, 1, 3), -- Marco Mudric insegna italiano alla classe 1A
(2, 2, 2), -- Daniele Berardinelli insegna informatica alla classe 2B
(2, 1, 2); -- Daniele Berardinelli insegna informatica anche alla classe 1A

```

## Query

Query 1

> Recuperare tutti gli studenti che frequentano una specifica classe, identificata dal numero e dalla sezione.

```sql
SELECT * FROM Studente
INNER JOIN Classe ON Studente.ID_Classe = Classe.ID_Classe
WHERE Classe.numero = X AND Classe.sezione = 'Y'; -- ovviamente X è il numero della classe e 'Y' fa riferimento alla sezione, questa è una generalizzazione che nel programma vengono inseriti dall'utente
```

Query 2

> Elencare tutti gli insegnanti che insegnano in una determinata classe.

```sql
SELECT DISTINCT * FROM Insegnante
INNER JOIN Insegnamento ON Insegnante.ID_Insegnante = Insegnamento.ID_Insegnante
WHERE Insegnamento.ID_Classe = (SELECT ID_Classe FROM Classe WHERE numero = x and sezione = 'Y');
```

Query 3

> Trovare tutte le materie insegnate da un determinato insegnante e le classi in cui le insegna.

```sql
SELECT Materia.nome AS materia, Classe.numero, Classe.sezione
FROM Insegnamento
JOIN Materia ON Insegnamento.ID_Materia = Materia.ID_Materia
JOIN Classe ON Insegnamento.ID_Classe = Classe.ID_Classe
WHERE Insegnamento.ID_Insegnante = 1;
```

Query 4

> Individuare il numero totale di studenti per ciascuna classe, ordinando il risultato in ordine decrescente.

```sql
SELECT Classe.numero, Classe.sezione, COUNT(Studente.ID_Studente) AS num_studenti FROM Classe
LEFT JOIN Studente ON Classe.ID_Classe = Studente.ID_Classe
GROUP BY Classe.ID_Classe,Classe.numero, Classe.sezione
ORDER BY num_studenti DESC;
```

Query 5

> Contare quanti insegnanti insegnano in più di una classe e restituire il numero totale di classi in cui ognuno insegna.

```sql
SELECT Insegnante.cod_fiscale, Insegnante.nome, Insegnante.cognome, COUNT(DISTINCT Insegnamento.ID_Classe) AS num_classi
FROM Insegnante
INNER JOIN Insegnamento ON Insegnante.ID_Insegnante = Insegnamento.ID_Insegnante
GROUP BY Insegnante.ID_Insegnante, Insegnante.cod_fiscale, Insegnante.nome, Insegnante.cognome
HAVING COUNT(DISTINCT Insegnamento.ID_Classe) > 1;
```

