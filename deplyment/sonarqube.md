# SonarQube + JaCoCo: Quality Gates for Production-Ready Code

## How We Use JaCoCo (Code Coverage Tool)

- **Tool**: JaCoCo 0.8.12
- **When**: Runs **automatically** after every `./gradlew test`
- **What**: Measures **lines** and **branches** executed by tests
- **Output**: `build/reports/jacoco/test/jacocoTestReport.xml` (XML for SonarQube) + HTML for local view
- **Enforcement in build** (`build.gradle`):

```groovy
jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                counter = 'BRANCH'
                value = 'COVEREDRATIO'
                minimum = 0.80   // build FAILS if < 80% branch coverage
            }
        }
    }
}
check.dependsOn jacocoTestCoverageVerification
```

**Interview point**: JaCoCo enforces coverage **at build time**. The build fails before code reaches SonarQube if < 80% branch coverage.

---

## How We Use SonarQube (Static Analysis + Dashboard)

- **Plugin**: `org.sonarqube` version `5.1.0.4882` at root `build.gradle`
- **Server**: Docker — SonarQube 10 Community + PostgreSQL 16 (`docker/docker-compose.sonar.yml`)
- **URL**: `http://localhost:9000`

**Configuration**:

```groovy
sonarqube {
    properties {
        property 'sonar.projectKey', 'microservices-pattern-lab'
        property 'sonar.host.url', 'http://localhost:9000'
        property 'sonar.coverage.exclusions', '**/dto/**,**/config/**'
    }
}
```

**Command to push coverage**:

```bash
./gradlew test jacocoTestReport sonarqube -Dsonar.login=<token>
```

**Pipeline**: Tests → JaCoCo XML → SonarQube reads XML + static analysis (bugs, vulns, smells, duplication) → Quality Gate pass/fail.

---

## Quality Gates — Interview Definitions

A **Quality Gate** is a boolean checkpoint that code must pass to be considered production-ready.

### Default "Sonar Way" Gate

| Metric | Threshold |
|--------|-----------|
| Coverage | ≥ 80% |
| Duplicated Lines | ≤ 3% |
| Reliability Rating | A (zero bugs) |
| Security Rating | A (zero vulnerabilities) |
| Security Hotspots Reviewed | 100% |
| Maintainability Rating | A (Tech Debt Ratio < 5%) |

### Rating System (A–E)

| Rating | Meaning |
|--------|---------|
| A | 0 issues — clean |
| B | 1–2 minor issues — low risk |
| C | 3–7 issues — medium risk |
| D | 8–20 issues — high risk |
| E | 21+ issues — very high risk |

### Each Metric Explained

| Metric | Interview Answer |
|--------|-----------------|
| **Reliability Rating (bugs)** | *"Rating A means zero bugs found by static analysis. A single bug drops it to B."* |
| **Security Rating (vulns)** | *"Rating A means zero exploitable weaknesses. Security is non-negotiable."* |
| **Security Hotspots** | *"Every hotspot must be manually reviewed as 'Safe' or 'Fixed'. 100% review required."* |
| **Maintainability Rating** | *"Rating A means Tech Debt Ratio < 5%. Code is easy to maintain."* |
| **Coverage** | *"We enforce 80% branch coverage via JaCoCo at build time, SonarQube mirrors it."* |
| **Duplicated Lines** | *"> 3% duplication fails the gate. Prevents copy-paste tech debt."* |

---

## Why Two Layers? (JaCoCo + SonarQube)

| Layer | When | What |
|-------|------|------|
| **JaCoCo** | During `gradle check` | Fails build immediately if < 80% coverage — **fast feedback** |
| **SonarQube** | During CI (`sonarqube` task) | Dashboard, trends, static analysis (bugs, vulns, smells), Quality Gate |

**Interview answer**: *"JaCoCo enforces coverage at build level for instant feedback. SonarQube aggregates across microservices, runs deeper analysis, and applies a multi-dimensional Quality Gate before code merges."*

---

## Custom Quality Gate (Production-Ready)

| Metric | Threshold | Why |
|--------|-----------|-----|
| Coverage | ≥ 80% | Matches JaCoCo enforcement |
| New Code Coverage | ≥ 80% | Legacy exempt; new code must be tested |
| Reliability Rating | A | Zero bugs tolerated |
| Security Rating | A | Zero vulnerabilities |
| Security Hotspots Reviewed | 100% | Every hotspot triaged |
| Maintainability Rating | A or B | Allow minor smells, keep debt < 5% |
| Duplicated Lines | ≤ 3% | No copy-paste across services |

---

## Interview Talking Points

1. **Shift-left**: Quality Gates catch issues at build/CI, not in production
2. **Two layers**: JaCoCo for fast feedback, SonarQube for comprehensive governance
3. **Single source of truth**: Green gate = deployable code
4. **New Code focus**: Gates on new/changed code so legacy doesn't block progress
