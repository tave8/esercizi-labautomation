# Esercizio: 2 pompe, 2 max, 4 soglie, scelta spegnimento graduale o livello minimo



strategia:

0 = livello minimo

1 = spegnimento graduale



Lo spegnimento graduale deve tenere in considerazione tutte le coppie di soglie. Invece il livello minimo deve considerare solo la soglia minima comune. 



Strategia Spegnimento graduale:



max2 -- rising edge: attiva seconda pompa



min2 -- falling edge: disattiva seconda pompa



max1 -- rising edge: attiva prima pompa



min1 -- falling edge: disattiva prima pompa





Strategia Livello minimo:



max2 -- rising edge: attiva seconda pompa



min2 -- (ignora)



max1 -- rising edge: attiva prima pompa



min1 -- falling edge: disattiva tutte pompe





\## Ragionamento A



L'idea è quella di cercare di accorpare, unire le due strategie, cercando di generalizzare i punti in comune, creando difatti una nuova logica che include, alterna tra entrambe. Tuttavia credo che potrebbe essere più intelligente lasciare le due strategie separate a livello dell'intero programma, lasciando che la strategia scelta decida quale sotto-programma caricare. Con ogni sotto-programma si intende un insieme di componenti che eseguono una sola strategia, ad esempio solo livello minimo e solo spegnimento graduale.

# 

# Esercizio: 2 pompe, 2 max, misura di livello e richiesta attivazioni

Il livello attuale non è più definito da soglie discrete, ma è un valore analogico. Il numero di pompe da attivare dipende dal livello raggiunto. Il livello raggiunto, espresso come valore analogico in input, va scalato. Per scalare questo livello analogico in percentuale, e quindi per determinare la percentuale di livello raggiunto, serve scalare questo livello analogico in input. Serve quindi definire il livello massimo, che rappresenta il 100%.

```
INPUT
  lvl\\\\\\\_input (valore grezzo dal sensore. es: 0-10000)
  soglia\\\\\\\_min\\\\\\\_perc
  soglia\\\\\\\_max1\\\\\\\_perc
  soglia\\\\\\\_max2\\\\\\\_perc

COSTANTI
  min\\\\\\\_scala\\\\\\\_grezzo (minimo del valore grezzo del segnale. es: 0)
  max\\\\\\\_scala\\\\\\\_grezzo (massimo del valore grezzo del segnale. es: 10000) 
  min\\\\\\\_scala\\\\\\\_fisica (minimo fisico del sensore. es: 0cm)
  max\\\\\\\_scala\\\\\\\_fisica (massimo fisico del sensore. es: 150cm)

TEMP
  lvl\\\\\\\_perc
  passato\\\\\\\_soglia\\\\\\\_min
  passato\\\\\\\_soglia\\\\\\\_max1
  passato\\\\\\\_soglia\\\\\\\_max2

OUTPUT
  p1\\\\\\\_cmd
  p2\\\\\\\_cmd

```

livello\_% = (valore\_grezzo - grezzo\_min) \* 100 / (grezzo\_max - grezzo\_min)



## Prima implementazione

Nella prima implementazione ho 1 soglia minima (in comune) e 2 soglie massime. Ovviamente le soglie sono in percentuale.

Sotto la soglia minima, tutte le pompe sono disattivate. Al superamento della prima soglia massima in percentuale, una pompa viene attivata. Al superamento della seconda soglia massima in percentuale, l'altra pompa viene attivata.



# Esercizio: 4 pompe, 3 max, 6 soglie, spegnimento graduale semplice

Spegnimento graduale significa che viene disattivata la pompa associata alla soglia sotto la quale il livello è appena sceso. Bisogna considerare che questo accoppiamento forte tra pompa e coppia di soglie, viene "rotto" quando una pompa non è usabile. Questo significa che l'ordine iniziale di attivazione/disattivazione delle pompe, deve essere cambiato.

Assumiamo il caso più semplice, quello in cui non si considera il fatto che una pompa possa venire disattivata e che un'altra debba prendere il suo posto. Assumo quindi che, letteralmente, ad ogni coppia di soglia viene attivata o disattivata la stessa pompa nell'ordine.

```

max3

min3

max2

min2

max1

min1

```

La prima implementazione quindi non tiene in considerazione il fatto che una pompa, quando non è usabile, deve essere sostituita da una usabile. Bisogna quindi tenere traccia di questa pompa, nello specifico associare il fatto che questa nuova pompa ha sostituito un'altra pompa. Forse si può modificare l'ordine delle pompe, che deve essere sorgente di verità?

