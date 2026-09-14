Esercizio: Alternanza 4 pompe, 1 alla volta, 2 soglie, con isteresi.

# Funzionalità

- Sotto la soglia minima -> disattiva pompa
- Sopra la soglia massima -> comanda pompa
- Una volta comandata la pompa sopra la soglia massima, rimane comandata finché non livello raggiunge meno di soglia minima (isteresi)
- Alternanza pompe da prima a ultima, in ordine numerico (1,2,3,4)
- Massimo 1 pompa comandata alla volta
- Quando la pompa attualmente comandata non è usabile, viene comandata la prima prossima pompa usabile
- Le pompe non usabili vengono saltate automaticamente
