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
