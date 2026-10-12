# M-HARP Project — Run Checklist

> Minimum configuration required to get the project running from scratch.
> Project root: `E:\data_compress\Flexible论文代码\Mix-Piece_Sim-Piece-main`

---

## 0. Quick start (30 seconds)

```bash
# One-time environment check
java -version       # expect 1.8+
mvn -version        # expect 3.6+
python --version    # expect 3.8+
python -c "import openpyxl"   # must not error

# Build
cd "E:/data_compress/Flexible论文代码/Mix-Piece_Sim-Piece-main"
mvn compile test-compile -DskipTests   # ⚠️ never run `mvn clean`

# Run tests
mvn test -Dtest=TestPLA
mvn test -Dtest=TestThreadScalingFull

# Run benchmark (generates baseline_multirun.csv)
mvn test -Dtest=BaselineMultiRun

# Regenerate LaTeX
python results/build_manuscript_sections.py
```

---

## 1. Required software

| Software | Min version | Recommended | Verify | Currently installed |
|---|---|---|---|---|
| **JDK** | 8 | 21 (LTS) | `java -version` | ✅ 21.0.6 |
| **Maven** | 3.6 | 3.9 | `mvn -version` | ✅ 3.9.9 |
| **Python** | 3.8 | 3.12 | `python --version` | ✅ 3.12.7 |

> The Java compile target is 1.8 (`<source>1.8</source>` in `pom.xml`), but JDK 21 is fully compatible.

---

## 2. Java dependencies

Declared in `pom.xml`. **Maven auto-downloads to the local repo.**

| Dependency | Version | Required? | Purpose |
|---|---|---|---|
| `com.github.luben:zstd-jni` | 1.5.4-2 | **Required** | Zstd compression (with native lib) |
| `org.apache.poi:poi` + `poi-ooxml` | 5.2.3 | Required | Read/write .xlsx evaluation sheets |
| `org.apache.commons:commons-math3` | 3.6.1 | Required | Math utilities |
| `org.apache.commons:commons-collections4` | 4.4 | Required | Collection extensions |
| `org.junit.jupiter:junit-jupiter` | 5.9.1 (BOM) | Test only | JUnit 5 |
| `org.projectlombok:lombok` | 1.18.34 | Required (provided) | Code generation |
| DL4J / ND4J / DataVec (4 pkgs) | 1.0.0-M2.1 / beta7 | Transitive | Not actually imported in source; can be removed |
| SLF4J + Logback + Log4j2 | 1.7.32 / 1.2.6 / 2.17.1 | Transitive | Logging (no manual config needed) |

### Key native library
- `zstd-jni` 1.5.4-2 ships Windows x64 `ZstdJNI.dll` inside the jar; Maven auto-loads it after download.
- Path: `~/.m2/repository/com/github/luben/zstd-jni/1.5.4-2/zstd-jni-1.5.4-2.jar`

### Maven repository config
- Configured: jitpack.io (`https://jitpack.io`) — fallback, not needed for normal deps
- Default: Maven Central
- Local cache: `C:\Users\czw\.m2\repository` (user) + `D:\Program Files\Maven\maven-repository` (system)

---

## 3. Python dependencies

| Package | Source | Required? | Install |
|---|---|---|---|
| `openpyxl` 3.1.5 | PyPI / Anaconda | **Required** (reads .xlsx) | `pip install openpyxl` or `conda install openpyxl` |
| `csv`, `re`, `pathlib`, `statistics`, `collections`, `gzip`, `io`, `os`, `sys`, `time`, `subprocess` | Stdlib | — | Comes with Python |

> No `requirements.txt` in the project. Anaconda base already provides openpyxl 3.1.5.
> Local modules `LinearCompressor` and `SingleStreamProtocol` appear to be project-internal modules with no corresponding `.py` files; running scripts that import them will raise ImportError.

---

## 4. Datasets (must be in place)

13 `.csv.gz` files in `src/test/resources/` — **shipped with the project, do not delete**:

```
Cricket.csv.gz             Lightning.csv.gz        MoteStrain.csv.gz
Wafer.csv.gz               WindSpeed.csv.gz        WindDirection.csv.gz
FaceFour.csv.gz            Pressure.csv.gz         BTCUSD.csv.gz
ETHUSD.csv.gz              SPX.csv.gz              STOXX50E.csv.gz
city_temperature.csv.gz    (backup) city_temperature-fixed.csv.gz
                          (backup) IoT_all.csv.gz
```

Loading convention: `getClass().getResourceAsStream("/Cricket.csv.gz")` — must live at the classpath root under `src/test/resources/`.

---

## 5. Third-party JARs (already in place, leave alone)

`lib/` directory:
- `JWave.jar` — wavelet transform (backup)
- `fpc-0.0.1-SNAPSHOT.jar` — FPC baseline (backup)

