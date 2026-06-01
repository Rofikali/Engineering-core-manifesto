# 📐 Low-Level Design: Factory Pattern Blueprint

## 1. Intent & Context
Centralize object instantiation routines. Protects business execution paths from having to know the initialization parameters or specific module variants required to spin up third-party or domestic service layers.

## 2. Implementation Pattern
```python
from typing import Dict, Type
import os

class NotificationFactory:
    """
    Centralized Factory configuration mapping strategy variants 
    to explicit execution models.
    """
    _registry: Dict[str, Type[PaymentStrategy]] = {}

    @classmethod
    def register_engine(cls, channel_type: str, engine_class: Type[PaymentStrategy]) -> None:
        cls._registry[channel_type] = engine_class

    @classmethod
    def get_engine(cls, channel_type: str) -> PaymentStrategy:
        engine = cls._registry.get(channel_type)
        if not engine:
            raise KeyError(f"Requested engine variant '{channel_type}' is unmapped in core registries.")
        
        # Centralize configuration variable access here instead of individual files
        if channel_type == "razorpay":
            return engine(
                key_id=os.environ["RAZORPAY_KEY_ID"], 
                secret=os.environ["RAZORPAY_KEY_SECRET"]
            )
        return engine()
```

## 3. Engineering Checklists
*   [ ] **Open-Closed Principle Compliance**: Leverage a class-level lookup registration method (`register_engine`) so you can add new strategies without rewriting the primary factory layout.
