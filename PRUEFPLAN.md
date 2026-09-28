# Prüfplan für den PMSM-Rechner

Prof. Dr.-Ing. Ralph Wystup M.Sc. — erstellt mit KI und Agent (Claude Code, Anthropic)

Fassung 2.7 · 28. September 2026

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

## M · Das Messblatt: von den Ausschlägen zu den Modellgrößen

Nachgetragen am 27.09.2026. Bis dahin konnte die Seite eine Messung nur als **fertige Datei**
übernehmen — mit Induktivitäten in Henry, also mit Zahlen, die jemand vorher ausgerechnet haben
musste. Genau dort liegt aber der Fehler, den niemand sieht: die Umrechnung von dem, was das Gerät
anzeigt, auf das, was das Modell braucht. Das Messblatt nimmt die **Ausschläge** entgegen und rechnet
selbst.

Damit prüft sich das Verfahren selbst. Aus den Größen der Feldrechnung folgt, was das Messgerät in
jedem Schritt anzeigen muss; trägt man genau diese Ausschläge ein, müssen dieselben Größen wieder
herauskommen. Vorhersage und Auswertung sind dieselbe Rechnung von zwei Seiten, und ein Fehler in
einer der beiden Richtungen fällt sofort auf. Die Probe braucht weder Motor noch Messgerät und läuft
bei jedem Bau mit.

Was sie **nicht** prüft, und das gehört dazu: ob sich die Maschine so verhält. Geprüft wird die
Auswertung, nicht die Physik.

