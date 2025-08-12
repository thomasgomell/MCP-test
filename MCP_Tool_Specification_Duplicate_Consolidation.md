# MCP Tool Spezifikation: Duplikat-Konsolidierung

**Tool Name:** `consolidate_duplicate_entities`  
**Kategorie:** Knowledge Graph Hygiene  
**Version:** 1.0  
**Datum:** 10. August 2025

## Übersicht

Automatisierte Konsolidierung von duplizierten Entitäten im Knowledge Graph mit semantischer Kategorisierung, deutscher Taxonomie und Zero-Data-Loss Garantie.

## Tool Definition

```json
{
  "name": "consolidate_duplicate_entities",
  "description": "Konsolidiert duplizierte Entitäten im Knowledge Graph durch intelligente Master-Node-Selektion und strukturierte Aspekt-Hierarchie-Erstellung",
  "parameters": {
    "properties": {
      "entity_name": {
        "type": "string",
        "description": "Name der zu konsolidierenden Entität (z.B. 'migRaven.Max')",
        "required": true
      },
      "consolidation_strategy": {
        "type": "string",
        "enum": ["auto", "manual", "preview"],
        "default": "auto",
        "description": "Konsolidierungsstrategie: auto=automatisch, manual=benutzergesteuert, preview=nur Analyse"
      },
      "target_language": {
        "type": "string",
        "enum": ["de", "en"],
        "default": "de",
        "description": "Zielsprache für Labels und Relationen"
      },
      "preserve_relations": {
        "type": "boolean",
        "default": true,
        "description": "Bestehende Relationen zu anderen Entitäten erhalten"
      },
      "aspect_categorization": {
        "type": "object",
        "properties": {
          "enable_auto_categorization": {
            "type": "boolean",
            "default": true,
            "description": "Automatische Kategorisierung in Aspekt-Typen"
          },
          "custom_categories": {
            "type": "array",
            "items": { "type": "string" },
            "description": "Benutzerdefinierte Kategorien zusätzlich zu Standard-Taxonomie"
          }
        }
      },
      "backup_before_merge": {
        "type": "boolean",
        "default": true,
        "description": "Backup der Original-Entitäten vor Konsolidierung erstellen"
      }
    },
    "required": ["entity_name"]
  }
}
```

## Funktionsweise

### 1. Duplikat-Detection

```python
def detect_duplicates(entity_name: str) -> DuplicateAnalysis:
    """
    Erkennt alle Entitäten mit gleichem Namen aber verschiedenen Labels
    """
    cypher_query = """
    MATCH (n {name: $entity_name})
    WITH n.name AS name,
         collect(DISTINCT apoc.coll.sort(labels(n))) AS label_combinations,
         collect(n) AS entities
    WHERE size(label_combinations) > 1
    RETURN name, label_combinations, entities, size(entities) AS duplicate_count
    """
    return execute_analysis(cypher_query, {"entity_name": entity_name})
```

### 2. Master-Node-Selektion

```python
def select_master_node(entities: List[Entity]) -> Entity:
    """
    Wählt Master-Node basierend auf Decision-Matrix aus
    """
    scoring_criteria = {
        "information_richness": 0.4,  # Anzahl Observations
        "actuality": 0.3,            # Neueste Timestamps
        "relation_count": 0.2,       # Anzahl Verbindungen
        "label_semantics": 0.1       # Semantische Aussagekraft
    }

    best_score = 0
    master_node = None

    for entity in entities:
        score = calculate_entity_score(entity, scoring_criteria)
        if score > best_score:
            best_score = score
            master_node = entity

    return master_node
```

### 3. Aspekt-Kategorisierung

```python
def categorize_aspects(entities: List[Entity], language: str = "de") -> Dict[str, List[Entity]]:
    """
    Kategorisiert Entitäten in semantische Aspekt-Gruppen
    """
    if language == "de":
        categories = {
            "Architekturaspekt": ["architecture", "deployment", "security", "system"],
            "Produktfeature": ["feature", "persona", "gamification", "ui"],
            "Strategieaspekt": ["strategy", "pricing", "market", "competitive"],
            "Dokument": ["documentation", "manual", "guide", "asset"],
            "Ereignis": ["event", "milestone", "go-live", "release"],
            "Anforderung": ["requirement", "specification", "constraint"]
        }

    categorized = {}
    for entity in entities:
        category = determine_category(entity, categories)
        if category not in categorized:
            categorized[category] = []
        categorized[category].append(entity)

    return categorized
```

