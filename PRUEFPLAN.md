# Prüfplan für den PMSM-Rechner

Prof. Dr.-Ing. Ralph Wystup · Fassung 2.2 · 26. September 2026

Dieses Blatt sagt, woran das Programm gemessen wird. Es ist bewusst vor dem Prüfen geschrieben und
nicht danach: Kriterien, die man sich nach dem Ergebnis ausdenkt, bestehen immer.

Jedes Kriterium hat eine Nummer, eine **Schranke** und ein **Prüfmittel**. Ein Kriterium ohne
Prüfmittel ist eine Absichtserklärung und wird als solche gekennzeichnet — lieber eine ehrliche
Lücke als ein grüner Haken ohne Deckung.

Ein Grundsatz durchzieht alles: **geprüft wird gegen etwas Unabhängiges.** Eine Rechnung, die mit
sich selbst verglichen wird, besteht jede Prüfung. Unabhängig sind hier: FEMM (fremder Löser),
geschlossene Formeln (Park, Reziprozität, Überlagerung), und das, was auf dem Bildschirm steht
(gegen das, was intern gerechnet wurde).

---

## A · Das Modell am Eingang

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| A1 | Die Geometriedatei ist vollständig und für sich lesbar: Punkte, Strecken, Bögen, Blockmarken, Werkstoffe, Windungszahlen; jeder Bogen hat gültige Endpunkte und Winkel, jede Blockmarke einen vorhandenen Werkstoff. | alle 191 Punkte, 120 Strecken, 113 Bögen, 50 Marken vorhanden | `pruefe_netz.js` |
| A2 | Die Seite lädt die Geometrie **beim Start aus der Datei** und nicht aus eingebauten Zahlen; eine andere Datei ist ladbar. | Ladeweg vorhanden, eine unbrauchbare Datei lässt die geladene unberührt | `pruefe_anzeige.js` |
| A3 | Die Eisenkennlinie ist monoton und differenzierbar; µ_r fällt mit B, wird nie negativ. | B(H) streng monoton, differentielles µ_r überall positiv, Sättigung erfasst | `pruefe_netz.js` |
| A4 | Das Netz ist gültig: kein Dreieck mit Fläche null oder negativer Orientierung, Summe der Dreiecksflächen = Modellfläche. | Abweichung < 0,01 % | `pruefe_netz.js` |
| A5 | Bei der Drehung drehen **drei** Dinge mit: die Rotorgeometrie, die Magnetisierungsrichtung der Magnete und die Strangströme. | $\Psi_\mathrm{PM}$ über eine Periode konstant auf < 2 % | `pruefe_drehung.js` |
| A6 | Die Strangströme bilden über eine **elektrische** Umdrehung ein sauberes Drehstromsystem: reine Sinusform ohne Oberwelle und Gleichanteil, gleiche Amplitude √($i_d$²+$i_q$²), genau ∓120° Versatz, Summe null. Eine elektrische Umdrehung sind bei p = 3 genau 120° mechanisch. | < 1e-9 A, Versatz auf 1e-6 Grad | `pruefe_stroeme.js` |
| A7 | **Der Arbeitspunkt steht im rotorfesten System still.** $i_d$ und $i_q$ sind dort vorgegeben und bleiben konstant; was sich dreht, ist das Koordinatensystem. An jeder gerechneten Stellung muss die Rücktransformation wieder genau dieselben $i_d$ und $i_q$ liefern — sonst wanderte der Arbeitspunkt während der Drehung, jede Stellung gehörte zu einem anderen Sättigungszustand, und das Mittel wäre über Ungleiches gebildet. | < 1e-12 A an jeder Stellung | `pruefe_stroeme.js` |

