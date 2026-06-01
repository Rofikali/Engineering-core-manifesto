# 📐 Low-Level Design: Architectural Dependency Injection (DI)

## 1. Intent & Context
Eliminate hardcoded dependency instantiation inside submodules or controller routing scopes. Objects must be given their dependencies from an external assembler framework, making code modular, testable, and isolated.

## 2. Production Implementation (FastAPI Example)
```python
from fastapi import Depends
from typing import Generator
import psycopg2

class DatabaseConnectionContainer:
    def __init__(self, dsn: str):
        self._dsn = dsn

    def get_db_session(self) -> Generator:
        """
        Yield-based Context Generator enforcing clean setup 
        and automatic teardown parameters.
        """
        connection = psycopg2.connect(self._dsn)
        cursor = connection.cursor()
        try:
            yield cursor
            connection.commit()
        except Exception:
            connection.rollback()
            raise
        finally:
            cursor.close()
            connection.close()

# Mock Global Container
container = DatabaseConnectionContainer(dsn="postgresql://user:pass@localhost:5432/db")

# Controller / Routing Ingress Injected Boundary
# Endpoint knows nothing about DSN, drivers, or pool allocation mechanics
def read_product_catalog(product_id: str, db=Depends(container.get_db_session)):
    db.execute("SELECT * FROM products WHERE id = %s", (product_id,))
    return db.fetchone()
```

## 3. Engineering Checklists
*   [ ] **Scope Demarcation**: Always separate lifecycle definitions cleanly (e.g., Singleton for long-lived global caches, Transient/Scoped for per-request database sessions).
*   [ ] **Test Environment Swapping**: Ensure injection architectures permit simple, total mocking of core structures during unit testing sequences.