Esempio (4 pompe, 3 max):

```
Ordine iniziale:
     prima: pompa 1
     seconda: pompa 2
     terza: pompa 3

Pompa 3 non è usabile

Ordine modificato:
     prima: pompa 1
     seconda: pompa 2
     terza: pompa 4

Pompa 3 è usabile, Pompa 2 non è usabile

Ordine modificato:
     prima: pompa 1
     seconda: pompa 3
     terza: pompa 4


Pompa 4 non è usabile



Ordine modificato:
     prima: pompa 1
     seconda: pompa 3
     terza: pompa 0




```

Quindi l'ordine viene mantenuto finché una pompa diventa non usabile.











# Esercizio: 2 pompe, 2 max, 4 soglie, spegnimento graduale semplice

Spegnimento graduale si riferisce ad avere più soglie in cui le pompe vengono disattivate. Cioè, invece di disattivare tutte le pompe all'aver passato sotto l'unica soglia minima, ci sono più soglie minime.

Questo permette alla portata di essere gestita in maniera più "fluida", meno brusca.

Mi chiedo se debba esistere l'assunto che `min1 < max1 < min2 < max2`, o se abbia perfino senso domandarsi se abbia senso considerarlo.

"Spegnimento graduale" si riferisce a come le pompe vengono comandate e disattivate.

L'attributo "semplice" si riferisce alla decisione di quale pompa viene scelta per essere comandata o disattivata. Nello specifico, l'accoppiamento stretto tra soglia e identificativo pompa viene stabilito. Quindi soglia 1 avrà "assegnato" pompa 1, soglia 2 avrà assegnato pompa 2, e così via.

Nello specifico, stiamo parlando di soglia minima e massima. Una soglia minima è una soglia tale che raggiungere sotto di quella soglia causa la disattivazione di una pompa (visto che stiamo associando una pompa a una coppia di soglie). Una soglia massima è una soglia tale che raggiungere sopra di quella soglia causa l'attivazione di una pompa.

Quindi:

* *Raggiunto sotto soglia minima* è un caso di falling edge (1 -> 0)
* *Raggiunto sopra soglia massima* è un caso di rising edge (0 -> 1)

Visto che stiamo associando una pompa ad una coppia di soglie, e ogni coppia di soglie ha soglia minima e soglia massima, allora per una data coppia di soglie, la stessa pompa verrà attivata o disattivata.

```
    lvl
     ^
     |                                              ---
max2 | ---------------------------------------------------
     |            ---                               |  |
     |           |   |                             |    --
min2 | ---------------------------------------------------
     |          |     --                         --
     |         |        |                       |
max1 | ---------------------------------------------------
     |      ---        --             --       |
     |     |              |         |   |     |
min1 | ---------------------------------------------------
     |  ---               ----     |    |    |
     | |                      -----      ----
     |
     ----------------------------------------------------> t

```

# Esercizio: 4 pompe, 3 max, 4 soglie, alternanza giornaliera con meno ore

# Ragionamento A

Alterna le pompe 1 volta al giorno. La prossima pompa da comandare sarà quella con minor ore, e l'alternanza viene decisa solo 1 volta al giorno.

1 volta al giorno, viene deciso l'ordine in cui le pompe verranno comandate ad ogni soglia raggiunta.

Quell'ordine viene mantenuto fino al prossimo giorno, in cui l'ordine viene aggiornato.

Da una decisione alla prossima, l'ordine viene alterato solo se qualche pompa diventa non usabile.

`ordine\\\\\\\_prima\\\\\\\_pompa` e simili variabili contengono il numero di pompa da attivare, nell'ordine specificato (prima, seconda, ecc.).

Se non esista una pompa in questo ordine (sarà uguale a 0), allora significa che non ci sono abbastanza pompe con meno ore, quindi imposta la prima che puoi.

Quindi:

```
scan cycle:

  if e\\\\\\\_mezzanotte AND NOT deciso\\\\\\\_ordine:
    ordine\\\\\\\_prima\\\\\\\_pompa = pompa con meno ore lavorate tra quelle usabili
    ordine\\\\\\\_seconda\\\\\\\_pompa = pompa con meno ore lavorate tra quelle usabili AND pompa != ordine\\\\\\\_prima\\\\\\\_pompa
    ordine\\\\\\\_terza\\\\\\\_pompa = pompa con meno ore lavorate tra quelle usabili AND pompa != ordine\\\\\\\_seconda\\\\\\\_pompa
    ordine\\\\\\\_quarta\\\\\\\_pompa = pompa con meno ore lavorate tra quelle usabili AND pompa != odine\\\\\\\_terza\\\\\\\_pompa

    deciso\\\\\\\_ordine = true

  if NOT e\\\\\\\_mezzanotte:
    deciso\\\\\\\_ordine = false




```

