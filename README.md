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
| Atribut | Klíč | Popis |
|--------------|------------------|---------------------------|
| **Event ID** | `data.win.system.eventID` | Windows Event ID |
| **Uživatelské jméno** | `data.win.eventdata.targetUserName` | Uživatelské jméno spojené s událostí |
| **Site, kde byl účet vytvořen** | `agent.labels.mcs.group.firma` | Skupina, kde byl účet vytvořen |
| **Název stanice (při neúspěšném přihlášení)** | `data.win.eventdata.workstationName` | Název počítače u neúspěšného přihlášení |
| **Název serveru** | `agent.name` | Název serveru, tenantu, kde se událost odehrála |
| **Název procesu** | `data.win.eventdata.newProcessName / data.win.eventdata.processName` | Název spuštěného procesu |
| **Přístup k objektu (souboru, složce, apod.)** | `data.win.eventdata.objectName` | Název nebo cesta objektu |
| **Přístupová oprávnění** | `data.win.eventdata.accesses` | Oprávnění k přístupu k objektu |
| **Bezpečnostní identifikátor uživatele (SID)** | `data.win.eventdata.securityID` | Unikátní bezpečnostní ID uživatele |
| **Identifikátor handle objektu** | `data.win.eventdata.handleID` | ID handle pro objekt |
| **Elevace oprávnění procesu** | `data.win.eventdata.tokenElevationType` | Typ zvýšení oprávnění procesu |
| **Instalace systémové služby** | `data.win.eventdata.serviceFileName` | Cesta k souboru instalované služby |
| **Název služby** | `data.win.eventdata.serviceName` | Název systémové služby |
| **Název úlohy** | `data.win.eventdata.taskName` | Název plánované úlohy nebo úkolu |

---

## Validace query

Validace by mohla probíhat kontrolou proti **fixnímu seznamu mapovaných atributů**.  
Pokud dotaz obsahuje neznámý atribut, model vrátí varování nebo chybu.