## B · Der Feldlöser für sich

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| B1 | Newton konvergiert; das Residuum fällt monoton und erreicht die Schranke. | Restfehler < 1e-7, ≤ 14 Schritte, Schritt fällt monoton | `pruefe_loeser_gesetze.js` |
| B2 | Der lineare Fall stimmt mit FEMMs eigenem linearem Lauf überein — dasselbe Modell, aber ein künstlicher Werkstoff mit festem µ_r = 3678 statt der Kennlinie. Dort gibt es keine Kennlinieninterpolation, an der sich zwei Löser unterscheiden könnten; was bleibt, ist die Vernetzung allein. | < 1 % auf Ψ₁, Ψ₂, Ψ₃ | `pruefe_loeser_gesetze.js` gegen `pruef_linear.txt` |
| B3 | **Überlagerung** bei linearem Eisen: ψ($i_a$ + $i_b$) = ψ($i_a$) + ψ($i_b$). Das ist eine Eigenschaft der Gleichung, keine Anpassung — verletzt sie der Löser, ist er falsch, egal was FEMM sagt. | < 0,1 % von $\Psi_\mathrm{PM}$ | `pruefe_loeser_gesetze.js` |
| B4 | **Reziprozität**: die Gegeninduktivität ist symmetrisch, M₂₁ = M₁₂, gemessen durch Bestromen des jeweils anderen Stranges. | < 0,5 % von L₀ | `pruefe_loeser_gesetze.js` |
| B5 | Die Lösung läuft mit feinerem Netz monoton auf einen Grenzwert zu, nicht sprunghaft. | Monotonie über drei Netzstufen | `pruefe_einstellungen.js` |

## C · Die messbaren Größen

Diese Größen sind der Angelpunkt der ganzen Kette: sie sind am gebauten Motor elektrisch zugänglich,
und nur deshalb kann eine Messung später an die Stelle der Rechnung treten.

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| C1 | Leerlauf: Ψ₁, Ψ₂, Ψ₃ Stellung für Stellung gegen FEMM — nicht nur im Mittel. | < 2 % von $\Psi_\mathrm{PM}$ | `pruefe_drehung.js` |
| C2 | Leerlauf: $\Psi_\mathrm{PM}$ aus der Grundwelle und die Welligkeit von $\ps$i_d$$. | < 2 % bzw. < 5 % | `pruefe_drehung.js` |
| C3 | Strangversuch: L₁₁, M₂₁, M₃₁ Stellung für Stellung. | < 2 % von L₀ | `pruefe_drehung.js` |
| C4 | Arbeitspunkt: $\ps$i_d$$ und $\ps$i_q$$ bei **unabhängig** verstelltem $i_d$ und $i_q$, nicht nur auf der Geraden $i_q$ = −$i_d$. | < 2,5 % von $\Psi_\mathrm{PM}$ | `pruefe_punkte.js` gegen eigenen FEMM-Lauf |
| C5 | Gegenseitigkeit im Arbeitspunkt: $l_{dq}$ = $l_{qd}$. | < 2 µH | `pruefe_punkte.js`, `pruefe_drehung.js` |

## D · Von den messbaren Größen zu den Kennwerten

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| D1 | Park hin und zurück ist verlustfrei. | < 1e-12 relativ | `pruefe_stroeme.js` |
| D2 | Der Weg über die Stranggrößen und der direkte Weg aus dem Feld liefern dieselben $L_d$, $L_q$. | < 1 % | `pruefe_drehung.js` |
| D3 | Der Ansatz selbst stimmt: $\ps$i_d$$ = $\Psi_\mathrm{PM}$ + $L_d$ $i_d$ und $\ps$i_q$$ = $L_q$ $i_q$ treffen die gerechneten Punkte. | < 0,5 % von $\Psi_\mathrm{PM}$ | `pruefe_drehung.js` |
| D4 | Die Rückrechnung führt keine Zahl auf sich selbst zurück: die Kennwerte stammen aus drei getrennten Versuchen. | Herkunft jeder Zahl benannt | `rueckrechnung.py` |
| D5 | Die Stellungszahl darf kein Vielfaches von 6 sein, sonst fällt die 6. Harmonische auf den Mittelwert. | die Seite bietet nur 5, 10, 20 an; die Prüfskripte brechen sonst ab | `pruefe_anzeige.js` |