| Nr. | Kriterium | Maßstab | Prüfmittel |
|:--|:--|:--|:--|
| M1 | **Der Rückweg trifft die Ausgangsgrößen.** Die vorhergesagten Ausschläge eingetragen, ergibt die Auswertung $\Psi_\mathrm{PM}$, $L_0$, $M_0$, $L_2$, $L_d$, $L_q$, $l_{dd}$, $l_{qq}$, $l_{dq}$ und beide Sekanten wieder — alle elf Größen. Geprüft auf dem Gitter der Feldrechnung (60 Stellungen je elektrischer Periode); auf gröberen Gittern wird der Verlust ausgewiesen statt verschwiegen. | Rechengenauigkeit, $10^{-9}$ relativ | `pruefe_messblatt.js`, `pruefe_messblatt_seite.js` |
| M2 | **Jeder Schritt erklärt sich, und zwar viererlei:** Ziel, **was eingeprägt wird**, **was gemessen wird**, und die Handgriffe. Die Trennung ist keine Förmlichkeit — $i_d$ und $i_q$ lassen sich an keiner Klemme einstellen, und eine Induktivität zeigt kein Gerät an. | je Schritt alle vier, Ziel ≥ 40 Zeichen | `pruefe_messblatt.js`, `pruefe_messblatt_seite.js` |
| M3 | **Eingetragen wird nur, was ein Gerät anzeigt oder eine Quelle einprägt.** In den Feldern stehen Spannungen, Ströme, Phasenwinkel, Drehzahlen und Winkel — in **Strangmaßen**, nicht in $dq$. Keine Induktivität, keine Flussverkettung, kein $i_d$. | Feldliste je Schritt | `pruefe_messblatt.js` |
| M4 | **Die lageabhängigen Messungen stehen als Tafel über dem Rotorwinkel**, nicht als Einzelwert: die Leerlaufspannung (Schritt 2) und der Strangversuch (Schritt 4). Aus ihnen folgen die Kennwerte durch Fourieranalyse über den Winkel, mit denselben Formeln, die die Kette auf die gerechneten Verläufe anwendet. | zwei Tafeln, volle elektrische Periode | `pruefe_messblatt_seite.js` |
| M5 | **Scheitel- oder Effektivwert wird gefragt, nicht angenommen.** Für die Induktivität kürzt sich der Faktor heraus, solange Spannung und Strom im selben Maß stehen; gemischt eingetragen liegt $L$ um 41 % daneben, ohne jedes Anzeichen. Die Vorhersage der Simulation folgt der Wahl — wer auf Effektivwerte stellt, sieht, was ein Effektivwertmessgerät anzeigen würde. | beide Maße gleiches Ergebnis; gemischt nachweislich falsch | `pruefe_messblatt.js`, `pruefe_messblatt_seite.js` |
| M6 | **Das leere Feld sagt, was zu erwarten ist.** In jedem Eingabefeld steht blass der Wert, den die Feldrechnung vorhersagt. Wer misst, erkennt den Fehlgriff sofort und nicht erst beim Auswerten. Das ist dieselbe Regel wie bei den gesperrten Knöpfen: ein Feld muss sagen, was es will. | jedes Feld mit Erwartung | `pruefe_messblatt_seite.js` |
| M7 | **Das Blatt meldet seine eigenen Widersprüche.** $L_2$ aus drei Kanälen, $l_{dq}$ gegen $l_{qd}$, der Rest über der doppelten Welle, das Nullsystem, die Summe der drei Gleichströme, die Schieflage der Anregung, die Widerstandsdrift durch Erwärmung, die Lage der d-Achse, die Streuung der drei Strangwiderstände: jede Probe steht im Blatt und sagt im Klartext, was ein Befund bedeuten würde. | jeder Schritt ≥ 1 Probe | `pruefe_messblatt.js` |
| M8 | **Die Übernahme stellt die ganze Seite um — und ist umkehrbar.** Nach „diese Messung übernehmen" rechnen alle Reiter mit den gemessenen Zahlen, der Schalter oben steht auf „Messung"; „zurück zur Simulation" stellt den gerechneten Stand wieder her, ohne das Blatt zu leeren. | Quelle wechselt, $D$ trägt die Messwerte | `pruefe_messblatt_seite.js` |
| M9 | **Der Weg über die Feldrechnung bleibt.** Das Messblatt tritt neben die Simulation, nicht an ihre Stelle. Beide stehen in Schritt 7 nebeneinander, mit dem Unterschied in Prozent. | Abgleichtafel mit beiden Spalten | `pruefe_messblatt_seite.js` |
| M10 | **Die Sekante $L_d$ bezieht sich auf $\Psi_\mathrm{PM}$ im Arbeitspunkt, nicht auf den Leerlaufwert.** Wegen der Kreuzsättigung sind das zwei verschiedene Zahlen; deshalb verlangt Schritt 6 einen zweiten Betriebspunkt mit $i_d = 0$ bei unverändertem $i_q$. Mit dem Leerlaufwert gerechnet käme $L_d$ bei dieser Maschine um zwei Prozent falsch heraus. | beide Wege werden ausgewiesen | `pruefe_messblatt.js` |
| M11 | **Was ein gröberes Gitter kostet, steht da.** Bei 24 Stellungen je Periode — der Zahl aus der Vorschrift — bleibt $L_0$ auf 0,005 %, $L_2$ auf 0,31 % und $\Psi_\mathrm{PM}$ auf 0,53 % genau; bei 12 Stellungen wird $\Psi_\mathrm{PM}$ um 2,7 % falsch. Diese Zahlen werden gerechnet, nicht geschätzt. | drei Gitter durchgerechnet | `pruefe_messblatt.js` |
| M12 | **Jeder Schritt trägt sein Messprotokoll.** Vor den Eingabefeldern steht, was einzustellen ist: Drehzahl, Rotorstellungen mit Schrittweite, Gleich- und Wechselanteil der Ströme, Frequenz, und bei den dq-Arbeitspunkten die **drei Strangströme**, die dafür einzuprägen sind. Jede Zeile mit Einheit. | je Schritt eine Vorgabetafel | `pruefe_messblatt.js`, `pruefe_messblatt_seite.js` |
| M13 | **Klartext und Einheiten, überall.** Spaltenköpfe heißen „Spannung Strang 1 gegen Sternpunkt", nicht „u₁ − u_N"; jede Zahl im Blatt trägt ihre Einheit und ein deutsches Dezimalkomma. Ein Blatt, das „0.01 %" neben „0,5 A" stellt, wird beim Ablesen zur Fehlerquelle. | kein Kürzel ohne Klartext, kein Dezimalpunkt | `pruefe_messblatt.js` |
| M14 | **Keine heimliche Quelle.** Das Blatt wird gefüllt, dann wird **jede** Zahl der gespeicherten Simulation auf NaN gesetzt und neu gerechnet. Die Ergebnisse des Blattes müssen Ziffer für Ziffer gleich bleiben — dann stammen sie allein aus den eingetragenen Werten. Die Vergleichsspalte daneben fällt dabei aus, und zwar sichtbar: der eigene Wert bleibt trotzdem stehen. | 3727 Zahlen zerstört, Ergebnis unverändert | `pruefe_messblatt_seite.js` |
| M15 | **Derselbe Weg mit der eigenen Feldrechnung.** Die Tafel lässt sich auch aus dem im Browser gerechneten Verlauf füllen. Die daraus über die Ablesungen gewonnenen $L_0$, $M_0$, $L_2$ müssen die Werte treffen, die dieselbe Rechnung unmittelbar aus dem Feld angibt. Damit ist belegt, dass der Umweg über Betrag und Phase nichts verfälscht — und dass die Tafelwerte wirklich aus einer Feldrechnung stammen können. | Rechengenauigkeit, $10^{-9}$ relativ | `pruefe_messblatt_seite.js` |
| M16 | **Nur das, was sich einstellen oder messen lässt.** Einstellbar sind Rotorwinkel, Drehzahl, Frequenz und ein eingeprägter Strom; messbar sind Strom, Spannung, der Phasenwinkel zwischen beiden, ein Winkel und die Temperatur. Jedes Feld des Blattes trägt eine dieser sieben Arten — kein Feld verlangt eine Induktivität, eine Flussverkettung, einen Widerstand oder ein $i_d$. Auch $R$ wird nicht eingetragen, sondern aus eingeprägtem Strom und gemessener Spannung gebildet; auch der Winkel der Spannung zur d-Achse wird nicht eingetragen, sondern aus dem eingeprägten Stromwinkel und dem gemessenen Phasenwinkel als $\delta = \gamma + \varphi$ gebildet. | Art je Feld, sieben zulässige | `pruefe_messblatt.js` |
| M17 | **Beim Öffnen ist jedes Feld leer.** Nichts ist vorbelegt, nichts gespeichert — eine Seite, die beim Start Messwerte zeigt, die niemand gemessen hat, ist gefährlicher als eine leere. Die Erwartung aus der Feldrechnung steht als Platzhalter mit vorangestelltem **≈** und ist damit unverwechselbar keine Eingabe. | null gefüllte Felder, Platzhalter beginnt mit ≈ | `pruefe_messblatt_seite.js` |
| M18 | **Eintragungen lassen sich sichern und auf Knopfdruck zurückholen.** Gesichert wird ausdrücklich, in den Browser und als Datei. Nach einem Neustart sind die Felder wieder **leer**; der Ladeknopf nennt den Zeitpunkt der Sicherung und stellt die Zahlen wieder her. Geladen werden die **Eintragungen**, nicht die Ergebnisse. | leer nach Neustart, vollständig nach einem Knopfdruck | `pruefe_messblatt_seite.js` |