### 4. Deutsche Relationen-Erstellung

```python
def create_german_relations(master_entity: Entity, aspect_entities: Dict[str, List[Entity]]) -> List[Relation]:
    """
    Erstellt semantische deutsche Relationen zwischen Master und Aspekten
    """
    relation_mapping = {
        "Architekturaspekt": "HAT_ARCHITEKTUR",
        "Produktfeature": "HAT_FEATURE",
        "Strategieaspekt": "HAT_STRATEGIE",
        "Dokument": "HAT_DOKUMENT",
        "Ereignis": "HAT_EREIGNIS",
        "Anforderung": "HAT_ANFORDERUNG"
    }

    relations = []
    for category, entities in aspect_entities.items():
        relation_type = relation_mapping.get(category, "HAT_ASPEKT")

        for entity in entities:
            relation = create_relation(
                source=master_entity.name,
                target=entity.name,
                relation_type=relation_type,
                properties={
                    "kategorie": category,
                    "konsolidierung_datum": datetime.now().isoformat(),
                    "automatisch_erstellt": True
                }
            )
            relations.append(relation)

    return relations
```

## Return Schema

```json
{
  "type": "object",
  "properties": {
    "status": {
      "type": "string",
      "enum": ["success", "error", "preview"],
      "description": "Ausführungsstatus"
    },
    "consolidation_summary": {
      "type": "object",
      "properties": {
        "original_entity_count": { "type": "integer" },
        "consolidated_entity_count": { "type": "integer" },
        "reduction_percentage": { "type": "number" },
        "master_entity": { "type": "string" },
        "aspect_entities": {
          "type": "array",
          "items": { "type": "string" }
        }
      }
    },
    "created_relations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "source": { "type": "string" },
          "target": { "type": "string" },
          "relation_type": { "type": "string" },
          "properties": { "type": "object" }
        }
      }
    },
    "backup_info": {
      "type": "object",
      "properties": {
        "backup_created": { "type": "boolean" },
        "backup_timestamp": { "type": "string" },
        "rollback_command": { "type": "string" }
      }
    },
    "errors": {
      "type": "array",
      "items": { "type": "string" }
    },
    "warnings": {
      "type": "array",
      "items": { "type": "string" }
    }
  }
}
```

## Verwendungsbeispiele

### 1. Automatische Konsolidierung

```json
{
  "tool": "consolidate_duplicate_entities",
  "parameters": {
    "entity_name": "migRaven.Max",
    "consolidation_strategy": "auto",
    "target_language": "de",
    "preserve_relations": true
  }
}
```

### 2. Preview-Modus

```json
{
  "tool": "consolidate_duplicate_entities",
  "parameters": {
    "entity_name": "Thomas Gomell",
    "consolidation_strategy": "preview",
    "target_language": "de"
  }
}
```

### 3. Manuelle Kategorisierung

```json
{
  "tool": "consolidate_duplicate_entities",
  "parameters": {
    "entity_name": "Neo4j",
    "consolidation_strategy": "manual",
    "aspect_categorization": {
      "enable_auto_categorization": false,
      "custom_categories": ["Technologie", "Datenbank", "Tool"]
    }
  }
}
```

## Sicherheitsfeatures

### 1. Backup & Rollback

```python
def create_backup(entities: List[Entity]) -> BackupInfo:
    """
    Erstellt vollständiges Backup vor Konsolidierung
    """
    backup_timestamp = datetime.now().isoformat()
    backup_data = {
        "entities": serialize_entities(entities),
        "relations": get_all_relations(entities),
        "timestamp": backup_timestamp
    }

    backup_id = store_backup(backup_data)

    return BackupInfo(
        backup_id=backup_id,
        rollback_command=f"restore_from_backup('{backup_id}')",
        timestamp=backup_timestamp
    )
```

### 2. Validierung