## E · Die Seite als Werkzeug

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| E1 | Die Seite lädt ohne Ausnahme; jeder Reiter ist gefüllt, kein leerer Abschnitt. | 0 Ausnahmen, 0 leere Abschnitte | `pruefe_seite.js` |
| E2 | Während der Rechnung ist **sichtbar**, dass gerechnet wird: bewegtes Element, Zähler, Restzeit. | alle drei vorhanden und veränderlich, Balken nie über 100 % | `pruefe_anzeige.js` |
| E3 | Vor einer neuen Rechnung sind alle Felder leer; gefüllt wird erst nach vollständigem Durchlauf. | nichts Halbes sichtbar | `pruefe_seite.js` |
| E4 | Ein abgebrochener Lauf hinterlässt nichts. | Felder wieder leer | `pruefe_seite.js` |
| E5 | Wird eine Einstellung geändert, werden vorhandene eigene Ergebnisse verworfen und der Grund genannt. | Quelle fällt zurück, Meldung sichtbar | `pruefe_seite.js` |
| E6 | Der geführte Lauf führt über alle Stufen bis zum Ende; „weiter" greift nach jeder Stufe und ist während der Rechnung gesperrt. | alle 7 Stufen | `pruefe_seite.js` |
| E7 | **Es wird nie gemischt.** Steht die Quelle auf „eigen", hängt keine angezeigte Zahl mehr an den gespeicherten Werten. | keine Änderung, wenn die gespeicherten Zahlen zerstört werden | `pruefe_kein_mischen.js` |
| E8 | Die angezeigten Gleichungen rechnen mit den Zahlen, die darüber in der Tafel stehen — bei $i_d$ = 0 **und** bei $i_d$ ≠ 0, sonst bleibt der Reluktanzterm ungeprüft. | < 0,02 V bzw. < 0,002 Nm | `pruefe_gleichungen.js` |
| E9 | Verglichen wird nur Gleiches: die Spalte „gegen FEMM" erscheint nur, wenn beide Seiten dasselbe Eisen haben. | Spalte fehlt bei linearem Eisen, erscheint mit Kennlinie wieder | `pruefe_anzeige.js` |
| E10 | Die Induktivitäten in den Gleichungen gehören zu dem Arbeitspunkt, der gerechnet wurde, und die Seite sagt, zu welchem. | Stromangabe mit Zahlen sichtbar | `pruefe_anzeige.js` |

## F · Das Zusammenspiel

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| F1 | Die geprüften Module sind eingefroren; jede Änderung fällt auf. | 6 Prüfsummen unverändert | `friere_ein.py` |
| F2 | Die Seite rechnet dasselbe wie dieselben Module unter Node — bei Eisen nach Kennlinie, also so, wie die Seite normalerweise rechnet. | < 0,1 % | `pruefe_anzeige.js` |
| F3 | Die Rechnung stimmt mit FEMM nicht nur bei einer Einstellung, sondern über Netzfeinheit, Stellungszahl und Arbeitspunkt hinweg. | Voreinstellung < 2 %, sonst < 3 % | `pruefe_einstellungen.js`, `pruefe_punkte.js` |
| F4 | Jede Zahl im Manuskript und in der Seite stammt aus einer Ergebnisdatei, keine ist von Hand getippt. | kein Unterschied | `pruefe_gleichstand.py`, `pruefe_html.py` |
| F5 | Die ganze Kette läuft in einem Zug durch, von der Geometriedatei bis zu den Motorgleichungen. | `rechne_alles.sh` ohne Beanstandung | `rechne_alles.sh` |
| F6 | Alles liegt doppelt: Arbeitsbereich und Ablageordner auf dem Rechner, dazu ein datiertes Archiv mit Prüfsumme. Verglichen wird Datei für Datei mit Größe, nicht nur die Anzahl. | nichts fehlt, keine Größe abweichend | `nach_pc.py` und Gegenzählung mit `dateiliste` |

