# Esempio analisi dati europei sulle emissioni dei gas serra con Python
<span style="color:red">**Nota**: alcuni aggiustamenti sono stati inseriti dopo la registrazione del webinar, per facilitare ulteriormente la lettura e l'analisi. Il link alla registrazione del webinar è [questo](https://www.youtube.com/watch?v=a07jSho77bg).</span>


## Importazione di `pandas`
Per cominciare, importiamo `pandas`.
`pandas` è una libreria di Python per la manipolazione e l'analisi dei dati.


```python
import pandas as pd
```

`pandas` include automaticamente al suo interno `numpy`, `scipy` e `matplotlib`.

## Lettura e pulizia dei dati
`pandas` integra delle funzioni già pronte di importazione delle principali tipologie di dato in circolazione:
- `pandas.read_csv()`
- `pandas.read_json()`
- `pandas.read_html()`
- `pandas.read_xml()`
- `pandas.read_csv()`
- `pandas.read_excel()`
- `pandas.read_sql()`

Nel nostro caso utilizziamo il dataset [Greenhouse gas emissions by source sector](https://ec.europa.eu/eurostat/databrowser/view/env_air_gge/default/table?lang=en) di Eurostat, che si presenta in formato CSV.


```python
# Sorgente file: https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/ENV_AIR_GGE/?format=SDMX-CSV&compressed=false&i
data = pd.read_csv('env_air_gge_linear.csv.gz') # Il file è compresso e salvato nella stessa cartella in cui è salvato questo notebook
data.shape
```




    (1967692, 10)



Abbiamo creato una struttura dati di `pandas` che si chiama **DataFrame**. Si tratta di una tabella di dati SQL: le **righe** sono identificate da un **indice**, le colonne corrispondono ai **campi**. Nel nostro caso ci sono quasi 2 milioni di righe e 10 colonne.

Adesso occupiamoci di capire come si presentino i dati ancora 'grezzi' nel nostro DataFrame.


```python
data.head() # Default: stampa le prime 5 righe
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DATAFLOW</th>
      <th>LAST UPDATE</th>
      <th>freq</th>
      <th>unit</th>
      <th>airpol</th>
      <th>src_crf</th>
      <th>geo</th>
      <th>TIME_PERIOD</th>
      <th>OBS_VALUE</th>
      <th>OBS_FLAG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ESTAT:ENV_AIR_GGE(1.0)</td>
      <td>10/06/22 11:00:00</td>
      <td>A</td>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1990</td>
      <td>0.04858</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ESTAT:ENV_AIR_GGE(1.0)</td>
      <td>10/06/22 11:00:00</td>
      <td>A</td>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1991</td>
      <td>0.04369</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ESTAT:ENV_AIR_GGE(1.0)</td>
      <td>10/06/22 11:00:00</td>
      <td>A</td>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1992</td>
      <td>0.04278</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ESTAT:ENV_AIR_GGE(1.0)</td>
      <td>10/06/22 11:00:00</td>
      <td>A</td>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1993</td>
      <td>0.04038</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ESTAT:ENV_AIR_GGE(1.0)</td>
      <td>10/06/22 11:00:00</td>
      <td>A</td>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1994</td>
      <td>0.03347</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Notiamo subito che non è così facile capire cosa significhino queste codifiche e, d'altro canto, è necessario che alcune informazioni siano opportunamente codificate. Quando i dati open sono distribuiti correttamente, sono disponibili dei **metadati** che li descrivono. Eurostat mette a disposizione una pagina per ogni dataset. Nel caso del nostro dataset: [env_air_gge](https://ec.europa.eu/eurostat/cache/metadata/en/env_air_gge_esms.htm).

Oltre ai valori osservati (**OBS_VALUE**), questo dataset ha 5 dimensioni principali:
- Inquinanti dell'aria (**AIRPOL**), ovvero i nomi dei gas serra.
- Entità geopolitiche (**GEO**).
- Il settore da cui provengono le emissioni (**SRC_CRF**), ad esempio quello energetico.
- L'anno di riferimento (**TIME**).
- L'unità di misura con cui leggere il dato (**UNIT**), nel nostro caso le migliaia o i milioni di tonnellate.

Andiamo ad analizzare le singole colonne sulla base dello spaccato qui sopra.


```python
data.columns
```




    Index(['DATAFLOW', 'LAST UPDATE', 'freq', 'unit', 'airpol', 'src_crf', 'geo',
           'TIME_PERIOD', 'OBS_VALUE', 'OBS_FLAG'],
          dtype='object')



Le colonne **DATAFLOW** e **LAST UPDATE** sembrano poco utili alla nostra analisi. Proviamo ad analizzarle più nel dettaglio per capire se è vero.


```python
data.DATAFLOW.describe()
```




    count                    1967692
    unique                         1
    top       ESTAT:ENV_AIR_GGE(1.0)
    freq                     1967692
    Name: DATAFLOW, dtype: object



La colonna **DATAFLOW** è un riferimento all'origine dei dati, che è l'Agenzia Europea dell'Ambiente. Contiene effettivamente sempre lo stesso valore (*unique = 1*) replicato 1967692 volte.


```python
data['LAST UPDATE'].describe()
# Nota: il nome della colonna ha uno spazio in mezzo, quindi sfruttiamo l'occasione per utilizzare una sintassi diversa per l'accesso
```




    count               1967692
    unique                    1
    top       10/06/22 11:00:00
    freq                1967692
    Name: LAST UPDATE, dtype: object



Per la colonna **LAST UPDATE**, il discorso è analogo: si tratta della data di aggiornamento del dato, che è la stessa per tutti i 1967692 record della tabella.

Eliminiamo queste due colonne.


```python
data.drop(columns=['DATAFLOW','LAST UPDATE'], inplace=True)
```

Si noti che senza inserire il parametro `inplace=True` (come comportamento standard la funzione ha `inplace=False`), `pandas` crea un riferimento al dataframe di partenza (invece di crearne una copia modificata). In generale, per ragioni di performance e - in certi contesti - di comodità, questo è il comportamento standard di Python: creare riferimenti anziché copie.

Su un blocco note come il nostro, modificare i dati inplace ha lo svantaggio che non si può ri-eseguire il codice soprastante con la garanzia che i risultati siano i medesimi.

Il nostro DataFrame ora si presenta così:


```python
data.describe(include='all')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>freq</th>
      <th>unit</th>
      <th>airpol</th>
      <th>src_crf</th>
      <th>geo</th>
      <th>TIME_PERIOD</th>
      <th>OBS_VALUE</th>
      <th>OBS_FLAG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1967692</td>
      <td>1967692</td>
      <td>1967692</td>
      <td>1967692</td>
      <td>1967692</td>
      <td>1.967692e+06</td>
      <td>1.697730e+06</td>
      <td>269962</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>1</td>
      <td>2</td>
      <td>11</td>
      <td>172</td>
      <td>36</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4</td>
    </tr>
    <tr>
      <th>top</th>
      <td>A</td>
      <td>MIO_T</td>
      <td>GHG</td>
      <td>TOTXMEMONIA</td>
      <td>HU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>z</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>1967692</td>
      <td>983846</td>
      <td>380140</td>
      <td>24820</td>
      <td>65630</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>223924</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.004706e+03</td>
      <td>7.070011e+03</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>std</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.071653e+00</td>
      <td>1.273772e+05</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>min</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.985000e+03</td>
      <td>-4.583370e+05</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.997000e+03</td>
      <td>0.000000e+00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.005000e+03</td>
      <td>4.349500e-02</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.013000e+03</td>
      <td>1.097602e+01</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>max</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.020000e+03</td>
      <td>5.827839e+06</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Notiamo che il metodo `describe()` consente di avere informazioni di natura statistica sul DataFrame (si notino le differenze tra le colonne contenenti stringhe e quelle contenenti numeri). Il parametro `include='all'` serve a non limitare la descrizione alle sole colonne numeriche, che sarebbe il comportamento di default, a meno di non averne.

Da qui notiamo che la colonna **freq** andrà eliminata (rappresenta la frequenza di diffusione del dato, che è annuale in tutti i casi) e quindi provvediamo.


```python
data.drop('freq', axis=1, inplace=True)
```

A scopo dimostrativo, abbiamo usato una maniera leggermente diversa di effettuare il drop: tramite il parametro `axis=1` stiamo indicando che l'etichetta `'freq'` è il nome di una colonna e non di una riga (le righe si indicano con `axis=0`, che è il valore di default). Questa maniera di riferirsi alle colonne e alle righe è propria di molte funzioni di `pandas`, quindi è bene impararla.

La situazione ora è la seguente:


```python
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>unit</th>
      <th>airpol</th>
      <th>src_crf</th>
      <th>geo</th>
      <th>TIME_PERIOD</th>
      <th>OBS_VALUE</th>
      <th>OBS_FLAG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1990</td>
      <td>0.04858</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1991</td>
      <td>0.04369</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1992</td>
      <td>0.04278</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1993</td>
      <td>0.04038</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MIO_T</td>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1994</td>
      <td>0.03347</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



### Dizionari per le codifiche
Vogliamo qui predisporre delle strutture dati che ci consentano di codificare il contenuto delle colonne: vorremmo per esempio poter associare al dato `'AT'` la stringa `'Austria'`. Per farlo, sfruttiamo una struttura dati di Python: il **dizionario** (che non è altro che un array associativo, in cui le chiavi non sono i numeri naturali in ordine crescente e progressivo - in Python questa struttura si chiama lista - ma possono essere sia numeri che stringhe).

Le codifiche sono disponibili in [un apposito repository](https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?sort=1&dir=dic%2Fen) di Eurostat in formato .dic, che dobbiamo capire come trattare. Ci avvaliamo della direttiva `with` che ci semplifica le operazioni di apertura e chiusura del file. Si noti che per identificare i blocchi di codice Python richiede di utilizzare l'indentazione (tramite spazi o tramite tabulazioni), quindi la direttiva `with` è seguita dal relativo blocco di codice indentato.


```python
# Sorgente file: https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?sort=1&file=dic%2Fen%2Fgeo.dic
with open('geo.dic', encoding='utf8') as f:
    line = f.readline()
line
```




    'EUR\tEurope\n'



Le linee del file sono le coppie chiave-valore del dizionario, separate da un carattere di tabulazione `\t` e chiuse da un carattere newline `\n`. Possiamo quindi costruire il nostro dizionario per i codici della colonna **geo** rimuovendo l'ultimo carattere `\n` e spezzando ciascuna linea all'occorrere del carattere `'\t'`.


```python
from itertools import islice

with open('geo.dic', encoding='utf8') as f:
    geo_dic = dict([line[:-1].split("\t") for line in f])
```

Costruiamo anche una piccola funzione per stampare solo una porzione del dizionario (le prime `n` righe):


```python
from itertools import islice

def dict_head(d, n = 5):
    return dict(islice(d.items(), 0, n))

dict_head(geo_dic)
```




    {'EUR': 'Europe',
     'EU': 'European Union (EU6-1958, EU9-1973, EU10-1981, EU12-1986, EU15-1995, EU25-2004, EU27-2007, EU28-2013, EU27-2020)',
     'EU_V': 'European Union (aggregate changing according to the context)',
     'EU27_2020_EFTA': 'European Union - 27 countries (from 2020) and European Free Trade Association (EFTA) countries',
     'EU27_2020_IS_K': 'European Union - 27 countries (from 2020) and Iceland under the Kyoto Protocol'}



Tramite il metodo `map()` dei Dataframe di `pandas`, potremo poi associare ai codici della colonna **geo** le stringhe con i nomi per esteso. In questo modo possiamo - per esempio - vedere quanti dati disponiamo nel nostro dataframe per ciascuna delle nazioni presenti, senza impazzire con i codici delle nazioni.


```python
data.geo.map(geo_dic).value_counts()
```




    Hungary                                                                           65630
    Slovenia                                                                          62152
    Romania                                                                           59550
    Poland                                                                            57698
    Ireland                                                                           57500
    Spain                                                                             56944
    Slovakia                                                                          56838
    European Union - 27 countries (from 2020)                                         56770
    Denmark                                                                           56458
    Lithuania                                                                         56072
    Finland                                                                           56054
    Bulgaria                                                                          55830
    Germany (until 1990 former territory of the FRG)                                  55388
    United Kingdom                                                                    55256
    Estonia                                                                           55070
    European Union - 28 countries (2013-2020) and Iceland under the Kyoto Protocol    54940
    European Union - 28 countries (2013-2020)                                         54940
    Portugal                                                                          54724
    France                                                                            54312
    Netherlands                                                                       54312
    Czechia                                                                           53984
    Belgium                                                                           53792
    Croatia                                                                           53692
    Italy                                                                             53586
    Latvia                                                                            53356
    Iceland                                                                           53048
    Greece                                                                            52648
    Austria                                                                           52582
    Switzerland                                                                       52442
    Türkiye                                                                           51884
    Luxembourg                                                                        51820
    Norway                                                                            51396
    Malta                                                                             50940
    Liechtenstein                                                                     49546
    Sweden                                                                            48856
    Cyprus                                                                            47682
    Name: geo, dtype: int64



Il metodo `value_counts()` consente di associare ad una Serie di `pandas` il numero di occorrenze di ciascun record. Sostanzialmente è un modo per costruire una tabella delle frequenze di ciascuna modalità.

Controlliamo se ci sono valori `NaN`, poiché il metodo `map()` associa `NaN` a tutti i codici non presenti tra le chiavi del dizionario. Asseriamo allora che il numero dei dati `NaN` sia zero, perché questo restituirà un errore solo in caso l'asserzione si verifichi essere falsa:


```python
assert data.geo.map(geo_dic).isna().sum() == 0
```

Il metodo `isna()` restituisce il valore `True` in corrispondenza di input che sono `NaN`, mentre il valore `False` in corrispondenza di input che non sono `NaN`.
Il metodo `sum()` restituisce la somma dei valori lungo le righe (o lungo le colonne se gli viene passato il parametro `axis=1`). Nel caso di valori booleani, restituisce la somma dei valori `True`. Si noti che un modo più elegante di fare questa operazione era di utilizzare `assert data.geo.map(geo_dic).notna().all()` (che restituisce `True` solo nel caso la serie di dati mappati contenga anche solo un valore `False`).

Rimuoviamo anche dal dizionario le chiavi non presenti nel dataset, in modo che il dizionario rappresenti tutte e sole le informazioni a disposizione. Lo facciamo sfruttando la cosiddetta *dict comprehension*, una delle caratteristiche distintive del Python, grazie alla quale possiamo innestare un ciclo `for` all'interno della definizione di un dizionario.


```python
geo_dic = {k:geo_dic[k] for k in data.geo.unique().tolist()}
dict_head(geo_dic)
```




    {'AT': 'Austria',
     'BE': 'Belgium',
     'BG': 'Bulgaria',
     'CH': 'Switzerland',
     'CY': 'Cyprus'}



In questo contesto è importante osservare che fino al 1989 la Germania era divisa e quindi il codice corrispondente alla Germania fa riferimento ai territori della Repubblica Federale di Germania, mentre dal 1990 fa riferimento alla Germania unificata. Il riferimento a territori diversi con una stessa sigla deve spingere alla massima attenzione nei confronti dei relativi dati!!

**Nota importante:** le righe corrispondenti ai codici `'EU27_2020'`, `'EU28'` e `'EU28_IS_K'` riportano dati aggregati di tutti gli stati dell'Unione Europea. Bisogna quindi ricordarsi di escluderle quando opportuno! Costruiamo un dizionario che non comprenda i codici di dati aggregati, per poterli escludere agevolmente all'occorrenza. Sfruttiamo la direttiva `del` per eliminare gli elementi da una copia del dizionario. Si noti la sintassi del ciclo `for` in Python, che si serve di una sequenza di valori da scorrere per assegnare i valori al relativo indice (questo assomiglia molto ad un metodo iterativo, nello stile dei linguaggi orientati agli oggetti).


```python
geo_dict_noagg = geo_dic.copy()
for k in ['EU27_2020','EU28','EU28_IS_K']:
    del geo_dict_noagg[k]

geo_dict_noagg
```




    {'AT': 'Austria',
     'BE': 'Belgium',
     'BG': 'Bulgaria',
     'CH': 'Switzerland',
     'CY': 'Cyprus',
     'CZ': 'Czechia',
     'DE': 'Germany (until 1990 former territory of the FRG)',
     'DK': 'Denmark',
     'EE': 'Estonia',
     'EL': 'Greece',
     'ES': 'Spain',
     'FI': 'Finland',
     'FR': 'France',
     'HR': 'Croatia',
     'HU': 'Hungary',
     'IE': 'Ireland',
     'IS': 'Iceland',
     'IT': 'Italy',
     'LI': 'Liechtenstein',
     'LT': 'Lithuania',
     'LU': 'Luxembourg',
     'LV': 'Latvia',
     'MT': 'Malta',
     'NL': 'Netherlands',
     'NO': 'Norway',
     'PL': 'Poland',
     'PT': 'Portugal',
     'RO': 'Romania',
     'SE': 'Sweden',
     'SI': 'Slovenia',
     'SK': 'Slovakia',
     'TR': 'Türkiye',
     'UK': 'United Kingdom'}



Si noti l'utilizzo del metodo `copy()` per creare una copia effettiva del dizionario, da modificare successivamente. Abbiamo infatti bisogno di effettuare questa operazione perché altrimenti l'operatore di assegnazione non effettua una vera copia ma un semplice riferimento e dunque l'eliminazione e/o la modifica di componenti di un dizionario influisce anche sul contenuto dell'altro (e viceversa).

Ora possiamo fare lo stesso discorso per i codici delle altre colonne. Ecco quella per i gas serra:


```python
# Sorgente file: https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?sort=1&file=dic%2Fen%2Fairpol.dic
with open('airpol.dic', encoding='utf8') as f:
    airpol_dic = dict([line[:-1].split("\t") for line in f])

assert data.airpol.map(airpol_dic).notna().all()
airpol_dic = {k:airpol_dic[k] for k in data.airpol.unique().tolist()}
data.airpol.value_counts() # I nomi per esteso sono lunghi, quindi qui stampiamo i codici
```




    GHG                 380140
    CH4                 282590
    CH4_CO2E            282590
    CO2                 265924
    N2O                 262258
    N2O_CO2E            262258
    HFC_CO2E             52788
    PFC_CO2E             49038
    SF6_CO2E             48334
    HFC_PFC_NSP_CO2E     41252
    NF3_CO2E             40520
    Name: airpol, dtype: int64




```python
airpol_dic
```




    {'CH4': 'Methane',
     'CH4_CO2E': 'Methane (CO2 equivalent)',
     'CO2': 'Carbon dioxide',
     'GHG': 'Greenhouse gases (CO2, N2O in CO2 equivalent, CH4 in CO2 equivalent, HFC in CO2 equivalent, PFC in CO2 equivalent, SF6 in CO2 equivalent, NF3 in CO2 equivalent)',
     'HFC_CO2E': 'Hydrofluorocarbones (CO2 equivalent)',
     'HFC_PFC_NSP_CO2E': 'Hydrofluorocarbones and perfluorocarbones - not specified mix (CO2 equivalent)',
     'N2O': 'Nitrous oxide',
     'N2O_CO2E': 'Nitrous oxide (CO2 equivalent)',
     'NF3_CO2E': 'Nitrogen trifluoride (CO2 equivalent)',
     'PFC_CO2E': 'Perfluorocarbones (CO2 equivalent)',
     'SF6_CO2E': 'Sulphur hexafluoride (CO2 equivalent)'}



**Nota importante:** le righe corrispondenti ai codici che terminano con *_CO2E* quantificano l'effetto del corrispondente gas serra sul riscaldamento globale, convertito in ammontare di CO<sub>2</sub>. Inoltre le righe corrispondenti al codice `'GHG'` sono relative ai valori aggregati di tutti i gas serra (convertiti in ammontare di CO<sub>2</sub>).
Anche in questa situazione bisognerà fare attenzione in fase di aggregazione dei dati. Costruiamo il dizionario con i valori non aggregati convertiti in ammontare di CO<sub>2</sub>:


```python
airpol_dic_noagg_co2e = airpol_dic.copy()
for k in ['CH4','GHG','HFC_PFC_NSP_CO2E','N2O']:
    del airpol_dic_noagg_co2e[k]
airpol_dic_noagg_co2e
```




    {'CH4_CO2E': 'Methane (CO2 equivalent)',
     'CO2': 'Carbon dioxide',
     'HFC_CO2E': 'Hydrofluorocarbones (CO2 equivalent)',
     'N2O_CO2E': 'Nitrous oxide (CO2 equivalent)',
     'NF3_CO2E': 'Nitrogen trifluoride (CO2 equivalent)',
     'PFC_CO2E': 'Perfluorocarbones (CO2 equivalent)',
     'SF6_CO2E': 'Sulphur hexafluoride (CO2 equivalent)'}



Costruiamo anche il dizionario con i valori non aggregati e non convertiti in ammontare di CO<sub>2</sub>:


```python
airpol_dic_noagg = {k:airpol_dic[k] for k in ['CH4','CO2','N2O']}
airpol_dic_noagg
```




    {'CH4': 'Methane', 'CO2': 'Carbon dioxide', 'N2O': 'Nitrous oxide'}



Infine, lavoriamo sul dizionario dei settori di provenienza delle emissioni:


```python
# Sorgente file: https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?sort=1&file=dic%2Fen%2Fsrc_crf.dic
with open('src_crf.dic', encoding='utf8') as f:
    src_crf_dic = dict([line[:-1].split("\t") for line in f])

assert data.src_crf.map(src_crf_dic).notna().all()
src_crf_dic = {k:src_crf_dic[k] for k in data.src_crf.unique().tolist()}

data.src_crf.value_counts()
```




    TOTXMEMONIA      24820
    TOTXMEMONIT      24820
    TOTX4_MEMONIT    24788
    TOTX4_MEMONIA    24788
    CRF2             24600
                     ...  
    CRF3I             4380
    CRF_INDCO2        3932
    CRF5F1            3752
    CRF5F2            3752
    CRF5F3            3628
    Name: src_crf, Length: 172, dtype: int64




```python
dict_head(src_crf_dic)
```




    {'CRF1': 'Energy',
     'CRF1A': 'Fuel combustion - sectoral approach',
     'CRF1A1': 'Fuel combustion in energy industries',
     'CRF1A1A': 'Fuel combustion in public electricity and heat production',
     'CRF1A1B': 'Fuel combustion in petroleum refining'}



**Nota importante:** le righe corrispondenti ai codici che cominciano con con *TOT* sono dei "totali", ovvero sono somme parziali dei dati relativi ad altre righe (si vedano anche [i metadati](https://ec.europa.eu/eurostat/cache/metadata/en/env_air_gge_esms.htm#stat_pres1655190747652)). Costruiamo anche qui il corrispondente dizionario con i valori non aggregati:


```python
src_crf_dic_noagg = src_crf_dic.copy()
for k in ['TOTXMEMO', 'TOTX4_MEMO', 'TOTX4_MEMONIA','TOTXMEMONIA','TOTX4_MEMONIT','TOTXMEMONIT']:
    del src_crf_dic_noagg[k]
dict_head(src_crf_dic_noagg)
```




    {'CRF1': 'Energy',
     'CRF1A': 'Fuel combustion - sectoral approach',
     'CRF1A1': 'Fuel combustion in energy industries',
     'CRF1A1A': 'Fuel combustion in public electricity and heat production',
     'CRF1A1B': 'Fuel combustion in petroleum refining'}



### Conversione delle unità di misura
La colonna dell'unità di misura può essere eliminata a patto di convertire i valori osservati tutti nella stessa unità di misura. Nei dati in esame ci sono due codici diversi:


```python
data.unit.value_counts(dropna=False)
```




    MIO_T    983846
    THS_T    983846
    Name: unit, dtype: int64



Il codice `MIO_T` indica che il valore di **OBS_VALUE** è in milioni di tonnellate, mentre il codice `THS_T` indica che il valore di **OBS_VALUE** è in migliaia di tonnellate. Possiamo dunque associare a ciascun codice il corrispettivo valore e moltiplicare poi il risultato per la colonna dei valori osservati (la moltiplicazione avviene valore per valore).


```python
data.OBS_VALUE = data.unit.map({'MIO_T': 1e6, 'THS_T': 1e3}) * data.OBS_VALUE
```

Si noti che abbiamo deciso di utilizzare la notazione scientifica per la rappresentazione dei valori.

A questo punto, possiamo eliminare la colonna delle unità di misura:


```python
data.drop(columns='unit', inplace=True)
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>airpol</th>
      <th>src_crf</th>
      <th>geo</th>
      <th>TIME_PERIOD</th>
      <th>OBS_VALUE</th>
      <th>OBS_FLAG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1990</td>
      <td>48580.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1991</td>
      <td>43690.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1992</td>
      <td>42780.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1993</td>
      <td>40380.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1994</td>
      <td>33470.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



## Colonna **OBS_FLAG**
Le informazioni contenute nella colonna **OBS_FLAG** sono strane e vanno spiegate guardando [i metadati](https://ec.europa.eu/eurostat/cache/metadata/en/env_air_gge_esms.htm#stat_pres1655190747652).


```python
data.OBS_FLAG.value_counts(dropna=False)
```




    NaN    1697730
    z       223924
    d        45444
    c          426
    cd         168
    Name: OBS_FLAG, dtype: int64



La sostanza è che la colonna segnala le ragioni per cui alcuni dati sono mancanti. Il fatto che ci sia una corrispondenza dei valori diversi da `NaN` della colonna **OBS_FLAG** con i valori pari a `NaN` della colonna **OBS_VALUE** è dimostrato dalle seguenti asserzioni, che dimostrano la corrispondenza biunivoca tra valori `NaN` in **OBS_VALUE** e valori diversi da `NaN` in **OBS_FLAG**:


```python
assert data[data.OBS_FLAG.notna()].OBS_VALUE.isna().all()
assert data[data.OBS_VALUE.isna()].OBS_FLAG.notna().all()
```

Possiamo allora eliminare la colonna **OBS_FLAG**:


```python
data.drop(columns='OBS_FLAG', inplace=True)
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>airpol</th>
      <th>src_crf</th>
      <th>geo</th>
      <th>TIME_PERIOD</th>
      <th>OBS_VALUE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1990</td>
      <td>48580.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1991</td>
      <td>43690.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1992</td>
      <td>42780.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1993</td>
      <td>40380.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1994</td>
      <td>33470.0</td>
    </tr>
  </tbody>
</table>
</div>



### Nuovi nomi alle colonne


```python
data.rename(columns={
    'airpol': 'Inquinante',
    'src_crf': 'Fonte',
    'geo': 'Entità geopolitica',
    'TIME_PERIOD': 'Anno',
    'OBS_VALUE': 'Tonnellate emesse'},
            inplace=True)
data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Inquinante</th>
      <th>Fonte</th>
      <th>Entità geopolitica</th>
      <th>Anno</th>
      <th>Tonnellate emesse</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1990</td>
      <td>48580.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1991</td>
      <td>43690.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1992</td>
      <td>42780.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1993</td>
      <td>40380.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CH4</td>
      <td>CRF1</td>
      <td>AT</td>
      <td>1994</td>
      <td>33470.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1967687</th>
      <td>SF6_CO2E</td>
      <td>TOTXMEMONIT</td>
      <td>UK</td>
      <td>2016</td>
      <td>432010.0</td>
    </tr>
    <tr>
      <th>1967688</th>
      <td>SF6_CO2E</td>
      <td>TOTXMEMONIT</td>
      <td>UK</td>
      <td>2017</td>
      <td>437360.0</td>
    </tr>
    <tr>
      <th>1967689</th>
      <td>SF6_CO2E</td>
      <td>TOTXMEMONIT</td>
      <td>UK</td>
      <td>2018</td>
      <td>535190.0</td>
    </tr>
    <tr>
      <th>1967690</th>
      <td>SF6_CO2E</td>
      <td>TOTXMEMONIT</td>
      <td>UK</td>
      <td>2019</td>
      <td>474500.0</td>
    </tr>
    <tr>
      <th>1967691</th>
      <td>SF6_CO2E</td>
      <td>TOTXMEMONIT</td>
      <td>UK</td>
      <td>2020</td>
      <td>406940.0</td>
    </tr>
  </tbody>
</table>
<p>1967692 rows × 5 columns</p>
</div>



## Grafici
Cerchiamo ora di costruire qualche rappresentazione grafica interessante.

### Confronto delle emissioni dei diversi paesi
Consideriamo le emissioni totali, da cui si escludono quelle dovute al consumo del suolo (perché difficili da stimare) e dei trasporti internazionali. Vogliamo costruire il grafico di confronto delle emissioni di alcuni paesi europei nel tempo.

Iniziamo isolando quindi in un'apposita *vista* del nostro DataFrame solo le righe per le quali la colonna **Fonte** è pari a `TOTX4_MEMO`. Di queste, siamo interessati alle colonne **Anno**, **Entità geopolitica** e **Tonnellate emesse**. In SQL questo corrisponderebbe sostanzialmente a una query di selezione (SELECT).


```python
total_emissions = data[data['Fonte'] == 'TOTX4_MEMO'][['Anno','Entità geopolitica','Tonnellate emesse']]
total_emissions.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Anno</th>
      <th>Entità geopolitica</th>
      <th>Tonnellate emesse</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>134527</th>
      <td>1990</td>
      <td>AT</td>
      <td>404430.0</td>
    </tr>
    <tr>
      <th>134528</th>
      <td>1991</td>
      <td>AT</td>
      <td>399970.0</td>
    </tr>
    <tr>
      <th>134529</th>
      <td>1992</td>
      <td>AT</td>
      <td>389000.0</td>
    </tr>
    <tr>
      <th>134530</th>
      <td>1993</td>
      <td>AT</td>
      <td>390520.0</td>
    </tr>
    <tr>
      <th>134531</th>
      <td>1994</td>
      <td>AT</td>
      <td>378650.0</td>
    </tr>
  </tbody>
</table>
</div>



Adesso sommiamo le tonnellate emesse se corrispondono a righe in cui coincidono **Anno** ed **Entità geopolitica**. (Essendo i dati pubblicati con frequenza annuale per ciascuna entità geopolitica, questa somma dovrebbe essere sostanzialmente inutile.)


```python
total_emissions = total_emissions.groupby(['Anno','Entità geopolitica'])['Tonnellate emesse'].sum()
total_emissions
```




    Anno  Entità geopolitica
    1985  HU                    4.477746e+08
    1986  HU                    4.397678e+08
          SI                    8.200912e+07
    1987  HU                    4.422936e+08
          SI                    7.904972e+07
                                    ...     
    2020  SE                    1.854988e+08
          SI                    6.356248e+07
          SK                    1.484683e+08
          TR                    2.100980e+09
          UK                    1.612189e+09
    Name: Tonnellate emesse, Length: 1126, dtype: float64



Notiamo che `groupby()` indicizza i dati in base alle due colonne specificate. Significa che fornendo una coppia di anno ed entità geopolitica (ovvero un indice a due valori, quindi bidimensionale) ci viene restituita la corrispondente somma.

Adesso passiamo da questa indicizzazione ad una tabella a doppia entrata tramite il metodo `unstack()`, che trasforma l'indice bidimensionale:


```python
total_emissions = total_emissions.unstack()
total_emissions
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Entità geopolitica</th>
      <th>AT</th>
      <th>BE</th>
      <th>BG</th>
      <th>CH</th>
      <th>CY</th>
      <th>CZ</th>
      <th>DE</th>
      <th>DK</th>
      <th>EE</th>
      <th>EL</th>
      <th>...</th>
      <th>NL</th>
      <th>NO</th>
      <th>PL</th>
      <th>PT</th>
      <th>RO</th>
      <th>SE</th>
      <th>SI</th>
      <th>SK</th>
      <th>TR</th>
      <th>UK</th>
    </tr>
    <tr>
      <th>Anno</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1985</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1986</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>82009120.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1987</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>79049720.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1988</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>453739880.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.323003e+09</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>76825100.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1989</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>444016260.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.248413e+09</td>
      <td>NaN</td>
      <td>1.233597e+09</td>
      <td>NaN</td>
      <td>75905720.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1990</th>
      <td>314532040.0</td>
      <td>583735980.0</td>
      <td>394530000.0</td>
      <td>216406900.0</td>
      <td>22387520.0</td>
      <td>797324080.0</td>
      <td>4.977551e+09</td>
      <td>285175420.0</td>
      <td>160862700.0</td>
      <td>414745220.0</td>
      <td>...</td>
      <td>884719000.0</td>
      <td>206253620.0</td>
      <td>1.909110e+09</td>
      <td>234565500.0</td>
      <td>1.003092e+09</td>
      <td>286398120.0</td>
      <td>74602920.0</td>
      <td>294463120.0</td>
      <td>8.824458e+08</td>
      <td>3.184333e+09</td>
    </tr>
    <tr>
      <th>1991</th>
      <td>329211980.0</td>
      <td>594611320.0</td>
      <td>321806440.0</td>
      <td>223978380.0</td>
      <td>24391680.0</td>
      <td>723436740.0</td>
      <td>4.793280e+09</td>
      <td>328029100.0</td>
      <td>149337820.0</td>
      <td>414954580.0</td>
      <td>...</td>
      <td>915267620.0</td>
      <td>196614540.0</td>
      <td>1.860433e+09</td>
      <td>242257060.0</td>
      <td>8.271845e+08</td>
      <td>287384420.0</td>
      <td>69186560.0</td>
      <td>256598860.0</td>
      <td>9.120551e+08</td>
      <td>3.219717e+09</td>
    </tr>
    <tr>
      <th>1992</th>
      <td>302663500.0</td>
      <td>593005600.0</td>
      <td>299378840.0</td>
      <td>222934660.0</td>
      <td>26176220.0</td>
      <td>701312520.0</td>
      <td>4.594973e+09</td>
      <td>303962360.0</td>
      <td>108682500.0</td>
      <td>419790840.0</td>
      <td>...</td>
      <td>917635620.0</td>
      <td>189988080.0</td>
      <td>1.809200e+09</td>
      <td>257725460.0</td>
      <td>7.655055e+08</td>
      <td>285492480.0</td>
      <td>69146380.0</td>
      <td>233704760.0</td>
      <td>9.368835e+08</td>
      <td>3.136924e+09</td>
    </tr>
    <tr>
      <th>1993</th>
      <td>304035480.0</td>
      <td>588426240.0</td>
      <td>296951520.0</td>
      <td>212622760.0</td>
      <td>27333420.0</td>
      <td>672231380.0</td>
      <td>4.559005e+09</td>
      <td>313182680.0</td>
      <td>86870040.0</td>
      <td>418220880.0</td>
      <td>...</td>
      <td>920458760.0</td>
      <td>197857720.0</td>
      <td>1.808664e+09</td>
      <td>251741500.0</td>
      <td>7.249003e+08</td>
      <td>286679440.0</td>
      <td>70127080.0</td>
      <td>220033220.0</td>
      <td>9.661626e+08</td>
      <td>3.059195e+09</td>
    </tr>
    <tr>
      <th>1994</th>
      <td>304943840.0</td>
      <td>606845980.0</td>
      <td>281287360.0</td>
      <td>208526680.0</td>
      <td>28328520.0</td>
      <td>637097080.0</td>
      <td>4.487640e+09</td>
      <td>328732920.0</td>
      <td>88836600.0</td>
      <td>429112960.0</td>
      <td>...</td>
      <td>923679960.0</td>
      <td>205482020.0</td>
      <td>1.786174e+09</td>
      <td>255617760.0</td>
      <td>7.145512e+08</td>
      <td>296698040.0</td>
      <td>71802760.0</td>
      <td>210127580.0</td>
      <td>9.420229e+08</td>
      <td>3.015148e+09</td>
    </tr>
    <tr>
      <th>1995</th>
      <td>317913400.0</td>
      <td>615349960.0</td>
      <td>287452400.0</td>
      <td>212174520.0</td>
      <td>28080360.0</td>
      <td>632582080.0</td>
      <td>4.469940e+09</td>
      <td>316813800.0</td>
      <td>80429540.0</td>
      <td>438561800.0</td>
      <td>...</td>
      <td>923597420.0</td>
      <td>207064180.0</td>
      <td>1.794191e+09</td>
      <td>274486220.0</td>
      <td>7.431445e+08</td>
      <td>294079760.0</td>
      <td>74927160.0</td>
      <td>212160780.0</td>
      <td>9.978122e+08</td>
      <td>2.989909e+09</td>
    </tr>
    <tr>
      <th>1996</th>
      <td>330686400.0</td>
      <td>630189720.0</td>
      <td>289184000.0</td>
      <td>214911240.0</td>
      <td>29462280.0</td>
      <td>644883540.0</td>
      <td>4.541977e+09</td>
      <td>369136380.0</td>
      <td>84242240.0</td>
      <td>451113640.0</td>
      <td>...</td>
      <td>966543320.0</td>
      <td>218584780.0</td>
      <td>1.849374e+09</td>
      <td>265316000.0</td>
      <td>7.547012e+08</td>
      <td>309879600.0</td>
      <td>77341580.0</td>
      <td>211532420.0</td>
      <td>1.074249e+09</td>
      <td>3.072903e+09</td>
    </tr>
    <tr>
      <th>1997</th>
      <td>329268140.0</td>
      <td>596317060.0</td>
      <td>276389860.0</td>
      <td>210207760.0</td>
      <td>29814860.0</td>
      <td>626953180.0</td>
      <td>4.401809e+09</td>
      <td>331064260.0</td>
      <td>83113140.0</td>
      <td>470704880.0</td>
      <td>...</td>
      <td>934090440.0</td>
      <td>218673240.0</td>
      <td>1.809039e+09</td>
      <td>278246580.0</td>
      <td>7.264244e+08</td>
      <td>289862900.0</td>
      <td>79498380.0</td>
      <td>210911000.0</td>
      <td>1.119072e+09</td>
      <td>2.976113e+09</td>
    </tr>
    <tr>
      <th>1998</th>
      <td>326528140.0</td>
      <td>616993560.0</td>
      <td>260117420.0</td>
      <td>216392840.0</td>
      <td>31130120.0</td>
      <td>602404420.0</td>
      <td>4.300395e+09</td>
      <td>314727300.0</td>
      <td>76109520.0</td>
      <td>493041560.0</td>
      <td>...</td>
      <td>935610320.0</td>
      <td>218766280.0</td>
      <td>1.687545e+09</td>
      <td>297720400.0</td>
      <td>6.579335e+08</td>
      <td>291650860.0</td>
      <td>77868700.0</td>
      <td>208020320.0</td>
      <td>1.125315e+09</td>
      <td>2.970408e+09</td>
    </tr>
    <tr>
      <th>1999</th>
      <td>320393320.0</td>
      <td>592050940.0</td>
      <td>232020560.0</td>
      <td>215700580.0</td>
      <td>32205960.0</td>
      <td>563186680.0</td>
      <td>4.164387e+09</td>
      <td>304338740.0</td>
      <td>71507460.0</td>
      <td>493565200.0</td>
      <td>...</td>
      <td>881509160.0</td>
      <td>222851840.0</td>
      <td>1.641812e+09</td>
      <td>330139280.0</td>
      <td>5.857765e+08</td>
      <td>279332760.0</td>
      <td>75411280.0</td>
      <td>202794780.0</td>
      <td>1.115193e+09</td>
      <td>2.850212e+09</td>
    </tr>
    <tr>
      <th>2000</th>
      <td>321027360.0</td>
      <td>596402500.0</td>
      <td>228495920.0</td>
      <td>212885080.0</td>
      <td>33289940.0</td>
      <td>604399640.0</td>
      <td>4.154974e+09</td>
      <td>286557260.0</td>
      <td>70023160.0</td>
      <td>507116440.0</td>
      <td>...</td>
      <td>874186760.0</td>
      <td>220209220.0</td>
      <td>1.591080e+09</td>
      <td>327254400.0</td>
      <td>5.585471e+08</td>
      <td>273933320.0</td>
      <td>74532300.0</td>
      <td>195482620.0</td>
      <td>1.199699e+09</td>
      <td>2.849954e+09</td>
    </tr>
    <tr>
      <th>2001</th>
      <td>336629720.0</td>
      <td>590131460.0</td>
      <td>240342180.0</td>
      <td>218987660.0</td>
      <td>33072700.0</td>
      <td>604620340.0</td>
      <td>4.219000e+09</td>
      <td>292838080.0</td>
      <td>71746160.0</td>
      <td>511235620.0</td>
      <td>...</td>
      <td>876685820.0</td>
      <td>225289040.0</td>
      <td>1.585607e+09</td>
      <td>325470500.0</td>
      <td>5.731089e+08</td>
      <td>277202380.0</td>
      <td>79623260.0</td>
      <td>204790180.0</td>
      <td>1.122802e+09</td>
      <td>2.862360e+09</td>
    </tr>
    <tr>
      <th>2002</th>
      <td>343650100.0</td>
      <td>590430080.0</td>
      <td>229991140.0</td>
      <td>212969340.0</td>
      <td>33986880.0</td>
      <td>590047620.0</td>
      <td>4.133368e+09</td>
      <td>290676240.0</td>
      <td>69355860.0</td>
      <td>511184980.0</td>
      <td>...</td>
      <td>867721960.0</td>
      <td>220548580.0</td>
      <td>1.546575e+09</td>
      <td>343316980.0</td>
      <td>5.794295e+08</td>
      <td>280042780.0</td>
      <td>80716860.0</td>
      <td>199225720.0</td>
      <td>1.146222e+09</td>
      <td>2.776159e+09</td>
    </tr>
    <tr>
      <th>2003</th>
      <td>365494860.0</td>
      <td>591573340.0</td>
      <td>249447740.0</td>
      <td>217243100.0</td>
      <td>35566500.0</td>
      <td>602570160.0</td>
      <td>4.120911e+09</td>
      <td>310587240.0</td>
      <td>77153540.0</td>
      <td>526159880.0</td>
      <td>...</td>
      <td>870179800.0</td>
      <td>222959080.0</td>
      <td>1.601823e+09</td>
      <td>323044960.0</td>
      <td>6.023857e+08</td>
      <td>281462500.0</td>
      <td>79402580.0</td>
      <td>199723940.0</td>
      <td>1.223012e+09</td>
      <td>2.802231e+09</td>
    </tr>
    <tr>
      <th>2004</th>
      <td>364076980.0</td>
      <td>594912540.0</td>
      <td>246327940.0</td>
      <td>219705800.0</td>
      <td>36473920.0</td>
      <td>606625120.0</td>
      <td>4.053097e+09</td>
      <td>286516700.0</td>
      <td>77668080.0</td>
      <td>528738340.0</td>
      <td>...</td>
      <td>876181720.0</td>
      <td>224554200.0</td>
      <td>1.624429e+09</td>
      <td>336230000.0</td>
      <td>5.952283e+08</td>
      <td>279166980.0</td>
      <td>81010340.0</td>
      <td>202694680.0</td>
      <td>1.261544e+09</td>
      <td>2.788136e+09</td>
    </tr>
    <tr>
      <th>2005</th>
      <td>368751220.0</td>
      <td>582943680.0</td>
      <td>248862980.0</td>
      <td>222263020.0</td>
      <td>36984680.0</td>
      <td>598034320.0</td>
      <td>3.952586e+09</td>
      <td>268814520.0</td>
      <td>76733620.0</td>
      <td>546609880.0</td>
      <td>...</td>
      <td>853218300.0</td>
      <td>220267300.0</td>
      <td>1.625203e+09</td>
      <td>343996960.0</td>
      <td>5.903017e+08</td>
      <td>267833680.0</td>
      <td>82051280.0</td>
      <td>202615840.0</td>
      <td>1.351728e+09</td>
      <td>2.756188e+09</td>
    </tr>
    <tr>
      <th>2006</th>
      <td>359051560.0</td>
      <td>571581460.0</td>
      <td>251688560.0</td>
      <td>221019080.0</td>
      <td>38028660.0</td>
      <td>603327540.0</td>
      <td>3.980395e+09</td>
      <td>299621880.0</td>
      <td>74110720.0</td>
      <td>530934080.0</td>
      <td>...</td>
      <td>832936620.0</td>
      <td>220207860.0</td>
      <td>1.685927e+09</td>
      <td>324680260.0</td>
      <td>5.962437e+08</td>
      <td>266279160.0</td>
      <td>82847360.0</td>
      <td>201862020.0</td>
      <td>1.435758e+09</td>
      <td>2.726596e+09</td>
    </tr>
    <tr>
      <th>2007</th>
      <td>347979620.0</td>
      <td>557009060.0</td>
      <td>267597920.0</td>
      <td>213350540.0</td>
      <td>39357940.0</td>
      <td>610113300.0</td>
      <td>3.877457e+09</td>
      <td>280937580.0</td>
      <td>88527740.0</td>
      <td>541545080.0</td>
      <td>...</td>
      <td>826519000.0</td>
      <td>226897860.0</td>
      <td>1.685208e+09</td>
      <td>315611720.0</td>
      <td>6.100855e+08</td>
      <td>261787840.0</td>
      <td>83585220.0</td>
      <td>194872620.0</td>
      <td>1.569141e+09</td>
      <td>2.678406e+09</td>
    </tr>
    <tr>
      <th>2008</th>
      <td>345639840.0</td>
      <td>556663860.0</td>
      <td>262583180.0</td>
      <td>219003060.0</td>
      <td>40241760.0</td>
      <td>589931660.0</td>
      <td>3.880940e+09</td>
      <td>266607580.0</td>
      <td>80304780.0</td>
      <td>528120120.0</td>
      <td>...</td>
      <td>824172240.0</td>
      <td>220987340.0</td>
      <td>1.659020e+09</td>
      <td>305926280.0</td>
      <td>5.943731e+08</td>
      <td>252280520.0</td>
      <td>86479120.0</td>
      <td>197002000.0</td>
      <td>1.555649e+09</td>
      <td>2.594790e+09</td>
    </tr>
    <tr>
      <th>2009</th>
      <td>318931420.0</td>
      <td>506117560.0</td>
      <td>227109020.0</td>
      <td>213149000.0</td>
      <td>39287140.0</td>
      <td>552019040.0</td>
      <td>3.615976e+09</td>
      <td>255402520.0</td>
      <td>66388140.0</td>
      <td>499253760.0</td>
      <td>...</td>
      <td>801796420.0</td>
      <td>211151300.0</td>
      <td>1.586888e+09</td>
      <td>293411240.0</td>
      <td>5.105570e+08</td>
      <td>235178020.0</td>
      <td>77811920.0</td>
      <td>179823020.0</td>
      <td>1.584677e+09</td>
      <td>2.369851e+09</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>337184260.0</td>
      <td>535285700.0</td>
      <td>237853580.0</td>
      <td>219690420.0</td>
      <td>37986740.0</td>
      <td>563286780.0</td>
      <td>3.747932e+09</td>
      <td>255917540.0</td>
      <td>84828540.0</td>
      <td>474967320.0</td>
      <td>...</td>
      <td>849769200.0</td>
      <td>220217340.0</td>
      <td>1.655776e+09</td>
      <td>276532580.0</td>
      <td>4.936142e+08</td>
      <td>259303960.0</td>
      <td>78753600.0</td>
      <td>183023380.0</td>
      <td>1.599016e+09</td>
      <td>2.426794e+09</td>
    </tr>
    <tr>
      <th>2011</th>
      <td>328597320.0</td>
      <td>493210560.0</td>
      <td>259088960.0</td>
      <td>203203760.0</td>
      <td>36727520.0</td>
      <td>557483040.0</td>
      <td>3.649746e+09</td>
      <td>235015020.0</td>
      <td>84638260.0</td>
      <td>463025980.0</td>
      <td>...</td>
      <td>792360940.0</td>
      <td>216416140.0</td>
      <td>1.651353e+09</td>
      <td>270505900.0</td>
      <td>5.206131e+08</td>
      <td>241921380.0</td>
      <td>78449240.0</td>
      <td>179124740.0</td>
      <td>1.718492e+09</td>
      <td>2.245951e+09</td>
    </tr>
    <tr>
      <th>2012</th>
      <td>317797480.0</td>
      <td>482118620.0</td>
      <td>239202480.0</td>
      <td>208575820.0</td>
      <td>34615880.0</td>
      <td>541549680.0</td>
      <td>3.672420e+09</td>
      <td>216698460.0</td>
      <td>80197240.0</td>
      <td>449980800.0</td>
      <td>...</td>
      <td>775373040.0</td>
      <td>214096400.0</td>
      <td>1.621020e+09</td>
      <td>262808220.0</td>
      <td>5.122601e+08</td>
      <td>230454420.0</td>
      <td>75947820.0</td>
      <td>169448760.0</td>
      <td>1.796845e+09</td>
      <td>2.310882e+09</td>
    </tr>
    <tr>
      <th>2013</th>
      <td>319637040.0</td>
      <td>482492320.0</td>
      <td>222925540.0</td>
      <td>212083220.0</td>
      <td>31770900.0</td>
      <td>520050360.0</td>
      <td>3.740716e+09</td>
      <td>223654100.0</td>
      <td>87827480.0</td>
      <td>411518380.0</td>
      <td>...</td>
      <td>776690800.0</td>
      <td>215125160.0</td>
      <td>1.604467e+09</td>
      <td>255246140.0</td>
      <td>4.663005e+08</td>
      <td>223785040.0</td>
      <td>73169340.0</td>
      <td>168157300.0</td>
      <td>1.764936e+09</td>
      <td>2.256268e+09</td>
    </tr>
    <tr>
      <th>2014</th>
      <td>305478860.0</td>
      <td>459721180.0</td>
      <td>236064580.0</td>
      <td>196305420.0</td>
      <td>33261040.0</td>
      <td>511326180.0</td>
      <td>3.582539e+09</td>
      <td>206823180.0</td>
      <td>84518660.0</td>
      <td>397883200.0</td>
      <td>...</td>
      <td>745404040.0</td>
      <td>216596720.0</td>
      <td>1.552834e+09</td>
      <td>254924420.0</td>
      <td>4.632149e+08</td>
      <td>216177820.0</td>
      <td>66610000.0</td>
      <td>160333260.0</td>
      <td>1.842358e+09</td>
      <td>2.095668e+09</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>314479140.0</td>
      <td>476469280.0</td>
      <td>249638880.0</td>
      <td>194328720.0</td>
      <td>33469620.0</td>
      <td>516935880.0</td>
      <td>3.596477e+09</td>
      <td>196374180.0</td>
      <td>72240240.0</td>
      <td>382537940.0</td>
      <td>...</td>
      <td>773910080.0</td>
      <td>218387320.0</td>
      <td>1.560626e+09</td>
      <td>271562540.0</td>
      <td>4.613019e+08</td>
      <td>216816620.0</td>
      <td>67340240.0</td>
      <td>163151560.0</td>
      <td>1.902319e+09</td>
      <td>2.025169e+09</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>318400340.0</td>
      <td>470313440.0</td>
      <td>241115520.0</td>
      <td>195435320.0</td>
      <td>35293040.0</td>
      <td>522075220.0</td>
      <td>3.610329e+09</td>
      <td>204422060.0</td>
      <td>78978740.0</td>
      <td>368047580.0</td>
      <td>...</td>
      <td>775551980.0</td>
      <td>214770040.0</td>
      <td>1.604239e+09</td>
      <td>264156080.0</td>
      <td>4.558121e+08</td>
      <td>215193000.0</td>
      <td>70799300.0</td>
      <td>165008920.0</td>
      <td>2.007685e+09</td>
      <td>1.925217e+09</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>327693280.0</td>
      <td>469022020.0</td>
      <td>253873460.0</td>
      <td>191920120.0</td>
      <td>36090220.0</td>
      <td>526130220.0</td>
      <td>3.547430e+09</td>
      <td>195168500.0</td>
      <td>83955380.0</td>
      <td>383211660.0</td>
      <td>...</td>
      <td>765576400.0</td>
      <td>211776760.0</td>
      <td>1.661559e+09</td>
      <td>284553740.0</td>
      <td>4.687606e+08</td>
      <td>212786920.0</td>
      <td>71074600.0</td>
      <td>169339280.0</td>
      <td>2.118029e+09</td>
      <td>1.880283e+09</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>314739720.0</td>
      <td>470999020.0</td>
      <td>241757820.0</td>
      <td>185903480.0</td>
      <td>35533160.0</td>
      <td>518739040.0</td>
      <td>3.406528e+09</td>
      <td>194306960.0</td>
      <td>80596200.0</td>
      <td>370113780.0</td>
      <td>...</td>
      <td>745590980.0</td>
      <td>211893180.0</td>
      <td>1.656444e+09</td>
      <td>269657980.0</td>
      <td>4.723171e+08</td>
      <td>208975800.0</td>
      <td>70378540.0</td>
      <td>168819620.0</td>
      <td>2.101221e+09</td>
      <td>1.844811e+09</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>319459980.0</td>
      <td>466409340.0</td>
      <td>238389420.0</td>
      <td>184779680.0</td>
      <td>35703780.0</td>
      <td>495211280.0</td>
      <td>3.203126e+09</td>
      <td>178621540.0</td>
      <td>58638580.0</td>
      <td>343243600.0</td>
      <td>...</td>
      <td>722505820.0</td>
      <td>204738980.0</td>
      <td>1.565866e+09</td>
      <td>255247100.0</td>
      <td>4.576965e+08</td>
      <td>203607440.0</td>
      <td>68455780.0</td>
      <td>159565120.0</td>
      <td>2.037612e+09</td>
      <td>1.780930e+09</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>294857140.0</td>
      <td>426337080.0</td>
      <td>197233120.0</td>
      <td>174032860.0</td>
      <td>35592920.0</td>
      <td>454311460.0</td>
      <td>2.919061e+09</td>
      <td>167590880.0</td>
      <td>46317120.0</td>
      <td>300145900.0</td>
      <td>...</td>
      <td>658747440.0</td>
      <td>197482680.0</td>
      <td>1.507856e+09</td>
      <td>231082440.0</td>
      <td>4.416315e+08</td>
      <td>185498800.0</td>
      <td>63562480.0</td>
      <td>148468300.0</td>
      <td>2.100980e+09</td>
      <td>1.612189e+09</td>
    </tr>
  </tbody>
</table>
<p>36 rows × 36 columns</p>
</div>



Siamo pronti a tracciare il grafico delle tonnellate di gas serra emesse negli anni. Anche se non è necessariamente la scelta migliore per il tipo di dato a disposizione, utilizziamo una spezzata per unire i punti sul piano cartesiano.


```python
total_emissions.plot.line(y=['IT','FR','DE'])
```




    <AxesSubplot:xlabel='Anno'>




    
![png](Gas_serra_files/Gas_serra_70_1.png)
    


**OSSERVAZIONE**: dal grafico sembra che la situazione della Germania sia stata e sia attualmente molto peggiore di quelle di Francia e Italia, che vanno circa assieme. Bisogna tenere anzitutto presente che questo grafico misura solamente una fetta delle emissioni (sono esclusi il consumo del suolo e il trasporto internazionale). Inoltre, la popolazione tedesca oggi è di circa 83 milioni di persone, contro i 59 milioni dell'Italia e i 68 milioni della Francia. Dunque queste stime andrebbero rapportate alla popolazione (che peraltro è variata anch'essa negli anni!).

Il grafico seguente tiene conto di un aggiustamento minimo, considerando un rapporto con la popolazione che - per semplicità - è stata fissata come costante negli anni analizzati. Questa decisione influenza la nostra già fin troppo semplificata analisi, che però ha scopi dimostrativi e non vuole essere tecnicamente ineccepibile. In un'analisi più accurata si potrebbero quindi sfruttare i dati demografici sull'andamento della popolazione nel tempo, tratti proprio dai database di Eurostat.


```python
corrected_emissions = pd.DataFrame(data={
    'IT': total_emissions['IT']/59e6,
    'FR': total_emissions['FR']/68e6,
    'DE': total_emissions['DE']/83e6
})
corrected_emissions.plot.line(y=['IT','FR','DE'])
```




    <AxesSubplot:xlabel='Anno'>




    
![png](Gas_serra_files/Gas_serra_72_1.png)
    


**OSSERVAZIONE:** al grafico mancano titolo, informazioni lungo gli assi, ecc.

Per alcuni tipi di personalizzazione ricorriamo all'importazione diretta di una parte di `matplotlib`. I diversi parametri dovrebbero essere autoesplicativi.


```python
import matplotlib.pyplot as plt

years = [*range(1990,2021)] # Lista di tutti i numeri da 1990 a 2020
countries = ['IT','FR','DE']

corrected_emissions.loc[years,:].plot.line(
    y = countries,
    title = 'Confronto delle emissioni nel tempo',
    xlabel = 'Anno',
    ylabel = 'Tonnellate emesse per abitante',
    label = [geo_dic[k] for k in countries],
    rot = 45, # Rotazione delle etichette sull'asse delle ascisse
    grid = True
)

# Rappresento i dati sull'asse verticale in notazione decimale
plt.ticklabel_format(style='plain')
```


    
![png](Gas_serra_files/Gas_serra_74_0.png)
    


**OSSERVAZIONE:** i paesi confrontati partono da situazioni iniziali diverse e dunque non è detto che i livelli assoluti siano confrontabili.

Si può avere una migliore idea del successo di un paese nel ridurre le emissioni rispetto ad un certo anno di partenza se si traccia un grafico della riduzione delle emissioni rapportate a quell'anno.


```python
import matplotlib.ticker as mtick

emissions_ratio = 100*corrected_emissions.loc[years]/corrected_emissions.loc[years[0],:]

ax = emissions_ratio.plot.line(
    y = countries,
    title = 'Confronto delle emissioni nel tempo',
    xlabel = 'Anno',
    ylabel = 'Percentuale di tonnellate emesse per abitante\nrispetto al ' + str(years[0]),
    label = [geo_dic[k] for k in countries],
    rot = 45, # Rotazione delle etichette sull'asse delle ascisse
    grid = True
)

# Rappresento i dati sull'asse verticale in notazione decimale
plt.ticklabel_format(style='plain')
ax.yaxis.set_major_formatter(mtick.PercentFormatter())
```


    
![png](Gas_serra_files/Gas_serra_76_0.png)
    


Questo grafico chiarisce che la Germania, che inizialmente sembrava essere la pecora nera delle tre, in realtà è l'entità geopolitica che è ha ridotto di più le proprie emissioni a partire dal 1990. Per un confronto serio, l'analisi non dovrebbe fermarsi qui e dovrebbe certamente tenere conto di una quantità molto maggiore di fattori.

Proponiamo anche una versione diversa in cui la percentuale è di volta in volta ricalcolata come variazione rispetto all'anno precedente e viene considerata la somma cumulativa delle percentuali.


```python
emissions_rate_cumsum = 100*corrected_emissions.pct_change().cumsum()

ax = emissions_rate_cumsum.plot.line(
    y = countries,
    title = 'Confronto delle emissioni nel tempo',
    xlabel = 'Anno',
    ylabel = 'Cumulativa della variazione percentuale delle tonnellate emesse\nper abitante rispetto all''anno precedente',
    label = [geo_dic[k] for k in countries],
    rot = 45, # Rotazione delle etichette sull'asse delle ascisse
    grid = True
)

# Rappresento i dati sull'asse verticale in notazione decimale
plt.ticklabel_format(style='plain')
ax.yaxis.set_major_formatter(mtick.PercentFormatter())
```


    
![png](Gas_serra_files/Gas_serra_78_0.png)
    


Qualitativamente, l'andamento è lo stesso di prima. Numericamente stiamo invece tenendo conto della variazione dell'ammontare totale di anno in anno (da questo la differenza nei numeri se si confronta questo con il grafico precedente).

**NOTA:** una buona rappresentazione grafica, in questo caso, sarebbe probabilmente quella tramite un grafico a barre. Avendo solo un dato per anno e non potendo dire quindi molto sugli andamenti stagionali o mensili, che non sono ben descritti dai segmenti rettilinei che li rappresentano, la barra rappresenta meglio il dato. Inoltre, si potrebbero raggruppare gli anni in periodi di 5 anni (o anche più brevi), per mitigare le oscillazioni più piccole.


```python
binned_emissions_rate = corrected_emissions.groupby(pd.cut(
    x = corrected_emissions.index,
    bins = pd.interval_range(start = years[0], end = years[-1], freq = 5)
)).sum().pct_change().cumsum()

binned_emissions_rate.plot.bar(
    y = countries,
    title = 'Confronto delle emissioni nel tempo',
    xlabel = 'Anno',
    ylabel = 'Cumulativa del rapporto tra le tonnellate emesse per abitante\ne le tonnellate emesse per abitante nel periodo precedente',
    label = [geo_dic[k] for k in countries],
    rot = 90 # Rotazione delle etichette sull'asse delle ascisse
)
```




    <AxesSubplot:title={'center':'Confronto delle emissioni nel tempo'}, xlabel='Anno', ylabel='Cumulativa del rapporto tra le tonnellate emesse per abitante\ne le tonnellate emesse per abitante nel periodo precedente'>




    
![png](Gas_serra_files/Gas_serra_80_1.png)
    


### Torta delle emissioni nel 2020
Con lo stesso procedimento seguito in precedenza si può ottenere una tabella a doppia entrata. Lungo le righe i diversi inquinanti, lungo le colonne le entità geopolitiche.


```python
emissions_by_agents = data[data['Anno'] == 2020][['Inquinante','Entità geopolitica','Tonnellate emesse']]
emissions_by_agents = emissions_by_agents.groupby(['Inquinante','Entità geopolitica'])['Tonnellate emesse'].sum()
emissions_by_agents = emissions_by_agents.unstack()
emissions_by_agents
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Entità geopolitica</th>
      <th>AT</th>
      <th>BE</th>
      <th>BG</th>
      <th>CH</th>
      <th>CY</th>
      <th>CZ</th>
      <th>DE</th>
      <th>DK</th>
      <th>EE</th>
      <th>EL</th>
      <th>...</th>
      <th>NL</th>
      <th>NO</th>
      <th>PL</th>
      <th>PT</th>
      <th>RO</th>
      <th>SE</th>
      <th>SI</th>
      <th>SK</th>
      <th>TR</th>
      <th>UK</th>
    </tr>
    <tr>
      <th>Inquinante</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CH4</th>
      <td>4.572040e+06</td>
      <td>5.572540e+06</td>
      <td>4310640.0</td>
      <td>3622120.0</td>
      <td>727940.0</td>
      <td>8.665980e+06</td>
      <td>3.883878e+07</td>
      <td>5727020.0</td>
      <td>887980.0</td>
      <td>7.351860e+06</td>
      <td>...</td>
      <td>1.335776e+07</td>
      <td>3746000.0</td>
      <td>3.334414e+07</td>
      <td>6.904620e+06</td>
      <td>1.710872e+07</td>
      <td>3446880.0</td>
      <td>1467420.0</td>
      <td>2464220.0</td>
      <td>4.944166e+07</td>
      <td>3.782122e+07</td>
    </tr>
    <tr>
      <th>CH4_CO2E</th>
      <td>1.143008e+08</td>
      <td>1.393125e+08</td>
      <td>107768160.0</td>
      <td>90553100.0</td>
      <td>18199900.0</td>
      <td>2.166513e+08</td>
      <td>9.709688e+08</td>
      <td>143174400.0</td>
      <td>22199100.0</td>
      <td>1.837961e+08</td>
      <td>...</td>
      <td>3.339460e+08</td>
      <td>93649440.0</td>
      <td>8.336043e+08</td>
      <td>1.726140e+08</td>
      <td>4.277179e+08</td>
      <td>86171340.0</td>
      <td>36684180.0</td>
      <td>61607420.0</td>
      <td>1.236041e+09</td>
      <td>9.455318e+08</td>
    </tr>
    <tr>
      <th>CO2</th>
      <td>1.356595e+09</td>
      <td>2.044596e+09</td>
      <td>656244780.0</td>
      <td>751346680.0</td>
      <td>154972080.0</td>
      <td>2.153817e+09</td>
      <td>1.322062e+10</td>
      <td>682542460.0</td>
      <td>226412160.0</td>
      <td>1.148699e+09</td>
      <td>...</td>
      <td>3.255480e+09</td>
      <td>649353540.0</td>
      <td>6.065315e+09</td>
      <td>8.279434e+08</td>
      <td>1.144409e+09</td>
      <td>433281400.0</td>
      <td>222697940.0</td>
      <td>537752860.0</td>
      <td>7.739634e+09</td>
      <td>6.910734e+09</td>
    </tr>
    <tr>
      <th>GHG</th>
      <td>1.577848e+09</td>
      <td>2.350940e+09</td>
      <td>885234860.0</td>
      <td>924129580.0</td>
      <td>184897480.0</td>
      <td>2.542152e+09</td>
      <td>1.494850e+10</td>
      <td>938608180.0</td>
      <td>272685400.0</td>
      <td>1.507069e+09</td>
      <td>...</td>
      <td>3.760078e+09</td>
      <td>807627240.0</td>
      <td>7.441336e+09</td>
      <td>1.126837e+09</td>
      <td>1.810830e+09</td>
      <td>639506560.0</td>
      <td>279839340.0</td>
      <td>648146160.0</td>
      <td>9.834852e+09</td>
      <td>8.453898e+09</td>
    </tr>
    <tr>
      <th>HFC_CO2E</th>
      <td>3.161862e+07</td>
      <td>5.966140e+07</td>
      <td>30680800.0</td>
      <td>24953880.0</td>
      <td>6439140.0</td>
      <td>7.234878e+07</td>
      <td>1.582351e+08</td>
      <td>6022100.0</td>
      <td>3325320.0</td>
      <td>9.220824e+07</td>
      <td>...</td>
      <td>2.073496e+07</td>
      <td>14579460.0</td>
      <td>9.397744e+07</td>
      <td>6.000876e+07</td>
      <td>3.579390e+07</td>
      <td>16895000.0</td>
      <td>5310540.0</td>
      <td>12219820.0</td>
      <td>1.053569e+08</td>
      <td>2.103844e+08</td>
    </tr>
    <tr>
      <th>HFC_PFC_NSP_CO2E</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>2.339940e+06</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>N2O</th>
      <td>2.265800e+05</td>
      <td>3.443800e+05</td>
      <td>302740.0</td>
      <td>182760.0</td>
      <td>16700.0</td>
      <td>3.297200e+05</td>
      <td>1.825820e+06</td>
      <td>356160.0</td>
      <td>69340.0</td>
      <td>2.670400e+05</td>
      <td>...</td>
      <td>4.923800e+05</td>
      <td>154160.0</td>
      <td>1.499340e+06</td>
      <td>2.197000e+05</td>
      <td>6.763000e+05</td>
      <td>339960.0</td>
      <td>49420.0</td>
      <td>121460.0</td>
      <td>2.520660e+06</td>
      <td>1.268240e+06</td>
    </tr>
    <tr>
      <th>N2O_CO2E</th>
      <td>6.752516e+07</td>
      <td>1.026580e+08</td>
      <td>90223360.0</td>
      <td>54467120.0</td>
      <td>4995420.0</td>
      <td>9.823100e+07</td>
      <td>5.441253e+08</td>
      <td>106140240.0</td>
      <td>20702080.0</td>
      <td>7.961992e+07</td>
      <td>...</td>
      <td>1.467137e+08</td>
      <td>45957500.0</td>
      <td>4.468225e+08</td>
      <td>6.547672e+07</td>
      <td>2.015376e+08</td>
      <td>101362140.0</td>
      <td>14718860.0</td>
      <td>36189980.0</td>
      <td>7.511693e+08</td>
      <td>3.779572e+08</td>
    </tr>
    <tr>
      <th>NF3_CO2E</th>
      <td>2.167200e+05</td>
      <td>1.531800e+05</td>
      <td>0.0</td>
      <td>7380.0</td>
      <td>0.0</td>
      <td>3.870000e+04</td>
      <td>1.944000e+05</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>6.480000e+03</td>
    </tr>
    <tr>
      <th>PFC_CO2E</th>
      <td>5.380200e+05</td>
      <td>3.094380e+06</td>
      <td>180.0</td>
      <td>579260.0</td>
      <td>0.0</td>
      <td>1.836000e+04</td>
      <td>3.730500e+06</td>
      <td>180.0</td>
      <td>0.0</td>
      <td>2.666700e+06</td>
      <td>...</td>
      <td>1.210320e+06</td>
      <td>2905560.0</td>
      <td>1.839600e+05</td>
      <td>4.280400e+05</td>
      <td>6.390000e+04</td>
      <td>1173960.0</td>
      <td>173160.0</td>
      <td>100980.0</td>
      <td>6.809400e+05</td>
      <td>2.717460e+06</td>
    </tr>
    <tr>
      <th>SF6_CO2E</th>
      <td>7.053100e+06</td>
      <td>1.464420e+06</td>
      <td>317440.0</td>
      <td>2222060.0</td>
      <td>290880.0</td>
      <td>1.046680e+06</td>
      <td>4.828774e+07</td>
      <td>728640.0</td>
      <td>46720.0</td>
      <td>7.904000e+04</td>
      <td>...</td>
      <td>1.993280e+06</td>
      <td>1181800.0</td>
      <td>1.432640e+06</td>
      <td>3.660800e+05</td>
      <td>1.307680e+06</td>
      <td>622880.0</td>
      <td>254880.0</td>
      <td>275200.0</td>
      <td>1.969620e+06</td>
      <td>6.565920e+06</td>
    </tr>
  </tbody>
</table>
<p>11 rows × 33 columns</p>
</div>



Ed ecco la torta delle emissioni italiane nel 2020, con le conversioni in ammontare di CO<sub>2</sub>:


```python
emissions_by_agents_IT = emissions_by_agents.loc[airpol_dic_noagg_co2e.keys(),'IT']
emissions_by_agents_IT.plot.pie(
    autopct='%1.1f%%',
    labels = airpol_dic_noagg_co2e.values(),
    explode = (0.1,) * len(airpol_dic_noagg_co2e),
    ylabel = ''
)
```




    <AxesSubplot:>




    
![png](Gas_serra_files/Gas_serra_84_1.png)
    


Alcuni valori osservati sono troppo piccoli in confronto agli altri. Proviamo a raggrupparli in un'unica categoria chiamata genericamente `'Other greenhouse gases (CO2 equivalent)'`. Aggreghiamo in questa categoria tutti i gas che costituiscono ciascuno una percentuale inferiore ad 1% del totale delle emissioni. Costruiamo quindi la serie di dati che contenga questa categoria.


```python
# Data selection and aggregation
to_include = (emissions_by_agents_IT/emissions_by_agents_IT.sum() >= 0.01)
emissions_by_agents_IT_mod = pd.concat([
    emissions_by_agents_IT[to_include],
    pd.Series(data=emissions_by_agents_IT[~to_include].sum(), index=['Oth'])
])
```

Costruiamo adesso il dizionario per questa serie di dati modificata:


```python
# Dictionary modification
airpol_dic_noagg_co2e_mod = airpol_dic_noagg_co2e.copy()
airpol_dic_noagg_co2e_mod['Oth'] = 'Other greenhouse gases (CO2 equivalent)'
for k in emissions_by_agents_IT[~to_include].index:
    del airpol_dic_noagg_co2e_mod[k]
```

Il grafico adesso risulta come segue:


```python
# Plotting
emissions_by_agents_IT_mod.plot.pie(
    autopct='%1.1f%%',
    labels = airpol_dic_noagg_co2e_mod.values(),
    explode = (0.1,) * len(airpol_dic_noagg_co2e_mod),
    ylabel = ''
)
```




    <AxesSubplot:>




    
![png](Gas_serra_files/Gas_serra_90_1.png)
    