---

## N · Die Seite als Werkzeug

Nachgetragen am 27.09.2026 auf eine Vorgabe hin, die sich im Satz zusammenfassen
lässt: **die Datei ist ein Werkzeug, kein Text.** Sie bestimmt eine Maschine auf
zwei Wegen — aus der Geometrie gerechnet und am Prüfstand gemessen — und hält
beide gegeneinander. Daraus folgt für jeden Abschnitt eine Frage, die sich
beantworten lassen muss: *rechnet er, oder misst er?* Ist beides nein, gehört er
ins Manuskript.

Drei Verstöße gegen diese Regel hatte die Seite, und alle drei sind der Grund für
die folgenden Kriterien: drei Bilder, die sich nie änderten; ein Vergleich, der
den Reglern nicht folgte; und ein roter Faden, der nach der Feldrechnung abbrach.

| Nr. | Kriterium | Maßstab | Prüfmittel |
|:--|:--|:--|:--|
| N1 | **Von der Zeichnung zum Feld, dann messen, dann vergleichen.** Die Gliederung heißt Start · 1 Zeichnung und Feldrechnung · 2 Messung am Motor · 3 Vergleich · Prüfplan · Manuskript. Modell und Feldrechnung sind **eine** Stufe: aus der technischen Zeichnung wird gerechnet, das ist derselbe Vorgang. Die Feldrechnung kommt immer zuerst — sie liefert nicht nur die Kennwerte, sondern auch die Erwartungswerte für jedes Feld des Messblatts. | sechs Reiter, Namen wörtlich | `pruefe_werkzeug.js` |
| N2 | **Die Startseite trägt den roten Faden.** Drei Stufen, jede mit *hinein*, *heraus*, *geprüft* und einem Knopf dorthin. Die Begründung steht aufklappbar daneben — ein Laboringenieur muss die Seite bedienen können, ohne sie zu lesen, und sie verstehen können, wenn er will. | je Stufe drei Zeilen, ≥ 200 Zeichen Tiefe, ein Knopf | `pruefe_werkzeug.js` |
| N3 | **Keine Zeichenfläche, die nur ein festes Bild zeigt.** Jede Darstellung hängt am Modell, am Arbeitspunkt oder an der eigenen Rechnung. Die drei Bilder, die immer dieselben gespeicherten Kurven zeigten (`c_verlauf`, `c_strang`, `c_leerlauf`), sind entfernt; ihr Inhalt steht im Manuskript. | Liste der Zeichenflächen wörtlich | `pruefe_werkzeug.js` |
| N4 | **Der Vergleich hält drei Wege nebeneinander:** über die Stranggrößen (am Motor nachmessbar), unmittelbar aus dem Feld (Gegenprobe der Rechnung), aus dem Messblatt (Aussage des Versuchs) — mit dem Abstand zwischen ihnen. | sechs Spalten, neun Größen | `pruefe_werkzeug.js` |
| N5 | **Bei kleinen Strömen fallen die beiden Rechenwege zusammen.** Der Weg über $L_0$, $M_0$, $L_2$ und der unmittelbare Weg über Sekante und Steigung rechnen dasselbe Feld. Bei $i_d = -1$ A, $i_q = 1$ A muss der Abstand unter einem Prozent bleiben; was dort steht, ist Rechenfehler und nichts sonst. Bei großen Strömen ist der Abstand dagegen **Sättigung** und gerade der physikalische Gehalt — die Seite sagt, welcher Fall vorliegt. | < 1 % bei $i_d = -1$, $i_q = 1$ | `pruefe_werkzeug.js` |
| N6 | **Der Arbeitspunkt ist in Stufe 3 keine freie Variable.** Ein Parametersatz gilt für den Punkt, an dem er gerechnet oder gemessen wurde; die Ströme daneben frei zu verschieben hieße, Parameter des einen Punktes mit Strömen eines anderen zu verrechnen — das führt das System ad absurdum. Der Punkt wird genau **einmal** gesetzt: in Stufe 1 beim Rechnen oder in Stufe 2 beim Messen. Stufe 3 zeigt ihn an, mit Herkunft, und vermerkt, dass die Induktivitäten bei linearem Eisen konstant sind. Frei bleiben nur n (eine magnetostatische Rechnung kennt keine Zeit) und R (kein Ergebnis der Feldrechnung, sondern Eingabe aus Schritt 1 der Messung). | keine Stromregler in Stufe 3; Anzeige folgt Stufe 1 bzw. Stufe 2 | `pruefe_werkzeug.js`, `pruefe_seite.js` |
| N7 | **Die Gleichungen stehen am Ende.** Der letzte Abschnitt von Stufe 3 sind die Spannungsgleichungen im Rotorsystem und das Moment, mit Zahlen, dazu der eingeschwungene Fall. Darauf läuft die ganze Kette hinaus. | letzter Abschnitt, Zahlen vorhanden | `pruefe_werkzeug.js` |
| N8 | **Umschalten geht in beide Richtungen**, jederzeit, solange beide Datensätze vorliegen — Simulation ↔ Messung, über die Kopfzeile oder über die Knöpfe im Vergleich. Danach steht die Messspalte vollständig. | sim → mess → sim, neun Größen in der Messspalte | `pruefe_werkzeug.js` |
| N10 | **Das Werkzeug steht vor dem Text.** Der Rechenknopf der Feldrechnung und das erste Eingabefeld für Messwerte müssen binnen zwei Fensterhöhen erreichbar sein. Wer ein Werkzeug hinter seine Begründung stellt, hat es versteckt — am 28.09.2026 lag der Rechenknopf bei 2307 px und das erste Messfeld bei 10 039 px, hinter acht Bildschirmen Messvorschrift. Die Begründung bleibt vollständig, aber aufklappbar. | beides unter 2 × Fensterhöhe | `pruefe_werkzeug.js` |
| N9 | **Jede Fassung sagt, was sie geändert hat.** Auf der Startseite steht in wenigen Zeilen das Wesentliche jeder Fassung — nicht die Liste aller Änderungen, sondern das, was ein Leser wissen muss, der die vorige Fassung kennt. | Tafel vorhanden, je Fassung eine Zeile | Sichtprüfung |