---

## G · Der Preprozess und der Weg über eine Zeichnung

Neu seit dem 26.09.2026. Hier wird aus einer Zeichnung ein Modell; alles, was dabei schiefgeht,
sieht hinterher aus wie ein Rechenergebnis. Deshalb gilt hier besonders: **nie stillschweigend
weiterrechnen, und jede Meldung nennt die Stelle.**

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| G1 | Die Gebiete werden unabhängig von den Blockmarken gefunden — als Zusammenhangskomponenten der Triangulierung, die keine Zwangskante überschreiten. Sonst ließe sich „dieses Gebiet hat noch keine Marke" gar nicht sagen. | 50 Gebiete für 50 Marken | `pruefe_preprozess.js` |
| G2 | Jede Blockmarke liegt in einem eigenen Gebiet, und die Punktlage trifft: ein Klick landet in dem Gebiet, das der Vernetzer später als Werkstoffbereich behandelt. | 0 Fehltreffer, auch bei gedrehtem Rotor | `pruefe_preprozess.js` |
| G3 | Die Gebietsflächen summieren sich zur Fläche des Rechennetzes — kein Gebiet fehlt, keines zählt doppelt. | < 0,5 % | `pruefe_preprozess.js` |
| G4 | **Linearität ist eine Werkstoffeigenschaft, kein Schalter.** Ein als nichtlinear erklärter Werkstoff ohne B(H)-Tafel wird gemeldet und nicht stillschweigend als lineares Eisen gerechnet. | Meldung statt Annahme | `pruefe_preprozess.js` |
| G5 | DXF wird gelesen, ohne dass etwas verschwindet: unbekannte Objektarten werden **mit Namen** gemeldet, übergangene Objekte einzeln. | 0 stillschweigende Verluste | `pruefe_dxf.js` |
| G6 | Die Geometrie kommt unverändert an: Punktzahl, Lage und eingeschlossene Bogenwinkel. | < 1 nm bzw. < 1e-9 Grad | `pruefe_dxf.js` |
| G7 | Der Streckenzug wird geprüft, nicht repariert: lose Enden, Kreuzungen, T-Stöße und doppelte Kanten werden **mit Ortsangabe** gemeldet. | jeder Befund mit Koordinate | `pruefe_dxf.js` |
| G8 | **Ein Modell ohne Randbedingung wird nicht gerechnet.** DXF trägt keine; fehlt sie, liegen alle Kennwerte um rund ein Prozent daneben, ohne dass etwas scheitert. Der Import schlägt den Außenrand vor, und das Fehlen wird gemeldet. | Vorschlag trifft die 2 Kanten bei r = 100 mm | `pruefe_dxf.js` |
| G9 | Der Weg über DXF liefert dieselbe Maschine: vernetzen, rechnen, und L_d muss die bekannte Zahl treffen. | < 0,1 % | `pruefe_dxf.js` |
| G10 | Die **Auflösung des Verfahrens** ist bekannt: dieselbe Eingabe liefert bitgenau dasselbe; eine Störung in der letzten Koordinatenstelle 0,0009 %, eine andere Punktreihenfolge 0,012 %. Unterhalb davon sagt kein Vergleich etwas aus. | dokumentiert | `pruefe_dxf.js` |

---

## H · Was fehlt, und was wörtlich dasteht

Nachgetragen am 26.09.2026 auf zwei Rückfragen hin: *„warum fehlen auf einmal die
Koppelinduktivitäten?"* und *„u_d, u_q, M — die sind doch auch geändert worden, warum?"*

