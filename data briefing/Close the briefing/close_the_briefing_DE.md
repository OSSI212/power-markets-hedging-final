# Close the briefing

Kurze Antworten auf die drei Abschlussfragen der Data Description (Abschnitt 3),
auf Basis der Exhibits 1-4. Alle Zahlen sind auf zwei Nachkommastellen gerundet.

---

## 1. Was war am 29. September 2023 bekannt, und was wurde erst während der Lieferung bekannt?

**Verfügbar bei der Auslegung des Hedges (Quote-Datum 29.09.2023):**

- Der Futures-Snapshot in `futures_prices.csv`: Base- und Peak-Terminpreise sowie der
  `market_activity`-Handelbarkeitsindikator für CAL, die vier Quartale und die zwölf Monate 2024.
- Der Kundenbestand: Kundenzahl und Jahresverbrauch pro Kunde für die drei Gruppen HB, GB, LB
  (Exhibit 1).
- Die veröffentlichten Standardlastprofile 2024 (`slp_profiles.csv`) - normierte Formen, keine
  Mengen.
- Die aus 2019-2022 abgeleitete historische Day-Ahead-Preisform (`shape_factors.csv`, Exhibit 3).

Alles, was zum Aufbau von Prognoselast, HPFC und Hedge nötig ist, steckt in diesem Datensatz. Der
Hedge ist eine **Ex-ante-Entscheidung**.

**Erst während der Lieferung bekannt (2024):**

- Realisierte Day-Ahead-Preise (`day_ahead_prices.csv`) und realisierte Ausgleichsenergiepreise /
  reBAP (`imbalance_prices.csv`), Exhibit 4.
- Tatsächlicher Kundenverbrauch je Viertelstunde (`actual_portfolio_load.csv`): realisiert
  43.138,00 MWh gegenüber prognostizierten 43.000,00 MWh - und, wichtiger als die Jahressumme, die
  viertelstündlichen Abweichungen, die die Ausgleichsenergie treiben.
- Das Wetter, die Kältephase im Dezember 2024 (Day-Ahead-Spitze 936,28 EUR/MWh am 12. Dezember),
  der isolierte reBAP-Ausschlag im Juni (14.999,99 EUR/MWh am 3. Juni), Kraftwerksausfälle und
  Brennstoffpreisbewegungen - jeder Treiber der Lücke zwischen Ex-ante-Plan und realisiertem
  Ergebnis.

Die Stages 4-7 bewerten den Hedge gegen Informationen, die es bei seiner Auslegung noch nicht gab:
Der Futures-Payoff ist am 29.09.2023 fixiert, die Rest-Day-Ahead- und Ausgleichsenergiekosten
nicht.

---

## 2. Datenprüfungen vor dem Vertrauen in die Analyse

- **Abdeckung.** Jede 2024-Reihe (`slp_profiles`, `shape_factors`, `day_ahead_prices`,
  `imbalance_prices`, `actual_portfolio_load`) hat exakt 35.136 Viertelstunden = 366 Tage x 96.
  `futures_prices` hat 34 Kontrakte (2 Jahr, 8 Quartal, 24 Monat).
- **Zeitbasis.** Alle Reihen werden über `timestamp_utc` gejoint. `timestamp_local` trägt
  wechselnde UTC-Offsets (+01:00 / +02:00) und wird daher nur für Wanduhr-Merkmale genutzt. Die
  Sommer-/Winterzeitumstellung steckt bereits in den Zeitstempeln: Q1 = 8.732 Viertelstunden,
  Q4 = 8.836; M03 = 2.972, M10 = 2.980.
- **Fehlende Werte.** Keine in einer der sechs Dateien.
- **Einheiten.** Energie in MWh, Preise in EUR/MWh, SLP-Spalten in normierten kWh. Der
  Day-Ahead-Preis ist ein Stundenwert, der über seine vier Viertelstunden wiederholt wird
  (geprüft).
- **Interne Konsistenz:**
  - jede SLP-Spalte summiert sich auf exakt 1.000.000,00 kWh; die skalierte Prognose stimmt bis auf
    Gleitkomma-Genauigkeit mit E_p = 35.000,00 / 6.000,00 / 2.000,00 MWh und 43.000,00 MWh
    insgesamt überein;
  - CAL Base (77,26) entspricht dem N_all-gewichteten Mittel der vier Quartals-Base-Futures;
    CAL Peak (86,60) dem N_peak-gewichteten Mittel; Q1 Base (66,42) dem Mittel von M01-M03;
    gleiches gilt für Q1 Peak (77,70);
  - der historische Shape-Faktor hat das Jahresmittel 1,00;
  - die implizite Off-Peak-Identität `P_base * N_all = P_peak * N_peak + P_off * N_off` reproduziert
    den notierten Base-Preis, wenn die rekonstruierte Stundenkurve über einen Block zurückgemittelt
    wird.