---

## P · Die Herleitung im Manuskript

Nachgetragen am 27.09.2026 auf die Beanstandung hin, die differentielle
Induktivitätsmatrix falle im Manuskript vom Himmel. Sie kann nur aus den **drei
statorfesten Strangspannungsgleichungen** durch Transformation kommen, und genau
dieser Weg muss dastehen — ohne übersprungene Zwischenstufe.

Der Unterschied zu den übrigen Abschnitten: hier wird ein **Text** geprüft. Zwei
Dinge lassen sich daran maschinell halten. Erstens die Vollständigkeit: welche
Schritte da sein müssen und welche Begriffe fallen. Zweitens — und das ist der
eigentliche Gewinn — die **Rechenaussagen** des Kapitels. Eine Herleitung, deren
Zwischenergebnisse nachgerechnet werden, ist keine Behauptung mehr.

| Nr. | Kriterium | Maßstab | Prüfmittel |
|:--|:--|:--|:--|
| P1 | **Der Ausgangspunkt ist das Dreiphasensystem.** Das Kapitel beginnt beim Induktionsgesetz am einzelnen Strang, $u_k = R\,i_k + \mathrm d\psi_k/\mathrm dt$, mit $\underline\psi_{123}(\underline i_{123}, \vartheta)$. Nichts wird vorausgesetzt, was nicht dort steht. | Formel und Text vorhanden | `pruefe_herleitung.js` |
| P2 | **Die Matrix entsteht im Strangsystem, nicht erst in $dq$.** Die Kettenregel trennt Transformator- und Bewegungsspannung; der Stromterm enthält die Jacobi-Matrix $\underline{\underline\ell}_{123} = \partial\underline\psi_{123}/\partial\underline i_{123}$. Die Transformation erzeugt die Matrix nicht, sie rechnet sie um. | beide Begriffe genannt, Schritte 0 und 1 vorhanden | `pruefe_herleitung.js` |
| P3 | **Die Transformation ist umkehrbar.** $\underline{\underline T}\,\underline{\underline T}^{-1} = \underline{\underline E}$, geprüft an 24 Winkeln. | $10^{-12}$ | `pruefe_herleitung.js` |
| P4 | **Der Drehterm folgt aus der Ableitung der Transformationsmatrix**, nicht aus einer Annahme: $\partial_\vartheta\underline{\underline T}\cdot\underline{\underline T}^{-1} = \begin{pmatrix}0&1\\-1&0\end{pmatrix}$, und daraus $-\omega_\mathrm{el}\Psi_q$ beziehungsweise $+\omega_\mathrm{el}\Psi_d$. | $10^{-12}$, an 24 Winkeln | `pruefe_herleitung.js` |
| P5 | **Die $dq$-Matrix ist die ähnlichkeitstransformierte Strangmatrix:** $\underline{\underline l} = \underline{\underline T}\,\underline{\underline\ell}_{123}(\vartheta)\,\underline{\underline T}^{-1} = \mathrm{diag}(L_d, L_q)$ mit den $L_0$, $M_0$, $L_2$ dieser Maschine — und zwar **winkelunabhängig über eine volle Periode**, nicht nur an einer Stelle. Die Nebendiagonale bleibt ungesättigt null; das ist der Beleg, dass $l_{dq}$ ausschließlich von der Sättigung lebt. | $10^{-12}$ über 24 Winkel | `pruefe_herleitung.js` |
| P6 | **Kein Schritt fehlt.** Die acht Schritte von den drei Strängen bis zum Bezug von $\Psi_\mathrm{PM}(i_q)$ stehen vollständig da, dazu die Begriffe Koenergie und Satz von Schwarz für die Symmetrie. | acht Überschriften, sechs Begriffe | `pruefe_herleitung.js` |
| P8 | **Jede Größe trägt ihre Argumente.** $\Psi_1(i_d, i_q, \vartheta_r)$ hängt von beiden Strömen und vom Winkel ab, $\Psi_d(i_d, i_q)$ nur noch von den Strömen — und dass der Winkel auf der linken Seite fehlt, ist die Aussage der Transformation und kein Versehen. Eine Flussverkettung ohne Argumente sieht aus wie eine Konstante; in einer Herleitung, die von der Abhängigkeit beider Ströme lebt, ist das Schlamperei. Geprüft in Glied 4 und Glied 5. | neun Fundstellen | `pruefe_herleitung.js` |
| P7 | **Das Kapitel nennt seine eigene Probe.** Im Text steht, mit welchem Mittel seine Rechenaussagen nachgerechnet werden — sonst bleibt „nachrechenbar" eine Behauptung über eine Behauptung. | Prüfmittel im Text genannt | `pruefe_herleitung.js` |
| P9 | **Jede Festlegung im ganzen Manuskript trägt ihre Argumente — oder sagt, warum nicht.** Alle abgesetzten Formeln werden durchgesehen; wo eine arbeitspunktabhängige Größe festgelegt wird, muss sie ihre Argumente tragen. Die argumentfreie Form ist zulässig, wo sie richtig ist — im ungesättigten Fall sind diese Größen Konstanten —, und dort, wo der Verfasser sie ausdrücklich mit `<!-- ARGUMENTFREI: … -->` begründet. Alles andere erscheint am Ende des Laufs als **Liste zum Durchsehen**, nicht als Abbruch: die Entscheidung bleibt beim Verfasser, die Suche nicht. | 22 Festlegungen, null offen | `pruefe_argumente.js` |
| P10 | **Ein Unbefangener muss die Rechnung nachvollziehen können.** Wo links eine Größe ohne Argument steht und rechts eines vorkommt, muss im Text stehen, warum — sonst bricht der Leser genau dort ab. Geprüft wird das an der schärfsten Stelle: in Schritt 4 steht $\vartheta$ rechts und nicht links, und der Text sagt, dass genau das die zu beweisende Behauptung ist. | Begründung im Text, Zahlenprobe im Kapitel P1–P8 | `pruefe_argumente.js`, `pruefe_herleitung.js` |