Beide Fragen treffen eine Lücke im bisherigen Plan. Abschnitt E prüfte, ob die angezeigten
Gleichungen mit den angezeigten Zahlen **rechnerisch** zusammenpassen (E8) — das findet einen
Vorzeichenfehler, aber nicht, dass ein Term umbenannt oder ein Modell stillschweigend gewechselt
wird. Und niemand prüfte, wie eine **fehlende** Zahl aussieht. Das war keine akademische Lücke: in
`mH(x)` stand `z(1000*x)`, und `1000·null` ist `0` — eine fehlende Induktivität erschien deshalb als
„0,000 mH", also als *gemessene Null*. Wer statt eines leeren Feldes eine Null sieht, liest, die
Kopplung sei verschwunden.

Die Regel dahinter gilt für das ganze Programm: **eine Zahl, die es nicht gibt, darf nie wie eine
Zahl aussehen — und das leere Feld muss sagen, warum es leer ist.** Das ist dieselbe Regel, die für
gesperrte Knöpfe schon gilt.

Die zweite Rückfrage führte auf die Gegenregel: **eine Zahl, die es gibt, muss auch dort auftauchen,
wo man sie sucht.** Die Symbole der Gleichungen hingen am Schalter „linear / sättigbar" — im linearen
Modell standen dort $L_d$, $L_q$ statt $l_{dd}$, $l_{qq}$, und die Kreuzterme fehlten ganz, obwohl
$l_{dq}$ gerechnet und in der Tafel ausgewiesen war. Das liest sich wie eine geänderte Gleichung, war
aber nur ein anderer Anzeigefall. Jetzt steht überall dieselbe, allgemeine Form; sie ist in beiden
Fällen richtig und fällt im linearen Modell von selbst auf die gewohnte zurück, weil dort
$l_{dd} = L_d$, $l_{qq} = L_q$ und $l_{dq} = 0$ ist.

| Nr. | Kriterium | Schranke | Prüfmittel |
|:--|:--|:--|:--|
| H1 | Eine fehlende Größe erscheint als Gedankenstrich, nie als Null. Eine wirklich gerechnete Null bleibt eine Null — beides muss unterscheidbar bleiben. | `mH(null)` = „—", `mH(0)` = „0,000" | `pruefe_fehlende_zahlen.js` |
| H2 | Fehlen L₀, M₀ und L₂, weil die eigene Rechnung ohne Versuch 2 lief, steht der Grund daneben samt dem Weg dorthin. Ergänzt werden sie nie aus der gespeicherten Rechnung (das ist E7 von der anderen Seite). | Text nennt Versuch 2, keine erfundene Null | `pruefe_fehlende_zahlen.js` |
| H3 | Die drei Maschinengleichungen stehen **in jedem Modell im selben Wortlaut**, in der allgemeinen Form: $u_d = R i_d + l_{dd}\frac{di_d}{dt} + l_{dq}\frac{di_q}{dt} - \omega_{el} L_q i_q$, $u_q = R i_q + l_{qq}\frac{di_q}{dt} + l_{dq}\frac{di_d}{dt} + \omega_{el}(\Psi_{PM} + L_d i_d)$, $M = \frac{3}{2} p [\Psi_{PM} i_q + (L_d - L_q) i_d i_q]$. Was sich zwischen den Modellen ändert, sind die **Zahlen**, nie der Wortlaut. | Zeichen für Zeichen, beide Modelle gleich | `pruefe_fehlende_zahlen.js` |
| H4 | **Konsistenz: jede gerechnete Induktivität erscheint auch in der Gleichung.** Steht $l_{dq}$ in der Parametertafel, muss es in der Gleichung stehen — sonst sucht man es dort vergebens und hält die Gleichung für geändert. Dass die allgemeine Form im linearen Modell auf die gewohnte zurückfällt, wird nachgerechnet ($l_{dd} = L_d$, $l_{qq} = L_q$, $l_{dq} = 0$) und nicht behauptet; die Zeile unter der Gleichung trennt differentielle von Sekanteninduktivitäten und nennt den Grenzfall. | keine Größe nur in der Tafel | `pruefe_fehlende_zahlen.js` |
| H5 | Das Moment zerfällt genau in Erreger- und Reluktanzanteil, und das Vorzeichen des Reluktanzanteils folgt $(L_d - L_q) i_d i_q$. Diese Maschine ist **invers schenkelig** ($L_q > L_d$), das Vorzeichen ist deshalb umgekehrt zur gewohnten Schenkelpolmaschine. | $M_{pm} + M_{rel} = M$ auf $10^{-12}$ | `pruefe_fehlende_zahlen.js` |
| H6 | **Seite und Manuskript sagen dasselbe.** Der Symbolsatz der $dq$-Spannungsgleichungen ist auf beiden Seiten derselbe ($l_{dd}$, $l_{dq}$, $l_{qd}$, $l_{qq}$, $L_d$, $L_q$, $\Psi_{PM}$, $\omega_{el}$). In keiner Maschinengleichung des Manuskripts steht mehr ein bloßes $\omega$ — das ist dort die Kreisfrequenz der **Wechselstrommessung** und eine andere Größe als $\omega_{el} = p\,\Omega$; der Unterschied wird im Text ausdrücklich genannt. | Symbol für Symbol | `pruefe_fehlende_zahlen.js` |
| H7 | **Kein Befund geht unterwegs verloren.** In der Rechenkette stand hinter fast jedem Prüfmittel ein `| tail -3`; ein Befund acht Zeilen vor dem Ende erschien damit nie auf dem Schirm — genau der Fehlertyp von H1, eine Ebene höher. Die Kette ruft jedes Prüfmittel über eine Helferfunktion auf, die **erst jede Befundzeile** und dann den Abschluss zeigt. | keine Abschneidung in der Kette | `pruefe_fehlende_zahlen.js` |