> `pom.xml` does not reference these; normal test runs do not need them.

---

## 6. Build artifacts (already in place, leave alone)

`target/` directory — **never run `mvn clean`**:
- `target/classes/` — 87 .class files (main code)
- `target/test-classes/` — 43 .class files (test code)
- `target/maven-status/`, `target/generated-sources/`, `target/surefire-reports/`

---

## 7. Environment variables

Make sure the following resolve (run in Git Bash):

```bash
echo $JAVA_HOME    # should print JDK install path
echo $MAVEN_HOME   # should print Maven install path
which java         # should find a binary
which mvn          # should find a binary
which python       # should find a binary
```

Windows (set via System Properties → Advanced → Environment Variables):

| Variable | Recommended value |
|---|---|
| `JAVA_HOME` | `D:\Program Files\Java\jdk-21` |
| `MAVEN_HOME` | `D:\Program Files\Maven\apache-maven-3.9.9` |
| `PATH` | Append `%JAVA_HOME%\bin` and `%MAVEN_HOME%\bin` |

> Already configured on this machine (all commands resolve directly).

---

## 8. Verification checklist (run these in order to confirm the env)

```bash
# 1. Java & Maven
cd "E:/data_compress/Flexible论文代码/Mix-Piece_Sim-Piece-main"
java -version && mvn -version

# 2. Compile (skip tests, fastest way to confirm the code compiles)
mvn compile test-compile -DskipTests -q
# Expected: BUILD SUCCESS, no errors

# 3. Python dependency
python -c "import openpyxl; print(openpyxl.__version__)"   # expect 3.x

# 4. Datasets complete
ls src/test/resources/*.csv.gz | wc -l   # expect 15 (incl. 2 backups)

# 5. Run a lightweight test
mvn test -Dtest=TestPLA -q
# Expected: Tests run: N, Failures: 0, Errors: 0

# 6. Run benchmark to generate baseline_multirun.csv
mvn test -Dtest=BaselineMultiRun
# Or directly:
java -cp "target/classes;target/test-classes;$(cat cp.txt)" \
     io.github.xkitsios.benchmarks.BaselineMultiRun
# Expected: writes 195 rows to results/baseline_multirun.csv

# 7. Regenerate LaTeX
python results/build_manuscript_sections.py
# Expected: "Wrote MANUSCRIPT_SECTIONS.tex"
```

---

## 9. Common issues

| Problem | Cause | Fix |
|---|---|---|
| `NoClassDefFoundError: com/github/luben/zstd/Zstd` | Missing zstd-jni jar | Add to classpath or run `mvn dependency:build-classpath` |
| `FileNotFoundException: src/test/resources/Cricket.csv.gz` | Dataset accidentally deleted | Do not delete datasets; restore from backup if missing |
| `ImportError: No module named openpyxl` | Missing Python package | `pip install openpyxl` |
| `BUILD FAILURE: cannot find symbol ... Lombok` | Lombok annotation processing failed | Install Lombok plugin in IDE, or just use `mvn compile` (Maven handles it) |
| Project broken after `mvn clean` | `target/` was wiped | Stop running `mvn clean`; rebuild with `mvn compile test-compile` |
| Port conflict (8080 etc.) | Default service port | Project exposes no network service, irrelevant |
| `OutOfMemoryError` (heap) | Large dataset at ε=0.0005 is very slow | `mvn test -Dargline="-Xmx4g"` or `set MAVEN_OPTS=-Xmx4g` |
| Python encoding garbled on Chinese path | Windows GBK | `set PYTHONIOENCODING=utf-8` |
| `module 'LinearCompressor' not found` | Missing custom Python module | Skip scripts that import it (e.g. use `build_tab_timing_*` instead) |

---

## 10. Recommended IDE setup

### IntelliJ IDEA
1. File → Open → select `pom.xml` in project root → Open as Project
2. Project SDK: choose JDK 21
3. Project language level: 8 (matches `<source>1.8</source>`)
4. Install plugin: Lombok
5. Run → Edit Configurations → Maven → command `mvn test -Dtest=BaselineMultiRun`

### VS Code
1. Install extensions: Extension Pack for Java, Python, Lombok
2. Open `settings.json` and add:
   ```json
   "java.configuration.runtimes": [{"name":"21","path":"D:\\Program Files\\Java\\jdk-21"}],
   "python.defaultInterpreterPath": "D:\\ProgramData\\anaconda3\\python.exe"
   ```

---

## 11. One-liner startup (recommended)

```bash
cd "E:/data_compress/Flexible论文代码/Mix-Piece_Sim-Piece-main" && \
  mvn test-compile -DskipTests -q && \
  echo "✅ Compile OK, ready to run tests"
```

Expected output: `✅ Compile OK, ready to run tests`.
