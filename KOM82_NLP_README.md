# KOM82_NLP – Bundestagsprotokollanalyse (WP 20, Sitzungen 146–212)

## Inhalt
| Datei | Zweck |
|:--|:--|
| `KOM82_NLP_v02.ipynb` | eigenständiges Notebook (alle Klassen + Läufe + Begründungen); v02 = Modelle statt Spezialregeln |
| `KOM82_NLP_v01.ipynb` | Vorgängerversion (regelbasiert) – Referenz für den Vorher/Nachher-Vergleich |
| `requirements.txt` | Abhängigkeiten |

## Einrichtung (Windows, VS Code)
```
pip install -r requirements.txt
ollama pull qwen3:8b
ollama pull qwen3-embedding:0.6b
```
Ordner (wie bisher): `KOM82_NLP/data/pdf/20_146.pdf … 20_212.pdf`, `data/validation/{dip,xml}`, optional `evaluation/protocol_split.json` (eingefrorener Split; fehlt er, wird einmalig ein deterministischer Split erzeugt).

Modelle beim ersten Lauf (Download über Hugging Face): `sat-3l-sm` (SaT), `mrm8488/bert-base-german-finetuned-ler` (Legal-NER), `classla/xlm-r-parlasent` (Sentiment). Zero-Shot nutzt das Ollama-Embeddingmodell.

## v02 gegenüber v01
Reguläre Ausdrücke nur noch für 11 formale Kennungen (`Kennung`); Vokabulare als Daten (`Lexikon`). Inhaltliche Entscheidungen durch Modelle: SaT (Satzgrenzen), Legal-NER (Gesetze, Normnennungen), Zero-Shot über Embeddings (IV-Eintragsart, Debattentyp, Rolle, Beitragsart, Zwischenruf-Art, Formelsätze, Abstimmungspassagen), LLM (Gesetzgebung, Titel-Grundform mit flexionstoleranter Belegprüfung, Abstimmungsergebnis). Zu jedem Modell ein einfaches, bewusst unvollständiges lexikalisches Signal (Kopfwort „Gesetz“ über Snowball-Stämme); keines entscheidet allein, Kap. 14 misst Lexik, NER, LLM und Hybrid getrennt. Ohne jedes Modell findet die v02 Gesetze über das lexikalische Signal. Designentscheidung Zero-Shot statt trainierter Klassifikator: Kap. 5b. Vergleich: v01-Ausgabeordner nach `output_v01/` kopieren – Kap. 14 stellt die Kennzahlen gegenüber.

## Ablauf
1. `MODUS = "dev"`, `SITZUNGEN = [146, 199, 205]` → Funktionsprobe (≈ 1 min ohne LLM-Erstlauf).
2. `MODUS = "dev"`, `SITZUNGEN = None` → Evaluation auf DEV (43 Protokolle); Methoden-Freeze.
3. `EVAL_SPLITS = ("validierung",)`, danach einmalig `("test",)` + `TEST_FREIGABE = True`.
4. Kernel-Neustart, `MODUS = "betrieb"`, `ZIP_ERSTELLEN = True` → Volllauf über alle 67 Protokolle.

RunPod: `RUNPOD_AKTIV = True` (Module `my_utils.runpod_pod` wie bisher; lokale Ollama-Instanz muss laufen, damit Version und Modell-Digests verglichen werden können). Alle LLM-/Embedding-Antworten liegen danach im Cache `output/cache/modelle.sqlite` und sind ohne GPU reproduzierbar.

## Prüfstand dieser Version
Entwickelt und end-to-end ausgeführt auf 20/146, 20/199, 20/205 mit dem pdfplumber-Backend und einem Mock-Ollama (Schnittstellentest, keine fachlichen LLM-Ergebnisse). Stufe 1 (Struktur) ist auf diesen Protokollen mit v01 identisch. Nicht hier geprüft (keine Paketinstallation möglich): SaT, Legal-NER (nur mit Test-Stub), die Zero-Shot-Güte mit echten Embeddings, PyMuPDF, echtes Ollama, ParlaSent und die DIP/XML-Evaluation – auf dem Zielsystem auszuführen; `BACKEND_VERGLEICH = True` vergleicht beide PDF-Backends auf einem Protokoll.