Se qualche pompa diventa non usabile (avendo già l'ordine pompe) bisogna modificare in qualche modo l'ordine pompe, sostituendo la prima pompa usabile (indipendentemente da ore lavorate) con quella che è appena diventata non usabile.

Funzionamento: Ad ogni soglia massima raggiunta, viene comandata la pompa nel rispettivo ordine già deciso.

Ad esempio, alla soglia 1 massima, viene comandata la pompa ordine\_prima\_pompa. Alla soglia 2 massima, viene comandata la pompa ordine\_seconda\_pompa.

# Ragionamento B

L'implementazione più semplice prevede un ordine delle pompe da comandare. Questo ordine non tiene in considerazione né l'usabilità della pompa al momento del rinnovo ordine, né se la pompa è attualmente comandata. Questo perché questi ultimi sono stati che possono cambiare in qualsiasi momento dopo il rinnovo ordine. Quindi è come fare un piano di battaglia, e poi adattarsi una volta in campo.

Se una pompa (ad esempio, la prima pompa nell'ordine) risulta non usabile al momento di comando, allora la prima pompa usabile viene comandata. Quindi una volta rinnovato, l'ordine *non* viene modificato. Quindi se nessuna pompa diventa inusabile dal rinnovo dell'ordine, l'ordine al momento `t2` (poco prima del rinnovo dell'ordine X, ad esempio alle ore 23:59) mostra correttamente lo stesso ordine delle pompe, da quella con meno ore a quella con più ore, al momento `t1` (appena rinnovato l'ordine X, poco dopo le ore 00:00).

# Esercizio: 4 pompe, 3 max, 4 soglie, alternanza meno ore

BUG: Quando la Pompa 1 ha il contatore secondi lavorati a 0, dovrebbe essere comandata perché 0 è più piccolo degli altri contatori.

Ad ogni soglia max raggiunta, comanda la pompa con minor ore lavorate.

Tieni in considerazione che una pompa può essere non usabile o già comandata.

Quindi, ad ogni soglia max raggiunta, comanda la pompa con minor ore lavorate, delle pompe usabili e non comandate.

```
scan cycle:
    prossima\\\\\\\_pompa\\\\\\\_da\\\\\\\_comandare = 0
    ultimo\\\\\\\_ore\\\\\\\_lavorate = 0

    foreach pompa in pompe:
        if pompa.usabile AND NOT pompa.comandata:
            if pompa.ore\\\\\\\_lavorate < ultimo\\\\\\\_ore\\\\\\\_lavorate OR ultimo\\\\\\\_ore\\\\\\\_lavorate = 0:
                prossima\\\\\\\_pompa\\\\\\\_da\\\\\\\_comandare = pompa
                ultimo\\\\\\\_ore\\\\\\\_lavorate = pompa.ore\\\\\\\_lavorate

    comanda prossima\\\\\\\_pompa\\\\\\\_da\\\\\\\_comandare, se diversa da 0


```

# Esercizio: 2 pompe, 2 max, 3 soglie

## Soglia | Numero pompe da comandare

Sotto Soglia min 0
Sopra Soglia max 1 1
Sopra Soglia max 2 2

Il requisito è che al massimo 2 pompe siano comandate contemporaneamente, ma questo solo se entrambe sono usabili e se le due soglie massime sono raggiunte.

Se le due soglie massime sono raggiunte ma almeno una pompa non è usabile, bisogna comandare le altre pompe usabili.

# Come implementare la "strategia di alternanza"

## Ragionamento A

Ci sono tante variabili binarie quanti sono le strategie di alternanza, cioè ogni strategia di alternanza ha la propria variabile binaria. La strategia da applicare è ricevuta in input. Il cambio della strategia comporta un effetto istantaneo.

Serve un meccanismo per assicurarsi che esattamente una strategia sia attiva alla volta (non meno di una, non più di una), cioè che il sistema non si trovi mai ad avere informazioni ambigue su quale strategia di alternanza sta essendo applicata in un dato istante. Quindi servirà anche una variabile binaria per rilevare questa ambiguità, quindi errore, che andrà comunicato in output.

Serve anche un meccanismo per impostare una strategia di default, se nessuna strategia viene fornita.

`
INPUT
strategia\_alternanza\_avvio: bool
strategia\_alternanza\_meno\_ore\_lavorate: bool

TEMP

OUTPUT
err\_strategia\_alternanza\_ambigua: bool

`

## Ragionamento B

Ogni strategia di alternanza è rappresentata come un numero. Le strategia di alternanza possibili sono mappate in una variabile di numero intero.

La differenza tra "strategia alternanza input" e "strategia alternanza attuale" è permette il disaccoppiamento tra input e uso attuale. In poche parole, disaccoppiare "quello che ricevo in input" con "quello che il sistema sta ancora usando".

In questo modo si può costruire una logica del tipo "cambia strategia solo quando quella attuale ha soddisfatto una certa condizione" oppure "verifica prima che la strategia in input sia valida, poi cambia la strategia attuale che sia uguale a quella in input".

Mappatura:

## numero | strategia

0 avvio (default)
1 meno ore lavorate
1 giornaliera

`
INPUT
strategia\_alternanza\_input: int

TEMP
strategia\_alternanza\_attuale: int

OUTPUT
ERR\_STRG\_ALTERNANZA\_INVALIDA: bool
`

# Alternanza giornaliera

Alternanza giornaliera significa alternare le pompe al raggiungimento di una certa ora.

Per fare questo, possiedo due strumenti:

* L'ora e minuto del PLC, che posso comparare con un orario predefinito (es: quando scocca mezzanotte, alterna pompe (imposta la prossima candidata))
* Gli Schedule Blocks, che permettono di impostare dei bit con logiche di tempo specifiche (ogni giorno alle ore X, da ora Y ecc. -- alza il bit di una variabile)

Caso con 2 pompe: Quando scocca mezzanotte, modifica la prossima pompa candidata (ovviamente, solo se la prossima candidata è anche usabile).

Serve un meccanismo per eseguire la logica del cambiare la prossima pompa candidata, solo una volta, allo scoccare della mezzanotte. Forse un rising edge all'ora 24:00?

IF ora e minuto = 24:00 ALLORA
rising edge, cambia prossima pompa candidata

# Alternanza per numero ore (parte quella con meno ore lavorate)

In questa variante dell'alternanza pompe per numero ore, parte la pompa con meno ore lavorate.

Non serve quindi impostare il numero ore, ma vengono contate il numero ore per cui ogni pompa lavora.

# Alternanza per numero ore (reset ore lavorate)

In questa variente dell'altenrnanza pompe per numero ore, viene impostata il numero ore per cui ogni pompa deve lavorare, prima di alternarsi con la prossima pompa.

Due pompe si alternano quando ognuna ha raggiunto il suo numero ore preset.

INPUT
numero\_sec\_preset

VARIABILI TEMP
pompa\_1\_numero\_sec
pompa\_2\_numero\_sec
pompa\_1\_superato\_numero\_sec
pompa\_2\_superato\_numero\_sec

Fintanto che una pompa è comandata, viene contato il tempo in cui è comandata. Se questo tempo supera il numero ore preset, allora questa pompa ha raggiunto il suo numero ore. Quando una pompa raggiunge il suo numero ore, il suo numero ore viene resettato e la prossima pompa candidata è l'altra pompa.

p1\_superato\_numero\_sec\_lavorati := p1\_numero\_sec\_lavorati > numero\_sec\_preset

FINTANTO CHE pompa\_1 E' COMANDATA:
SE pompa\_1\_superato\_numero\_sec:
prossima\_pompa\_candidata := pompa\_2

incrementa numero sec di pompa\_1

# Esercizio: Alternanza 4 pompe, 1 alla volta, 2 soglie, con isteresi.

## Funzionalità

* Sotto la soglia minima -> disattiva pompa
* Sopra la soglia massima -> comanda pompa
* Una volta comandata la pompa sopra la soglia massima, rimane comandata finché non livello raggiunge meno di soglia minima (isteresi)
* Alternanza pompe da prima a ultima, in ordine numerico (1,2,3,4)
* Massimo 1 pompa comandata alla volta
* Quando la pompa attualmente comandata non è usabile, viene comandata la prima prossima pompa usabile
* Le pompe non usabili vengono saltate automaticamente