---

## Das Abnahmeblatt

Der Plan allein genügt nicht. Beim ersten zeilenweisen Durchgang am 26.09.2026 zeigte sich, dass
sieben Kriterien einem Prüfmittel zugeschrieben waren, das sie gar nicht prüft — B1, B2 und D1 kamen
in `pruefe_fem_js.py` überhaupt nicht vor, A1 und A3 waren Skripten zugeschrieben, die bauen
beziehungsweise drucken. **Eine Prüfung, die es nicht gibt, sieht in einer Tabelle genauso aus wie
eine, die besteht.**

Deshalb gibt es `rechnung/abnahme.py`. Es bindet jedes Kriterium an einen Befehl *und* an ein Muster
in dessen Ausgabe, lässt jedes Prüfmittel genau einmal laufen und schreibt zu jeder Nummer die
gemessene Zeile. Findet sich keine passende Zeile, steht dort `UNGEPRÜFT`. Damit lässt sich die
Lücke, die diesen Abschnitt veranlasst hat, nicht wiederholen.

## Was dieser Plan nicht prüft

Damit niemand mehr hineinliest, als dasteht:

* **Die Messung am wirklichen Motor.** Die Vorschrift dafür steht im Manuskript samt Erwartungswerten;
  gemessen wurde noch nicht. Alle Zahlen dieses Programms sind gerechnete Zahlen.
* **FEMM selbst.** FEMM ist hier der Maßstab, nicht der Prüfling. Stimmt FEMM an einer Stelle nicht,
  merkt es dieses Programm nicht — die einzigen Prüfungen, die davon unabhängig sind, sind B3, B4
  und D1, weil dort gegen geschlossene Gesetze geprüft wird.
* **Wirbelströme, Ummagnetisierungsverluste, Temperatur.** Gerechnet wird magnetostatisch.
* **Die Stromregelung eines wirklichen Umrichters.** $i_d$ und $i_q$ werden hier eingestellt, nicht geregelt.
