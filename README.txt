CLEANUPD.R4X
============

CLEANUPD.R4X ist die R4X-Ressourcen-Cleanup-Diagnose. `module.R4MF`
beschreibt Buildprofil, Quellen, Artefakt, Zielpfad, R4L-Imports und Contract.

Build:

    DevTools\Scripts\Build.bat -app CLEANUPD

Ergebnis:

    Code\zig-out\CLEANUPD.R4X

Contract:
- Build-Profil: `Zig/R4XStart`
- R4XStart-Entry: `cleanupd_main`
- App-Klasse: `console`
- R4L-Imports: `R4SYS`, `R4DEV`, `R4DRAW`
- Zielpfad im Image: `C:\R4OS\SOFTWARE\TERMINAL\DIAG\CLEANUPD.R4X`

ProgramInstance-Speicher 0.59.7
--------------------------------

    CLEANUPD /PROGRAMSTORAGE

Der Modus prueft den ressourcenproportionalen ProgramInstance-Speicher:

- normaler Console-Exit und Console-Kill;
- GUI-Kill mit lazy Command-/Rasterpayload;
- Service-Close und Service-Kill ohne Console-/GUI-Payload;
- einen kontrollierten Warm-up fuer wiederverwendete Task-Stack-/Page-Table-
  Infrastruktur mit exakter Owner-/Payload-Rueckkehr und danach 72
  deterministisch synchronisierte Kills waehrend
  `io_wait(WAIT_FOREVER)`: ein vom Parent einmal geoeffneter `SVCSTALL`-
  Handle, belegte Queue, R4DEV-Taskzustand `blocked`/`completion`, Request-
  Cancellation, Client-Reap und vollstaendige Heap-/PMM-/VM-/Ownerbaseline
  nach jedem gemessenen Zyklus;
- 18 gemischte Console-/GUI-/Service-Churnzyklen;
- Rueckkehr der Program-, Heap-, PMM-, VM-, Stack-, Image- und Ownerwerte
  zur warmen Baseline;
- den begrenzten R4DEV-Kernelselftest mit Fehlerinjektion an jeder eager und
  lazy Payloadstufe, Reverse-Rollback, Zaehlerbalance und Nullinitialisierung.

Fuer den GUI-Lauf startet CLEANUPD sich intern mit LaunchPolicy `gui` und dem
Argument `/GUIPAYLOADHOLD`. Diese headless-faehige Fixture erzeugt ueber
R4DRAW ausdruecklich Command- und Rasterpayload und benoetigt weder einen
Desktop-Host noch eine Window-ID.

Der Modus verlangt die optionalen R4DEV-v2-Funktionen
`program_instance_storage_summary` und
`program_instance_storage_self_test`. `ProgramInstanceStorageSummary` meldet
unter anderem Kern-/Payloadgroessen, aktive Klassen/Payloads, aktuelle und
Peak-Reservierung sowie Allocation-, Release-, Fehler- und Rollbackzaehler.
Payloadbytes sind angeforderte Heap-Strukturbytes; Image, Stack und App-VM
werden separat ueber die vorhandenen Ressourcensichten bilanziert.

Ein erfolgreicher Lauf endet eindeutig mit:

    CLEANUPD programstorage killwait cycles=72 blocked=completion queue=OK reap=OK baseline=OK slots=OK
    CLEANUPD programstorage result: OK
    CLEANUPD result: OK

Automatisierte QEMU-Abnahme:

    Tests\Runtime\Run-ProgramInstanceStorageRuntime0597.ps1

Dynamische ProgramRegistry 0.59.8
--------------------------------

    CLEANUPD /PROGRAMREGISTRY

Der Modus startet 24 gleichzeitig laufende, leichte Instanzen von CLEANUPD
mit dem internen Argument `/REGISTRYHOLD` und prueft jede ueber ihren
besitzenden ProcessHandle. Anschliessend fuellt er die bereits allokierten
Registry-Chunks bis zur naechsten Growth-Grenze, armiert ueber den
bootoption-gebundenen R4DEV-Selftest genau einen Growth-OOM und verlangt:

- der neue Spawn scheitert mit diagnostiziertem Admission-Fehler;
- alle bestehenden IDs, stabilen Coreadressen und Handles bleiben intakt;
- die One-shot-Schaltung wird konsumiert beziehungsweise im Cleanup immer
  noch einmal idempotent zurueckgesetzt;
- nach Kill/Wait/Reap einer gehaltenen Instanz funktioniert ein Ersatzspawn;
- abschliessend kehren Live-Set und Ownerressourcen zur Baseline zurueck.

Der automatisierte QEMU-Lauf setzt dafuer temporaer:

    OPTION PROGRAMREGISTRY selftest=yes

Automatisierte QEMU-Abnahme:

    Tests\Runtime\Run-DynamicProgramRegistryRuntime0598.ps1
