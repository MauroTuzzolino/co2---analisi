# Chi inquina davvero di più? Analisi delle emissioni di CO₂ (1990-2024) con Power BI

Report interattivo in tre pagine su emissioni di CO₂, crescita economica e mix delle fonti energetiche, costruito su dati pubblici di Our World in Data.

> **Autore:** `Mauro Tuzzolino` · **LinkedIn:** `https://www.linkedin.com/in/mauro-tuzzolino-fullstack`

---

## Obiettivo

Rispondere a tre domande, mostrando che la stessa classifica cambia a seconda della metrica scelta:

1. **Chi inquina di più?** Emissioni totali, pro capite e cumulate storiche.
2. **Quali paesi sono riusciti a crescere riducendo le emissioni?** Analisi del "disaccoppiamento" tra PIL e CO₂ (1990-2022).
3. **Da dove arrivano le emissioni e come è cambiato il mix?** Carbone, petrolio, gas, cemento e flaring (1990-2024).

## Dati

| | |
|---|---|
| **Fonte** | [Our World in Data, repository `owid/co2-data`](https://github.com/owid/co2-data) |
| **File** | `owid-co2-data.csv` (50.411 righe × 79 colonne, 1750-2024) |
| **Unità** | CO₂ in milioni di tonnellate (Mt); pro capite in tonnellate per persona; PIL in dollari internazionali (fonte e anno base nel codebook) |

Le emissioni sono **territoriali** (quelle prodotte sul territorio del paese), non basate sui consumi.

## Struttura del report

| Pagina | Contenuto |
|---|---|
| **1. Chi inquina di più?** | Mappa e Top 10 per emissioni totali, pro capite (paesi con oltre 1 milione di abitanti) e cumulate dal 1750; slicer per anno |
| **2. Crescita senza emissioni?** | Dispersione variazione PIL vs variazione CO₂ per paese, classificazione in disaccoppiamento *assoluto / relativo / nessuno*, andamento di un singolo paese (base 100), Top 10 riduzioni |
| **3. Il mix delle fonti** | Mix mondiale e per paese nel tempo, Top 10 dipendenza dal carbone, Top 10 riduzione della quota di carbone |

## Principali risultati

**Pagina 1 (anno 2024)**
- Per **emissioni totali** guida la Cina (≈ 12.289 Mt, il 31,8% del totale mondiale), seguita da USA (≈ 4.904) e India (≈ 3.193).
- Per **emissioni pro capite** (paesi con oltre 1 milione di abitanti) i primi sono Qatar (41,3 t), Kuwait (26,2 t) e Bahrain (24,3 t); gli USA sono a 14,2 t, la Cina a 8,7 t, l'Italia a 5,1 t.
- Per **emissioni cumulate** storiche guidano gli USA (≈ 434.866 Mt), davanti a Cina (≈ 285.087) e Russia (≈ 122.808).

**Pagina 2 (1990-2022, 163 paesi con dati completi)**
- **40 paesi (24,5%)** hanno fatto crescere il PIL riducendo le emissioni (*disaccoppiamento assoluto*); 74 hanno visto le emissioni crescere meno del PIL (*relativo*); 49 nessun disaccoppiamento.
- Italia: PIL **+44,8%**, CO₂ **-22,4%**. Regno Unito: PIL +73,7%, CO₂ -48,3%. Germania: PIL +94,0%, CO₂ -36,7%.
- Cina e India sono in disaccoppiamento *relativo*: le emissioni sono cresciute (+372% e +390%), ma meno del PIL (+697% e +498%).

**Pagina 3 (1990-2024)**
- Mix mondiale: il carbone passa dal 39,6% al **42,7%**, il petrolio dal 39,6% al **30,5%**, il gas dal 17,3% al **21,7%**.
- Italia: carbone dal 16,1% al **3,8%**, gas dal 20,6% al **41,7%**.
- Più dipendenti dal carbone: Corea del Nord (91,4%), Sudafrica (84,4%), Cina (73,4%), Laos (68,4%), India (66,0%).
- Maggiori riduzioni della quota di carbone (punti percentuali): Irlanda (-39,5), Grecia (-39,2), Danimarca (-39,1), Regno Unito (-36,8). In senso opposto, Laos (+67,6), Indonesia (+49,2) e Filippine (+44,9).

## Metodologia

### 1. Pulizia dei dati (Power Query)

- Selezione di **16 colonne su 79**, scelte in base alle tre domande.
- Impostazione dei tipi con **impostazioni locali en-US**, perché il CSV usa il punto come separatore decimale.
- Rimozione delle entità duplicate "(GCP)" (stesse regioni calcolate con un'altra metodologia).
- Colonna **Tipo entità** (*Paese* / *Aggregato*): continenti, gruppi di reddito, "World", aviazione e navigazione internazionale sono separati dai paesi per evitare doppi conteggi.
- Filtro sul periodo **dal 1990** (dati affidabili) e rimozione delle righe senza CO₂.
- Risultato: **8.204 righe**, 216 paesi e 19 aggregati.
- Rinomina delle colonne con le unità di misura.

### 2. Modello e misure DAX

Tabella unica, con le logiche di calcolo spostate in **misure DAX**. Alcuni esempi:

```dax
CO2 Paesi (Mt) =
CALCULATE(SUM('CO2'[CO2 totale (Mt)]), 'CO2'[Tipo entità] = "Paese")

CO2 pro capite (t) =
DIVIDE([CO2 Paesi (Mt)] * 1000000, [Popolazione Paesi])

Var CO2 % =
VAR Ini = [CO2 inizio (Mt)]
VAR Fin = [CO2 fine (Mt)]
RETURN IF(NOT ISBLANK(Ini) && NOT ISBLANK(Fin), DIVIDE(Fin - Ini, Ini))

Posizione carbone =
IF(
    NOT ISBLANK([Quota carbone (classifica)]),
    RANKX(ALLSELECTED('CO2'[Paese]), [Quota carbone (classifica)], , DESC, DENSE)
)
```

### 3. Scelte metodologiche

- **Nessun valore nullo sostituito con zero**: uno zero significa "emissioni nulle", un vuoto significa "dato non disponibile".
- **Il pro capite si ricalcola** (non si somma): `CO2 / popolazione`.
- **Le cumulate si leggono a un anno preciso**, non si sommano tra anni.
- **Classifiche con misure dedicate** (`RANKX` + filtro sulla posizione) al posto del filtro Top N su misure, che dava risultati incoerenti.
- **Soglie di popolazione** (1 milione per il pro capite, 5 milioni per le classifiche del carbone) per evitare che i micro-stati falsino le graduatorie.
- Le **quote del mix** usano come denominatore la somma delle cinque fonti, così fanno esattamente 100%.

### 4. Controlli di qualità

- Nel 2022 la somma delle emissioni dei paesi (≈ 36.562 Mt) è inferiore del 2,6% alla riga *World* (≈ 37.528 Mt): la differenza corrisponde ad aviazione e navigazione internazionale, che non appartengono a nessun paese.
- Verifica dei valori noti (es. Italia 2024: ≈ 301,9 Mt e ≈ 5,09 t pro capite) a ogni passaggio.
- Confronto dei risultati di ogni grafico con un calcolo indipendente in Python/pandas.

## Limiti e cautele

- Il **PIL è disponibile solo fino al 2022**: l'analisi della pagina 2 si ferma lì.
- I **tagli dei paesi dell'ex blocco sovietico** (Romania, Slovacchia, Bielorussia...) riflettono in parte la recessione industriale degli anni '90, non solo politiche climatiche.
- Per molti paesi le colonne per fonte (soprattutto carbone e gas) sono vuote: probabilmente significa "nessun consumo registrato", ma i dati non lo specificano. Nelle quote restano vuote.
- Le cinque fonti coprono circa il 99% del totale: manca l'industria "altra", esclusa per semplicità.
- Le emissioni sono territoriali: non tengono conto di quelle "importate" tramite i beni consumati.

## Competenze dimostrate

Power Query (M) · modellazione dati · DAX (`CALCULATE`, `ALLSELECTED`, `REMOVEFILTERS`, `RANKX`, variabili) · data cleaning e controlli di coerenza · progettazione di dashboard · data storytelling

## Come riprodurre

1. Scarica `owid-co2-data.csv` dal [repository di Our World in Data](https://github.com/owid/co2-data).
2. Apri `Analisi CO2.pbix` con Power BI Desktop.
3. In **Trasforma dati → Origine** aggiorna il percorso del file CSV.
4. **Chiudi e applica** per ricaricare il modello

## Fonte e crediti

Dati: Our World in Data, `owid/co2-data` (vedi il codebook del repository per le fonti primarie e le definizioni delle variabili). Analisi e report: `Mauro Tuzzolino`.