```python
def validate_consolidation(original_entities: List[Entity], result: ConsolidationResult) -> ValidationReport:
    """
    Validiert Konsolidierung auf Vollständigkeit und Konsistenz
    """
    checks = [
        check_information_preservation(original_entities, result),
        check_relation_integrity(result),
        check_naming_consistency(result),
        check_orphan_prevention(result)
    ]

    return ValidationReport(
        passed=all(check.passed for check in checks),
        checks=checks
    )
```

### 3. Rollback-Funktion

```python
def rollback_consolidation(backup_id: str) -> RollbackResult:
    """
    Stellt ursprünglichen Zustand aus Backup wieder her
    """
    backup_data = load_backup(backup_id)

    # Lösche konsolidierte Entitäten
    delete_consolidated_entities(backup_data.consolidated_entities)

    # Stelle Original-Entitäten wieder her
    restore_entities(backup_data.original_entities)
    restore_relations(backup_data.original_relations)

    return RollbackResult(status="success", restored_entities=len(backup_data.original_entities))
```

## Integration in MCP Server

### 1. Tool Registration

```python
# In mcp_server.py
from tools.duplicate_consolidation import ConsolidateDuplicateEntities

tools = [
    # ... bestehende Tools
    ConsolidateDuplicateEntities(),
]
```

### 2. Dependencies

```python
# requirements.txt Erweiterung
neo4j>=5.0.0
python-dateutil>=2.8.0
dataclasses-json>=0.5.0
```

### 3. Configuration

```yaml
# config.yaml Erweiterung
duplicate_consolidation:
  default_language: "de"
  backup_retention_days: 30
  max_entities_per_consolidation: 100
  auto_categorization_threshold: 0.8
```

## Monitoring & Metrics

### 1. Performance Metrics

- Durchschnittliche Ausführungszeit pro Konsolidierung
- Anzahl konsolidierte Entitäten pro Tag/Woche
- Erfolgsrate der automatischen Kategorisierung
- Backup-Größe und Retention-Statistiken

### 2. Quality Metrics

- Information-Preservation-Rate (Ziel: 100%)
- Relation-Integrity-Score (Ziel: >95%)
- User-Satisfaction mit automatischer Kategorisierung
- Rollback-Häufigkeit (Ziel: <5%)

### 3. Health Checks

```python
def health_check() -> HealthStatus:
    """
    Überprüft Tool-Verfügbarkeit und Dependencies
    """
    checks = [
        check_neo4j_connection(),
        check_backup_storage_space(),
        check_categorization_model_availability(),
        check_german_taxonomy_integrity()
    ]

    return HealthStatus(
        healthy=all(check.passed for check in checks),
        checks=checks
    )
```

## Roadmap

### Version 1.0 (MVP)

- ✅ Grundlegende Duplikat-Detection
- ✅ Master-Node-Selektion nach Decision-Matrix
- ✅ Deutsche Aspekt-Kategorisierung
- ✅ Backup & Rollback-Funktionalität

### Version 1.1 (Enhanced)

- 🔄 Machine Learning für Kategorisierung
- 🔄 Batch-Processing für Multiple-Entitäten
- 🔄 UI für manuelle Review und Approval
- 🔄 Advanced Relation-Pattern-Recognition

### Version 1.2 (Advanced)

- 📋 Cross-Entity-Dependency-Analysis
- 📋 Automated Taxonomy-Evolution
- 📋 Multi-Language-Support (EN, FR, IT)
- 📋 Integration mit External Knowledge Bases

## Fazit

Das `consolidate_duplicate_entities` Tool standardisiert und automatisiert die erfolgreiche migRaven.Max Konsolidierung als wiederverwendbare MCP-Funktionalität. Es bietet:

- **Skalierbarkeit:** Anwendbar auf beliebige Duplikat-Fälle
- **Sicherheit:** Backup/Rollback und Validierung
- **Konsistenz:** Deutsche Taxonomie-Standards
- **Automatisierung:** Minimaler manueller Aufwand
- **Qualität:** Zero-Data-Loss und Integrity-Checks

Die Integration dieses Tools transformiert manuelle Graph-Hygiene in systematische, automatisierte Knowledge-Quality-Assurance.
