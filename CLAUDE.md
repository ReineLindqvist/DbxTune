# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DbxTune is a Java database performance monitoring and tuning toolkit supporting multiple DBMS vendors (ASE, SQL Server, PostgreSQL, IQ, MySQL, Oracle, DB2, HANA, Replication Server). It provides GUI-based real-time monitoring, headless data collection (NO-GUI mode), offline replay, and a centralized web dashboard (DbxCentral).

## Build Commands

The project uses Apache Ant with Java 11 (source/target).

```bash
ant compile       # Compile Java sources
ant dist          # Build distribution ZIP (dbxtune_YYYY-MM-DD.zip)
ant clean         # Clean build artifacts
ant -p            # List all available targets
```

## Running

```bash
./bin/asetune.sh          # ASE monitoring
./bin/sqlservertune.sh    # SQL Server monitoring
./bin/postgrestune.sh     # PostgreSQL monitoring
./bin/dbxcentral.sh       # Central web dashboard
./bin/sqlw.sh             # SQL query tool (JDBC)
```

All launcher scripts delegate to `bin/dbxtune.sh` with tool-specific arguments.

## Architecture

### Plugin-based DBMS Support

Abstract base class `DbxTune` (`src/com/dbxtune/DbxTune.java`) uses Template Method pattern. Each DBMS has a concrete subclass (e.g., `AseTune`, `SqlServerTune`, `PostgresTune`) at `src/com/dbxtune/`.

Key interfaces each DBMS implementation provides:
- `ICounterController` — main controller driving data collection
- `IDbmsConfig` — DBMS-specific configuration
- `IObjectLookupInspector` — object inspection in GUI

### Core Subsystems

- **Counter Modules (`src/com/dbxtune/cm/`)** — Performance collectors per DBMS. Each `cm/<vendor>/` package contains `CountersModel` subclasses that query DBMS-specific views/tables and support delta calculations.
- **Persistent Counter Storage (`src/com/dbxtune/pcs/`)** — Records all performance data to H2 databases (one DB per day).
- **Alarm System (`src/com/dbxtune/alarm/`)** — Threshold-based alerting with writers for email, Slack, syslog, REST, etc.
- **DbxCentral (`src/com/dbxtune/central/`)** — Jetty-based web server aggregating data from multiple NO-GUI collectors. Web UI in `resources/WebContent/`.
- **GUI (`src/com/dbxtune/gui/`)** — Swing desktop application for real-time monitoring and offline replay.
- **SQL tools (`src/com/dbxtune/tools/`)** — SQL Window (sqlw), Tail Window, PerfDemo utilities.

### Adding a New DBMS Vendor

Follow the guide in `README_howto_add_dbxVendor.md`. Key steps: create a `*Tune.java` main class, implement counter modules in `cm/<vendor>/`, add icons, launcher scripts, and build.xml entries.

## Key Dependencies

- **UI**: Swing, SwingX, JFreeChart, RSyntaxTextArea, MigLayout
- **Web**: Embedded Jetty 9.4
- **Storage**: H2 database (recording DB)
- **Utilities**: Apache Commons, Guava, Jackson, Log4j 2, JSQLParser
- **SSH**: JSCH
- JDBC drivers in `lib/jdbc_drivers/` for each supported DBMS

## Project Layout

- `src/com/dbxtune/` — All Java source code
- `test/` — Test sources
- `lib/` — Third-party JARs (including `jdbc_drivers/`, `jetty/`)
- `resources/` — Web content, tooltip XML definitions
- `bin/` — Launch scripts (.sh, .bat)
- `conf/` — Configuration files (e.g., `dbxtune.properties`)
- `build.xml` — Ant build file

## IDE

Eclipse project files (`.project`, `.classpath`) are included. Code formatting rules in `EclipseCodeStyleFormatter.xml`.
