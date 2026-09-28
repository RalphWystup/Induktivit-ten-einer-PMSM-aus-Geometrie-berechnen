# Die Induktivitäten einer PMSM aus Geometrie und Werkstoffdaten

<img src="Foto_Ralph_Wystup.jpg" align="right" width="140" alt="Prof. Dr.-Ing. Ralph Wystup">

Prof. Dr.-Ing. Ralph Wystup M.Sc. — erstellt mit KI und Agent (Claude Code, Anthropic)

Eine Maschine hat nicht eine Induktivität je Achse, sondern zwei, und sie sind verschieden. Welche von beiden gemeint ist, entscheidet sich an der Gleichung, in die sie eingesetzt wird. Wer die falsche nimmt, rechnet bei Sättigung um Faktoren daneben.

**Seite öffnen:** https://ralphwystup.github.io/Induktivit-ten-einer-PMSM-aus-Geometrie-berechnen/ — Fassung 2.7 · 28. September 2026 · Fassung 2.7

Die Seite ist **eine einzige HTML-Datei**. Sie rechnet im Browser; es gibt keinen Server, keine Installation und keine Aufzeichnung. Wer sie ohne Netz benutzen will, lädt `PMSM_Rechner_2.7.html` herunter und öffnet sie im Browser — das ist dieselbe Datei.

Zeichnung → FEM → an den Klemmen messbare Größen → eigene Messwerte einsetzen → DGL-Größen berechnen und mit FEMM vergleichen. Eigener Feldlöser und eigene Vernetzung im Browser, drehender Rotor, Messblatt mit Erwartungswerten in jedem Feld.

Alle Zahlen sind gerechnete Zahlen; am gebauten Motor ist noch nicht gemessen worden. Das Messblatt in Stufe 2 nimmt die Ablesungen entgegen und rechnet dieselben Größen daraus.

## Dateien

| Datei | Inhalt |
|:--|:--|
| [`PMSM_Rechner_2.7.html`](PMSM_Rechner_2.7.html) | die Seite selbst (Fassung 2.7 · 28. September 2026 · Fassung 2.7) |
| [`MANUSKRIPT_PMSM_Induktivitaeten_2.7.pdf`](MANUSKRIPT_PMSM_Induktivitaeten_2.7.pdf) | Manuskript: Herleitung vom Feld bis zu den Differentialgleichungen, Messvorschrift, Herkunft der differentiellen Induktivitätsmatrix aus den drei Strangspannungsgleichungen |
| [`PRUEFPLAN.md`](PRUEFPLAN.md) | Prüfplan: jedes Kriterium mit Schranke, Prüfmittel und gemessenem Wert |
| `index.html` | leitet auf die Seite weiter, damit GitHub Pages sie unter der Adresse oben zeigt |

## Lizenz

MIT, siehe [LICENSE](LICENSE) — mit einer Ausnahme für den Vernetzer Triangle, die dort beschrieben ist.
