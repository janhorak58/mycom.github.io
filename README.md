# Dotazy

> **Poznámka:**  
> Chatbot v této fázi pouze převádí analytické dotazy na validní databázové dotazy (query).  
> **Nedokáže odpovídat na obecné otázky** jako *"Jaké činnosti doporučuješ prověřit?"* nebo *"Je tato cesta obvyklá?"*.  

---

## Příkladové dotazy

### Jaké nové účty byly založeny tento týden?
- Filtr:
  - **Event ID:** `4720`
- Výstup:
  - Seznam uživatelských jmen (`data.win.eventdata.targetUserName`)

```json
"query": {
    "bool": {
        "must": [],
        "filter": [
            {
                "range": {
                    "@timestamp": {
                        "gte": "now-1w/w",
                        "lte": "now/w"
                    }
                }
            },
            {
                "terms": {
                    "data.win.system.eventID": [
                        4720
                    ]
                }
            }
        ]
    }
},
```

---

### Do jakých skupin byl účet `lenovo_tmp_wrpiBBBF` přiřazen?
- Filtr:
  - **Event ID:** [`4728`, `4732`, `4756`] 
  - **Uživatelské jméno:** `data.win.eventdata.targetUserName`
- Výstup:
  - **Skupiny:** `rule.groups`

```json
"query": {
    "bool": {
        "must": [],
        "filter": [
            {
                "match_phrase": {
                    "data.win.eventdata.targetUserName": "lenovo_tmp_wrpiBBBF"
                }
            },
            {
                "range": {
                    "@timestamp": {
                        "gte": "now-1d/d",
                        "lte": "now/d"
                    }
                }
            },
            {
                "terms": {
                    "data.win.system.eventID": [
                        4728,
                        4732,
                        4756
                    ]
                }
            }
        ]
    }
},
```


---

### Na kolik zařízení se přihlásil účet `lenovo_tmp_wrpiBBBF`? Jaké byly rozestupy mezi přihlášeními?
- Filtr:
  - **Uživatelské jméno:** `data.win.eventdata.targetUserName`
  - **Úspěšné přihlášení:** `Event ID 4624`
  - **Neúspěšné pokusy:** `Event ID 4625`
- Výstup:
  - Časy přihlášení účtu (`rule.timestamp`)
  - Počet hitů s eventID 4624 s různým zařízením (`data.win.eventdata.workstationName` /
`data.win.eventdata.targetDomainName` / `agent.name`)

```json
"query": {
    "bool": {
        "must": [
            {
                "match_phrase": {
                    "data.win.eventdata.targetUserName": "lenovo_tmp_wrpiBBBF"
                }
            }
        ],
        "filter": [
            {
                "range": {
                    "@timestamp": {
                        "gte": "now-1d/d",
                        "lte": "now/d"
                    }
                }
            },
            {
                "terms": {
                    "data.win.system.eventID": [
                        4624,
                        4625
                    ]
                }
            }
        ]
    }
}
```

---

### Jaká byla činnost účtu `lenovo_tmp_usfvHFTX` na serveru `NBPEG001`?
- Filtr:
  - **Uživatelské jméno:** `data.win.eventdata.targetUserName`
  - **Název serveru:** `agent.name`
- Výstup
  - Název procesu: `data.win.eventdata.newProcessName`
  - Elevace oprávnění: `data.win.eventdata.tokenElevationType: %%1937`

```json
"query": {
    "bool": {
        "must": [],
        "filter": [
            {
                "match_phrase": {
                    "data.win.eventdata.targetUserName": "lenovo_tmp_usfvHFTX"
                }
            },
            {
                "match_phrase": {
                    "agent.name": "NBPEG001"
                }
            },
            {
                "range": {
                    "@timestamp": {
                        "gte": "now-1d/d",
                        "lte": "now/d"
                    }
                }
            }
        ]
    }
}
```

---

### Jaké další aplikace spustil účet `GTT-LAB-04$`?
- Filtr:
  - **Uživatelské jméno:** `data.win.eventdata.targetUserName`
  - **Nový proces:** `Event ID 4688`
  - **Existuje pole Nového názvu procesu:** `data.win.eventdata.newProcessName`
- Výstup:
  - **Nový název procesu:** `data.win.eventdata.newProcessName`

```json
"query": {
    "bool": {
        "must": [
            {
                "match_phrase": {
                    "data.win.eventdata.targetUserName": "GTT-LAB-04$"
                }
            }
        ],
        "filter": [
            {
                "range": {
                    "@timestamp": {
                        "gte": "now-1d/d",
                        "lte": "now/d"
                    }
                }
            },
            {
                "terms": {
                    "data.win.system.eventID": [
                        4688
                    ]
                }
            }
        ]
    }
},
```

---

### Založil účet `MSOL_5949b6813aca` nějaké služby?
filtr:
  - **Uživatelské jméno:** `data.win.eventdata.targetUserName`
  - **Založení služby:** `Event ID 4697`
  > Pozn. U této formulace otázky mám model problém  srozeznáním vhodného eventID. Buď vrátí 4720, nebo žádný.
  - **Existuje pole založeného služby:** `data.win.eventdata.serviceName`
