# DKB → CSV → YNAB

Transform DKB transaction CSV files so that YNAB can understand them.

## Use online

- https://jurihock.de/dkb-csv-ynab
- https://jurihock.github.io/dkb-csv-ynab

> [!NOTE]
> This website doesn't use any cookies or any other tracking technologies.
> Your data is processed locally in your browser and isn't sent anywhere else.

## Run offline

```
npm install
npm run dev
```

## Details

### Original DKB format

```
"Girokonto";"IBAN"

"Kontostand vom DD.MM.YYYY:";"1.234,56 €"
""
"Buchungsdatum";"Wertstellung";"Status";"Zahlungspflichtige*r";"Zahlungsempfänger*in";"Verwendungszweck";"Umsatztyp";"IBAN";"Betrag (€)";"Gläubiger-ID";"Mandatsreferenz";"Kundenreferenz"
...
```

### Converted YNAB format

```
"Date","Payee","Memo","Outflow","Inflow"
...
```

### CSV column mapping

| DKB | YNAB | Format |
| --- | ---- | ------ |
| `Wertstellung` or `Buchungsdatum` | `Date` | `YYYY-MM-DD` |
| `Zahlungsempfänger*in` or `IBAN` | `Payee` | |
| `Verwendungszweck` | `Memo` | |
| `Betrag (€)` or `Betrag` | `Outflow` | `≤ 0` `1234.56` |
| `Betrag (€)` or `Betrag` | `Inflow` | `≥ 0` `1234.56` |