---

## Q · Die beiden Grundsätze

Nachgetragen am 27.09.2026. Beide gelten für alle Projekte dieser Werkstatt und
stehen als solche in `/workspace/GRUNDSAETZE.md`; hier stehen sie als prüfbare
Kriterien dieses Projekts.

**Der erste:** *Alles, was geschrieben steht, muss man mit Hand am Arm auf dem
Papier nachrechnen können.* Der Rechner ist darin nicht besser als ein Mensch,
nur schneller — er führt denselben Algorithmus aus. Ein Verfahren, das sich
nicht von Hand ausführen lässt, ist entweder nicht verstanden oder nicht richtig
aufgeschrieben.

**Der zweite:** *Zu jedem Ergebnis führen mindestens zwei unabhängige Wege.* Ein
einzelner Weg liefert eine Zahl, aber keine Aussage. Bei der Feldrechnung sind
es hier vier, und **FEMM ist einer davon und nicht die Wahrheit**.

| Nr. | Kriterium | Maßstab | Prüfmittel |
|:--|:--|:--|:--|
| Q0 | **Nie geraten, nie spekuliert — gerechnet oder gemessen.** Manuskript, Prüfplan und Messblatt werden nach Wendungen durchsucht, die eine Vermutung an die Stelle einer Zahl setzen. „vermutlich", „schätzungsweise", „dürfte", „erfahrungsgemäß", „gefühlt" sind harte Befunde. Weiche Wendungen („etwa", „grob", „angenommen") sind zulässig, wenn die gerechnete Zahl daneben steht oder die Stelle das Schätzen ausdrücklich verwirft; sonst erscheinen sie als Liste zum Durchsehen. **Auch Diagnosen** dürfen nicht raten: statt „der Winkelbezug stimmt vermutlich nicht" gehört dorthin, was zu prüfen ist und in welcher Reihenfolge. | null harte Befunde | `pruefe_spekulation.js` |
| Q1 | **Jedes numerische Verfahren steht als Handalgorithmus da.** Newton-Schleife, konjugierte Gradienten mit Jacobi-Vorkonditionierung, Fourieranalyse über dem Winkel, spektrale Ableitung, Differenzenquotient, bilineare Interpolation — je mit Schritten, Startwerten und Abbruchbedingung. | sieben Abschnitte, ≥ 4000 Zeichen | `pruefe_handrechnung.js` |
| Q2 | **Kein Verfahren ohne Abbruchbedingung und Schrittzahl.** „Iteriert bis zur Konvergenz" ist keine Aussage: es gehören die Schranke und die Größenordnung der wirklich gebrauchten Schritte hin ($10^{-7}$ und zehn bis fünfzehn beim Newton, $10^{-6}$ und höchstens 2000 bei den konjugierten Gradienten). | vier Zahlenangaben im Text | `pruefe_handrechnung.js` |
| Q3 | **Das Kleinstbeispiel wird nachgerechnet, nicht abgeschrieben.** Die von Hand durchgerechnete CG-Tafel im Manuskript wird mit demselben Verfahren wie in `fem.js` neu gerechnet; jede Zahl muss im Text stehen, und nach zwei Schritten muss bei zwei Unbekannten die exakte Lösung dastehen. | vier Zahlen wörtlich, Restfehler $10^{-12}$ | `pruefe_handrechnung.js` |
| Q4 | **Übernommene Bausteine sind benannt, samt Verfahren.** Übernommen ist genau einer — die Vernetzung (Triangle, Delaunay mit Verfeinerung nach Ruppert, exakte Prädikate). Ein Baustein, dessen Verfahren niemand angeben kann, ist kein Werkzeug, sondern ein Orakel. | Algorithmus im Text genannt | `pruefe_handrechnung.js` |
| Q5 | **Die vier Wege zur Feldrechnung stehen da — und es gibt sie.** FEMM 4.2, der Löser in Python, der Löser in JavaScript, die geschlossenen Gesetze ohne jeden Löser. Die zugehörigen Dateien müssen vorhanden sein; ein Kapitel, das Wege nennt, die es nicht gibt, wäre schlimmer als keines. | vier Wege im Text, vier Dateien auf der Platte | `pruefe_handrechnung.js` |
| Q6 | **FEMM ist ausdrücklich Prüfmethode und nicht Maßstab.** Der Satz steht im Manuskript, und der vierte Weg — Überlagerung und Reziprozität — braucht überhaupt keinen Löser und fällt auch dann auf, wenn alle Löser denselben Fehler hätten. | Satz vorhanden, beide Gesetze genannt | `pruefe_handrechnung.js`, `pruefe_loeser_gesetze.js` |
| Q7 | **Auch darunter zwei Wege.** Geometrie über Modelldatei und DXF; $L_d$, $L_q$ über die Stranggrößen und unmittelbar aus dem Feld; $\Psi_\mathrm{PM}$ als Grundwelle und aus der Leerlaufspannung; $L_2$ aus drei Kanälen; $l_{dq}$ aus zwei Anregungen; das Messblatt über Vorhersage und Rückweg; das Moment aus der Gleichung und aus seinen beiden Anteilen. Wo es **keinen** zweiten Weg gibt (Strangwiderstand, Vernetzung), steht das ausdrücklich da. | sieben Zeilen in der Tafel | `pruefe_handrechnung.js` |
| Q8 | **Die Namensnennung steht auf jedem ausgelieferten Stück.** `AUSFUHR/VORGABEN.md` verlangt wörtlich: *Prof. Dr.-Ing. Ralph Wystup M.Sc. — erstellt mit KI und Agent (Claude Code, Anthropic)*. Sie steht im Kopf der Seite, im Autorfeld des Manuskripts und im Kopf des Prüfplans — und wie die Fassungsnummer an **einer** Stelle im Bauskript (`NAMENSNENNUNG` in `setze_zahlen.py`), von wo sie überall hingesetzt wird. | drei Stücke, ein Wortlaut | `pruefe_gleichstand.py` |

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
