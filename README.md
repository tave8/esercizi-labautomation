# Esercizio: 4 pompe, 3 max, 4 soglie, alternanza meno ore

BUG: Quando la Pompa 1 ha il contatore secondi lavorati a 0, dovrebbe essere comandata perché 0 è più piccolo degli altri contatori.

Ad ogni soglia max raggiunta, comanda la pompa con minor ore lavorate.

Tieni in considerazione che una pompa può essere non usabile o già comandata.

Quindi, ad ogni soglia max raggiunta, comanda la pompa con minor ore lavorate, delle pompe usabili e non comandate.

```
scan cycle:
    prossima_pompa_da_comandare = 0
    ultimo_ore_lavorate = 0

    foreach pompa in pompe:
        if pompa.usabile AND NOT pompa.comandata:
            if pompa.ore_lavorate < ultimo_ore_lavorate OR ultimo_ore_lavorate = 0:
                prossima_pompa_da_comandare = pompa
                ultimo_ore_lavorate = pompa.ore_lavorate

    comanda prossima_pompa_da_comandare, se diversa da 0


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
strategia_alternanza_avvio: bool
strategia_alternanza_meno_ore_lavorate: bool

TEMP

OUTPUT
err_strategia_alternanza_ambigua: bool

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
strategia_alternanza_input: int

TEMP
strategia_alternanza_attuale: int

OUTPUT
ERR_STRG_ALTERNANZA_INVALIDA: bool
`

# Alternanza giornaliera

Alternanza giornaliera significa alternare le pompe al raggiungimento di una certa ora.

Per fare questo, possiedo due strumenti:

- L'ora e minuto del PLC, che posso comparare con un orario predefinito (es: quando scocca mezzanotte, alterna pompe (imposta la prossima candidata))
- Gli Schedule Blocks, che permettono di impostare dei bit con logiche di tempo specifiche (ogni giorno alle ore X, da ora Y ecc. -- alza il bit di una variabile)

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
numero_sec_preset

VARIABILI TEMP
pompa_1_numero_sec
pompa_2_numero_sec
pompa_1_superato_numero_sec
pompa_2_superato_numero_sec

Fintanto che una pompa è comandata, viene contato il tempo in cui è comandata. Se questo tempo supera il numero ore preset, allora questa pompa ha raggiunto il suo numero ore. Quando una pompa raggiunge il suo numero ore, il suo numero ore viene resettato e la prossima pompa candidata è l'altra pompa.

p1_superato_numero_sec_lavorati := p1_numero_sec_lavorati > numero_sec_preset

FINTANTO CHE pompa_1 E' COMANDATA:
SE pompa_1_superato_numero_sec:
prossima_pompa_candidata := pompa_2

incrementa numero sec di pompa_1

# Esercizio: Alternanza 4 pompe, 1 alla volta, 2 soglie, con isteresi.

## Funzionalità

- Sotto la soglia minima -> disattiva pompa
- Sopra la soglia massima -> comanda pompa
- Una volta comandata la pompa sopra la soglia massima, rimane comandata finché non livello raggiunge meno di soglia minima (isteresi)
- Alternanza pompe da prima a ultima, in ordine numerico (1,2,3,4)
- Massimo 1 pompa comandata alla volta
- Quando la pompa attualmente comandata non è usabile, viene comandata la prima prossima pompa usabile
- Le pompe non usabili vengono saltate automaticamente