- Výstup:
  - **Založená služba:** `data.win.eventdata.newProcessName`

```json
{
    "bool": {
        "must": [
            {
                "match_phrase": {
                    "data.win.eventdata.targetUserName": "MSOL_5949b6813aca"
                }
            }
        ],
        "filter": [
            {
                "range": {
                    "@timestamp": {
                        "gte": "now-1d/d",
                        "lte": "now/d"
                    }
                }
            },
            {
                "terms": {
                    "data.win.system.eventID": [
                        4720
                    ]
                }
            }
        ]
    }
}
```

---

### Prověř naplánované úlohy
> Poznámka: velmi obecný dotaz   
- Filtr:
  - **Existuje atribut názvu úlohy:** `data.win.eventdata.taskName`
- **Event ID (nezafiltruje se):** `4698, 4699, 4702` 
Výstup:
- **Při analýze:**  
  - Název úlohy: `data.win.eventdata.taskName`

```json
"query": {
        "bool": {
            "must": [],
            "filter": [
                {
                    "exists": {
                        "field": "data.win.eventdata.taskName"
                    }
                },
                {
                    "range": {
                        "@timestamp": {
                            "gte": "now-1d/d",
                            "lte": "now/d"
                        }
                    }
                }
            ]
        }
    }
```

---

## Dotazy které nejsou v této fázi relevantní
- *Je masivní čtení aplikací `notepad.exe` obvyklé?*  
- *Je tato cesta pro procesy obvyklá?*  
  - Například analýza činnosti za poslední měsíc
- *Jaké další činnosti doporučuješ prověřit?*  



---



## Mapování atributů na eventy

<table>
  <thead>
    <tr>
      <th>Atribut</th>
      <th>Klíč</th>
      <th>Popis</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Event ID</strong></td>
      <td><code>data.win.system.eventID</code></td>
      <td>Windows Event ID</td>
    </tr>
    <tr>
      <td><strong>Uživatelské jméno</strong></td>
      <td><code>data.win.eventdata.targetUserName</code></td>
      <td>Uživatelské jméno spojené s událostí</td>
    </tr>
    <tr>
      <td><strong>Site, kde byl účet vytvořen</strong></td>
      <td><code>agent.labels.mcs.group.firma</code></td>
      <td>Skupina, kde byl účet vytvořen</td>
    </tr>
    <tr>
      <td><strong>Název stanice (při neúspěšném přihlášení)</strong></td>
      <td><code>data.win.eventdata.workstationName</code></td>
      <td>Název počítače u neúspěšného přihlášení</td>
    </tr>
    <tr>
      <td><strong>Název serveru</strong></td>
      <td><code>agent.name</code></td>
      <td>Název serveru, tenantu, kde se událost odehrála</td>
    </tr>
    <tr>
      <td><strong>Název procesu</strong></td>
      <td><code>data.win.eventdata.newProcessName / data.win.eventdata.processName</code></td>
      <td>Název spuštěného procesu</td>
    </tr>
    <tr>
      <td><strong>Přístup k objektu (souboru, složce, apod.)</strong></td>
      <td><code>data.win.eventdata.objectName</code></td>
      <td>Název nebo cesta objektu</td>
    </tr>
    <tr>
      <td><strong>Přístupová oprávnění</strong></td>
      <td><code>data.win.eventdata.accesses</code></td>
      <td>Oprávnění k přístupu k objektu</td>
    </tr>
    <tr>
      <td><strong>Bezpečnostní identifikátor uživatele (SID)</strong></td>
      <td><code>data.win.eventdata.securityID</code></td>
      <td>Unikátní bezpečnostní ID uživatele</td>
    </tr>
    <tr>
      <td><strong>Identifikátor handle objektu</strong></td>
      <td><code>data.win.eventdata.handleID</code></td>
      <td>ID handle pro objekt</td>
    </tr>
    <tr>
      <td><strong>Elevace oprávnění procesu</strong></td>
      <td><code>data.win.eventdata.tokenElevationType</code></td>
      <td>Typ zvýšení oprávnění procesu</td>
    </tr>
    <tr>
      <td><strong>Instalace systémové služby</strong></td>
      <td><code>data.win.eventdata.serviceFileName</code></td>
      <td>Cesta k souboru instalované služby</td>
    </tr>
    <tr>
      <td><strong>Název služby</strong></td>
      <td><code>data.win.eventdata.serviceName</code></td>
      <td>Název systémové služby</td>
    </tr>
    <tr>
      <td><strong>Název úlohy</strong></td>
      <td><code>data.win.eventdata.taskName</code></td>
      <td>Název plánované úlohy nebo úkolu</td>
    </tr>
  </tbody>
</table>

---

## Validace query

Validace by mohla probíhat kontrolou proti **fixnímu seznamu mapovaných atributů**.  
Pokud dotaz obsahuje neznámý atribut, model vrátí varování nebo chybu.