- **Handelbarkeit.** `market_activity` teilt sich klar - illiquide <= 10, handelbar >= 182, nichts
  dazwischen. Die Aufteilung brauchbar / illiquide ändert sich für jede Schwelle in dieser Lücke
  nicht und stimmt mit einer Regel "mindestens 5 % der CAL-Aktivität derselben Load-Art" überein.
- **Plausibilität.** Quantile und Extremwerte jeder Preisreihe wurden geprüft; das Day-Ahead-Maximum
  (936,28 am 12.12.) und der reBAP-Deckel (14.999,99 am 3.6.) wurden auf konkrete Daten
  zurückverfolgt und gegen die Tagesmittelreihe gegengeprüft.

---

## 3. Was würde für ein echtes Retail-Portfolio weiterhin fehlen?

Fünf Vereinfachungen, mit der wahrscheinlichen Wirkungsrichtung:

1. **Keine Handelsfriktionen.** `market_activity` ist ein synthetischer Wert; Geld-Brief-Spannen,
   Transaktionskosten, Brokergebühren und ganzzahlige Kontraktgrößen sind nicht modelliert,
   Positionen sind kontinuierliche MW. *Wirkung:* der granulare Hedge wäre teurer umzusetzen als
   gezeigt - Spreads sind bei Quartals- und Monatsprodukten breiter -, was seinen Vorteil
   gegenüber dem groben CAL-Hedge verkleinert und umkehren könnte.
2. **Standardlastprofile statt gemessener Last.** Die Prognose nutzt feste SLP-Formen und nimmt an,
   dass Kundenzahl und Verbrauch pro Kunde exakt bekannt sind, ohne Lieferantenwechsel, Abwanderung
   oder Neukundengewinnung im Jahresverlauf. *Wirkung:* reales Form- und Mengenrisiko sind größer
   als im Modell, daher sind die Rest-Day-Ahead-Kosten (Stage 4) und die Ausgleichsenergie-
   Abrechnung (Stage 5) untertrieben - der Hedge wirkt besser als in der Praxis.
3. **Historische Preisform aus 2019-2022.** Diese Jahre umfassen die Gaspreiskrise und weniger
   installierte Photovoltaik als 2024. *Wirkung:* die HPFC-Form kann die Mittagsstunden (2024 mehr
   Solar) und die Abendrampe falsch bepreisen und verzerrt so die Value-Neutrality-Aufteilung und
   die Rest-Form-Schätzung (RMSE) in Stage 3; am wahrscheinlichsten ist eine systematische
   Überbewertung der Mittagsstunden.
4. **Ein einziges Quote-Datum und kein Rebalancing.** Der Hedge wird einmalig am 29.09.2023
   gesetzt; es gibt keinen Intraday-Markt, keinen kontinuierlichen Handel und keine Anpassung, wenn
   sich die Lastprognose im Lauf von 2024 aktualisiert. *Wirkung:* ein echter Desk würde die
   Position über Monate aufbauen und Restrisiko abbauen, sobald Information eintrifft; das
   Einmal-Ergebnis ist eine konservative (pessimistische) Schätzung der Rest-Exposure.
5. **Symmetrischer reBAP und kein Price-Impact.** Ausgleichsenergie wird zu einem veröffentlichten
   symmetrischen Preis abgerechnet, und das Portfolio wird als zu klein angenommen, um ihn zu
   bewegen. *Wirkung:* in einer angespannten, systemweiten Phase wie der Dezember-Kältephase 2024
   hätte ein großes, unterdecktes Retail-Portfolio eine schlechtere Abrechnung als der historische
   reBAP nahelegt - die Imbalance-Prämie in Stage 5 ist für solche Ereignisse eine Untergrenze.

Weitere Auslassungen: keine Kredit-, Margin- oder Sicherheitenanforderungen; keine Risikolimits
oder regulatorischen / netzseitigen Vorgaben; keine Erneuerbaren-PPAs oder Demand-Response; und
reBAP ist ein Abrechnungspreis, keine Prognose - Stage 5 stützt sich also auf eine realisierte
2024-Reihe, die ex ante selbst unbekannt war.
